[//]: # (title: Updating a Kotlin Multiplatform project to support AGP 9)
<show-structure for="chapter,procedure" depth="3"/>

Kotlin Multiplatform projects targeting Android need to migrate to using the new
[Android Gradle Library plugin](https://developer.android.com/kotlin/multiplatform/plugin)
to be able to use Android Gradle plugin version 9.0 and newer.

> AGP 9 is supported in IntelliJ IDEA starting with version 2025.3 TODO
> and Android Studio starting with Otter 2025.3 TODO
> 
{style="note"}

## Mandatory changes

### Migrate to built-in Kotlin

AGP 9.0 brings built-in Kotlin support that is enabled by default,
so you don't need to enable the Kotlin Android Gradle plugin (`org.jetbrains.kotlin.android`) explicitly anymore.

If you're using [kapt](https://kotlinlang.org/docs/kapt.html) or custom `kotlinOptions`,
you may need to perform additional migration steps.

Follow the [migration guide](https://developer.android.com/build/migrate-to-built-in-kotlin)
to make sure you don't miss anything.

### Migrate to the Android Gradle Library plugin

Previously, to configure an Android target in a multiplatform module you needed to use
the KMP plugin (`org.jetbrains.kotlin.multiplatform`)
together with either
the Android application (`com.android.application`) or the Android library (`com.android.library`) plugin.

With AGP 9.0, these plugins stop being compatible with KMP,
and you need to migrate to the new Android Gradle Library plugin built specifically for KMP.

> To make your project work with AGP 9.0 in the short term, you can manually enable the deprecated APIs.
> To do that, in the `gradle.properties` file of your project add this property:
> `android.enableLegacyVariantApi=true`.
>
> The legacy APIs are going to be removed completely in AGP 10. Make sure you finish the migration before that!
>
{style="note"}

For library migration steps, see the [guide in Android documentation](https://developer.android.com/kotlin/multiplatform/plugin#migrate).

To migrate an app project, you need to have the Android entry point and the shared code in properly configured separate modules.
There is a [general tutorial for migrating a sample app](#step-by-step-migration-of-a-sample-app),
but the mandatory parts are:
* [Create and configure a shared module](#create-shared-modules)
* [Create and configure an Android app module](#android-app)

## Recommended changes

Along with updating your Gradle configuration, we recommend to review the structure of your KMP project.
The general approach is to extract shared code in its own module, and create a separate module for each platform-specific
entry point.

In terms of [Android modularization approach](https://developer.android.com/topic/modularization),
we recommend creating separate **app modules** for platform-specific entry points
(so, a separate module for Android, web, and JVM)
and **feature modules** for shared code.

<!-- TODO link to the blog post explaining the changes -->

## Step by step migration of a sample app

The example project that you will prepare for the migration is a Compose Multiplatform app that is the result of the
[](compose-multiplatform-new-project.md)
tutorial.
The example consists of a single Gradle module (`composeApp`) that contains all the shared code and KMP entry
points,
and the `iosApp` project with the iOS-specific code and configuration.

To prepare for the AGP 9 migration and to create a generally more flexible and scalable project structure,
you will:

* Create feature modules for shared UI and business logic code.
* Create app modules with entry points for each platform.

If you are looking to only implement changes mandatory for the AGP 9 migration,
you required steps are:

* [Create and configure a shared module](#create-shared-modules)
* [Create and configure an Android app module](#android-app)

<!-- When the new structure is implemented in the wizard, this is going to change: 
     following the tutorial will bring you to the new structure already.
     So we need to save a sample of "how things used to be" and link to it when the wizard update hits.-->

### Create modules for the Android entry point and other app targets

The only entry point that needs isolating to enable AGP 9 migration is the Android app.
But in a project with several targets enabled, it's more straightforward and transparent to keep all the entry points
on the same level of the project hierarchy.
So below we provide instructions for all targets supported in the sample app. 

#### Android app

Create and configure a new entry point module for the Android app:

1. Create the `androidApp` directory at the root of the project.
2. Inside that directory, create an empty `build.gradle.kts` file and the `src` directory.
3. Add the new module to project settings in the `settings.gradle.kts` file by adding this line at the end of the file:

    ```kotlin
    include(":androidApp")
    ```
4. Configure the Gradle build script for the new module.

    1. In the `gradle/libs.versions.toml` file, add the Kotlin Android Gradle plugin to your version catalog:
       TODO this needs to be removed before actually building with AGP 9

        ```text
        [plugins]
        kotlinAndroid = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
        ```

    2. In the `androidApp/build.gradle.kts` file, specify the plugins necessary for the shared UI module:

        ```kotlin
        plugins {
           alias(libs.plugins.kotlinAndroid)
           alias(libs.plugins.androidApplication)
           alias(libs.plugins.composeMultiplatform)
           alias(libs.plugins.composeCompiler)
        }
        ```

    3. Make sure all of these plugins are mentioned in the **root** `build.gradle.kts` file:

        ```kotlin
        plugins {
            alias(libs.plugins.kotlinAndroid) apply false
            alias(libs.plugins.androidApplication) apply false
            alias(libs.plugins.composeMultiplatform) apply false
            alias(libs.plugins.composeCompiler) apply false
            // ...
        }
        ```

    4. To add the necessary dependencies, copy existing dependencies from the `androidMain.dependencies {}` block
       of the `composeApp` build script, and add the dependency on the `composeApp` itself. 
       In this example the end result should look like this:

       ```kotlin
       kotlin {
           dependencies { 
               implementation(projects.composeApp)
               implementation(libs.androidx.activity.compose)
               implementation(compose.uiToolingPreview)
           }
       }
       ```

    5. Copy the entire `android {}` block with Android-specific configuration from the `composeApp/build.gradle.kts`
       file to the `androidApp/build.gradle.kts` file.
    6. Change the configuration from an Android application to an Android library,
       since that's what it becomes. In `composeApp/build.gradle.kts`:
       * Change the reference to the Gradle plugin:

           <compare type="top-bottom">
              <code-block lang="kotlin">
                  alias(libs.plugins.androidApplication)
              </code-block>
              <code-block lang="kotlin">
                  alias(libs.plugins.androidLibrary)
              </code-block>
           </compare>
       
        * Remove the application property lines from the `android.defaultConfig {}` block:

          <compare type="top-bottom">
              <code-block lang="kotlin">
                  defaultConfig {
                      applicationId = "com.jetbrains.demo"
                      minSdk = libs.versions.android.minSdk.get().toInt()
                      targetSdk = libs.versions.android.targetSdk.get().toInt()
                      versionCode = 1
                      versionName = "1.0"
                  }
              </code-block>
              <code-block lang="kotlin">
                  defaultConfig {
                      minSdk = libs.versions.android.minSdk.get().toInt()
                  }
              </code-block>
           </compare>
       
    7. Select **Build | Sync Project with Gradle Files** in the main menu, or click the Gradle refresh button in the
       editor.

5. Copy the `composeApp/src/androidMain` directory into the `androidApp/src/` directory.
6. Rename the `androidApp/src/androidMain` directory into `main`.
7. If everything is configured correctly, the imports in the `androidApp/src/main/.../MainActivity.kt` file are working
   and the code is compiling.
8. To run your Android app, change and rename the **composeApp** Android run configuration or add a similar one.
   In the **General | Module** field, change `demo.composeApp` to `demo.androidApp`.
9. Start the run configuration to make sure that the app runs as expected.
10. If everything works correctly, remove the `composeApp/src/androidMain` directory.

You have extracted the Android entry point to a separate module.
If you don't have any other entry points and want to keep changes to a minimum to upgrade to AGP 9,
you can jump straight to [configuring a shared module](#configure-a-shared-module).

#### Desktop JVM app

Create and configure the JVM desktop app module:

1. Create the `desktopApp` directory at the root of the project.
2. Inside that directory, create an empty `build.gradle.kts` file and the `src` directory.
3. Add the new module to project settings in the `settings.gradle.kts` file by adding this line at the end of the file:

    ```kotlin
    include(":desktopApp")
    ```
4. Configure the Gradle build script for the new module.

    1. In the `gradle/libs.versions.toml` file, add the Kotlin JVM Gradle plugin to your version catalog:

        ```text
        [plugins]
        kotlinJvm = { id = "org.jetbrains.kotlin.jvm", version.ref = "kotlin" }
        ```

    2. In the `desktopApp/build.gradle.kts` file, specify the plugins necessary for the shared UI module:

        ```kotlin
        plugins {
           alias(libs.plugins.kotlinJvm)
           alias(libs.plugins.composeMultiplatform)
           alias(libs.plugins.composeCompiler)
           alias(libs.plugins.composeHotReload)
        }
        ```

    3. Make sure all of these plugins are mentioned in the **root** `build.gradle.kts` file:

        ```kotlin
        plugins {
            alias(libs.plugins.kotlinJvm) apply false
            alias(libs.plugins.composeHotReload) apply false
            alias(libs.plugins.composeMultiplatform) apply false
            alias(libs.plugins.composeCompiler) apply false
            // ...
        }
        ```

    4. To add the necessary dependencies on other modules, copy existing dependencies from the
       `commonMain.dependencies {}` and `jvmMain.dependencies {}` blocks
       of the `composeApp` build script. In this example the end result should look like this:

       ```kotlin
       kotlin {
           dependencies { 
               implementation(projects.sharedLogic)
               implementation(projects.sharedUI)
               implementation(compose.desktop.currentOs)
               implementation(libs.kotlinx.coroutinesSwing)
           }
       }
       ```

    5. Copy the `compose.desktop {}` block with desktop-specific configuration from the `composeApp/build.gradle.kts`
       file to the `desktopApp/build.gradle.kts` file:

        ```kotlin
        compose.desktop {
            application {
                mainClass = "compose.project.demo.MainKt"

                nativeDistributions {
                    targetFormats(TargetFormat.Dmg, TargetFormat.Msi, TargetFormat.Deb)
                    packageName = "compose.project.demo"
                    packageVersion = "1.0.0"
                }
            }
        }
        ```
    6. Select **Build | Sync Project with Gradle Files** in the main menu, or click the Gradle refresh button in the
       editor.

5. Move the code: In the `desktopApp/src` directory, create a new `main` directory.
6. Copy the `composeApp/src/jvmMain/kotlin` directory into the `desktopApp/src/main/` directory:
   It's important that the package coordinates are aligned with the `compose.desktop {}` configuration.
7. If everything is configured correctly, the imports in the `desktopApp/src/main/.../main.kt` file are working
   and the code is compiling.
8. To run your desktop app, change and rename the **composeApp [jvm]** run configuration or add a similar one.
   In the **Gradle project** field, change `ComposeDemo:composeApp` to `ComposeDemo:desktopApp`.
9. Start the run configuration to make sure that the app runs as expected.
10. If everything works correctly:
    * Remove the `composeApp/src/jvmMain` directory.
    * In the `composeApp/build.gradle.kts` file, remove the desktop-related code:
        * the `compose.desktop {}` block,
        * the `jvmMain.dependencies {}` block inside the Kotlin `sourceSets {}` block,
        * the `jvm()` target declaration inside the `kotlin {}` block.

#### Web app

Create and configure the web app module:

1. Create the `webApp` directory at the root of the project.
2. Inside that directory, create an empty `build.gradle.kts` file and the `src` directory.
3. Add the new module to project settings in the `settings.gradle.kts` file by adding this line at the end of the file:

    ```kotlin
    include(":webApp")
    ```
4. Configure the Gradle build script for the new module.

    1. In the `webApp/build.gradle.kts` file, specify the plugins necessary for the shared UI module:

        ```kotlin
        plugins {
           alias(libs.plugins.kotlinMultiplatform)
           alias(libs.plugins.composeMultiplatform)
           alias(libs.plugins.composeCompiler)
        }
        ```

    2. Make sure all of these plugins are mentioned in the **root** `build.gradle.kts` file:

        ```kotlin
        plugins {
            alias(libs.plugins.kotlinMultiplatform) apply false
            alias(libs.plugins.composeMultiplatform) apply false
            alias(libs.plugins.composeCompiler) apply false
            // ...
        }
        ```

    3. Copy the JavaScript and Wasm target declarations from the `composeApp/build.gradle.kts` file into the `kotlin {}` block
       in the `webApp/build.gradle.kts` file:

        ```kotlin
        kotlin {
            js {
                browser()
                binaries.executable()
            }

            @OptIn(ExperimentalWasmDsl::class)
            wasmJs {
                browser()
                binaries.executable()
            }
        }
        ```

    4. Add the necessary dependencies on other modules:

       ```kotlin
       kotlin {
           sourceSets {
               commonMain.dependencies { 
                   implementation(projects.sharedLogic)
                   // Provides the necessary entry point API
                   implementation(compose.ui)
               }
           }
       }
       ```

    5. Select **Build | Sync Project with Gradle Files** in the main menu, or click the Gradle refresh button in the
       editor.

5. Copy the entire `composeApp/src/webMain` directory into the `webApp/src` directory.
   If everything is configured correctly, the imports in the `webApp/src/webMain/.../main.kt` file are working
   and the code is compiling.
6. In the `webApp/src/webMain/resources/index.html` file update the script name: from `composeApp.js` to `webApp.js`.
7. Run your web app: change and rename the **composeApp [wasmJs]** and **composeApp [js]** run configurations or add similar ones.
   In the **Gradle project** field, change `ComposeDemo:composeApp` to `ComposeDemo:webApp`.
8. Start the run configurations to make sure that the app runs as expected.
9. If everything works correctly:
    * Remove the `composeApp/src/webMain` directory.
    * In the `composeApp/build.gradle.kts` file, remove the web-related code:
        * the `webMain.dependencies {}` block inside the Kotlin `sourceSets {}` block,
        * the `js {}` and `wasmJs {}` target declarations inside the `kotlin {}` block.


### Create shared modules

The rule of thumb for the migration of shared modules is:

* If you implement Compose Multiplatform UI for each of your apps, you only need a single `shared` module that
  includes all multiplatform dependencies.
* If at least one of your apps has fully native UI and only should depend on common code for business logic,
  create a `sharedLogic` and a `sharedUI` module to separate them.

The UI in the sample project is fully implemented using Compose Multiplatform,
so a single `shared` module makes sense.
But to illustrate the more general case there's an optional section on extracting non-UI code and the changes you
would need to make. TODO link

#### Configure a `shared` module

Since you have already extracted all entry points (platform-specific code) out of the `composeApp` module,
you can simply continue using it as the module with common code.
You only need to adjust the Gradle configuration:

1. In `gradle/libs.versions.toml`,
   add the Android Gradle Library plugin to your version catalog:

    ```text
    [plugins]
    androidMultiplatformLibrary = { id = "com.android.kotlin.multiplatform.library", version.ref = "agp" }
    ```

2. In the `composeApp/build.gradle.kts` file, add the plugin the plugins necessary for the shared UI module:

    ```kotlin
    plugins {
       alias(libs.plugins.kotlinMultiplatform)
       alias(libs.plugins.androidMultiplatformLibrary)
       alias(libs.plugins.composeMultiplatform)
       alias(libs.plugins.composeCompiler)
       alias(libs.plugins.composeHotReload)
    }
    ```

#### Create a shared logic module

Before actually creating a module, you need to decide on what is business logic, which code is both UI- and
platform-independent.
In this example, the only clear candidate is the `currentTimeAt()` function that returns exact time for a pair of
location and time zone.
The `Country` data class, for example, relies on `DrawableResource` from Compose Multiplatform and can't be separated
from UI code.

> If your project already has a `shared` module, for example, because you don't share the entirety of UI code,
> then you can use this module in place of `sharedLogic` — or rename it to better differentiate between shared logic and UI.
> 
{style="note"}

Isolate the corresponding code in a `sharedLogic` module:

1. Create the `sharedLogic` directory at the root of the project.
2. Inside that directory, create an empty `build.gradle.kts` file and the `src` directory.
3. Add the new module to `settings.gradle.kts` by adding this line at the end of the file:

    ```kotlin
    include(":sharedLogic")
    ```
4. Configure the Gradle build script for the new module.

    1. In the `gradle/libs.versions.toml` file, add the Android Gradle Library plugin to your version catalog:

        ```text
        [plugins]
        androidMultiplatformLibrary = { id = "com.android.kotlin.multiplatform.library", version.ref = "agp" }
        ```

    2. In the `sharedLogic/build.gradle.kts` file, specify the plugins necessary for the shared logic module:

       ```kotlin
       plugins {
           alias(libs.plugins.kotlinMultiplatform)
           alias(libs.plugins.androidMultiplatformLibrary)
       }
       ```
    3. Make sure these plugins are mentioned in the **root** `build.gradle.kts` file:

       ```kotlin
       plugins {
         alias(libs.plugins.androidMultiplatformLibrary) apply false
         alias(libs.plugins.kotlinMultiplatform) apply false
         // ...
       }
       ```
    4. In the `kotlin {}` block, specify the targets that the common module should support in this example:

        ```kotlin
        kotlin {
            // There's no need for iOS framework configuration since sharedLogic
            // is not going to be exported as a framework, only sharedUI is.
            iosArm64()
            iosSimulatorArm64()
     
            jvm()
     
            js {
                browser()
            }
     
            @OptIn(ExperimentalWasmDsl::class)
            wasmJs {
                browser()
            }
        }
        ```
    5. For Android, instead of the `androidTarget {}` block, add the `androidLibrary {}` configuration to the
       `kotlin {}` block:

        ```kotlin
        kotlin {
            // ...
            androidLibrary {
                namespace = "com.jetbrains.greeting.demo.sharedLogic"
                compileSdk = libs.versions.android.compileSdk.get().toInt()
                minSdk = libs.versions.android.minSdk.get().toInt()
        
                compilerOptions {
                    jvmTarget = JvmTarget.JVM_11
                }
            }
        }
        ```
    6. Add the necessary time dependencies for the common and JavaScript source sets in the same
       way they are declared for `composeApp`:

        ```kotlin
        kotlin {
            sourceSets {
                commonMain.dependencies {
                    implementation("org.jetbrains.kotlinx:kotlinx-datetime:%dateTimeVersion%")
                }
                webMain.dependencies {
                    implementation(npm("@js-joda/timezone", "2.22.0"))
                }
            }
        }
        ```
    7. Select **Build | Sync Project with Gradle Files** in the main menu, or click the Gradle refresh button in the
       editor.

5. Move the business logic code identified in the beginning:
    1. Create a `commonMain/kotlin` directory inside `sharedLogic/src`.
    2. Inside `commonMain/kotlin`, create the `CurrentTime.kt` file.
    3. Move the `currentTimeAt` function from the original `App.kt` to `CurrentTime.kt`.
6. Make the function available to the `App()` composable at its new place.
   To do that, declare the dependency between `composeApp` and `sharedLogic` in the `composeApp/build.gradle.kts` file:

    ```kotlin
    commonMain.dependencies {
        implementation(projects.sharedLogic)
    }
    ```
7. Run **Build | Sync Project with Gradle Files** again to make the dependency work.
8. In the `composeApp/commonMain/.../App.kt` file, import the `currentTimeAt()` function to fix the code.
9. Run the application to make sure that your new module functions properly.

You have isolated the shared logic in a separate module and successfully used it cross-platform.
Next step, creating a shared UI module.

#### Create a shared UI module

Extract shared code implementing common UI elements in the `sharedUI` module:

1. Create the `sharedUI` directory at the root of the project.
2. Inside that directory, create an empty `build.gradle.kts` file and the `src` directory.
3. Add the new module to `settings.gradle.kts` by adding this line at the end of the file:

    ```kotlin
    include(":sharedUI")
    ```
4. Configure the Gradle build script for the new module:

    1. If you haven't done this for the `sharedLogic` module, in `gradle/libs.versions.toml`,
       add the Android Gradle Library plugin to your version catalog:

        ```text
        [plugins]
        androidMultiplatformLibrary = { id = "com.android.kotlin.multiplatform.library", version.ref = "agp" }
        ```

    2. In the `sharedUI/build.gradle.kts` file, specify the plugins necessary for the shared UI module:

        ```kotlin
        plugins {
           alias(libs.plugins.kotlinMultiplatform)
           alias(libs.plugins.androidMultiplatformLibrary)
           alias(libs.plugins.composeMultiplatform)
           alias(libs.plugins.composeCompiler)
           alias(libs.plugins.composeHotReload)
        }
        ```

    3. Make sure all of these plugins are mentioned in the **root** `build.gradle.kts` file:

        ```kotlin
        plugins {
            alias(libs.plugins.androidMultiplatformLibrary) apply false
            alias(libs.plugins.composeHotReload) apply false
            alias(libs.plugins.composeMultiplatform) apply false
            alias(libs.plugins.composeCompiler) apply false
            alias(libs.plugins.kotlinMultiplatform) apply false
            // ...
        }
        ```

    4. In the `kotlin {}` block, specify the targets that the shared UI module should support in this example:

        ```kotlin
        kotlin {
            listOf(
                iosArm64(),
                iosSimulatorArm64()
            ).forEach { iosTarget ->
                iosTarget.binaries.framework {
                    // This is the name of the iOS framework you're going
                    // to import in your Swift code.
                    baseName = "sharedUI"
                    isStatic = true
                }
            }
     
            jvm()
     
            js {
                browser()
                binaries.executable()
            }
     
            @OptIn(ExperimentalWasmDsl::class)
            wasmJs {
                browser()
                binaries.executable()
            }
        }
        ```

    5. For Android, instead of the `androidTarget {}` block, add the `androidLibrary {}` configuration to the
       `kotlin {}` block:

        ```kotlin
        kotlin {
            // ...
            androidLibrary {
                namespace = "com.jetbrains.greeting.demo.sharedLogic"
                compileSdk = libs.versions.android.compileSdk.get().toInt()
                minSdk = libs.versions.android.minSdk.get().toInt()
         
                compilerOptions {
                    jvmTarget = JvmTarget.JVM_11
                }
       
                // Enables Compose Multiplatform resources to be used in the Android app
                androidResources {
                    enable = true
                }
            }
        }
        ```

    6. Add the necessary dependencies for the shared UI in the same way they are declared for `composeApp`:

       ```kotlin
       kotlin {
           sourceSets {
               commonMain.dependencies { 
                   implementation(projects.sharedLogic)
                   implementation(compose.runtime)
                   implementation(compose.foundation)
                   implementation(compose.material3)
                   implementation(compose.ui)
                   implementation(compose.components.resources)
                   implementation(compose.components.uiToolingPreview)
                   implementation(libs.androidx.lifecycle.viewmodelCompose)
                   implementation(libs.androidx.lifecycle.runtimeCompose)
                   implementation("org.jetbrains.kotlinx:kotlinx-datetime:%dateTimeVersion%")
               }
           }
       }
       ```
    7. Select **Build | Sync Project with Gradle Files** in the main menu, or click the Gradle refresh button in the
       editor.
5. Create a new `commonMain/kotlin` directory inside `sharedUI/src`.
6. Move resource files to the `sharedUI` module: the entire directory of `composeApp/commonMain/composeResources` should
   be relocated to `sharedUI/commonMain/composeResources`.
7. In the `sharedUI/src/commonMain/kotlin directory`, create a new `App.kt` file.
8. Copy the entire contents of the original `composeApp/src/commonMain/.../App.kt` to the new `App.kt` file.
9. Comment out all code in the old `App.kt` file in the meantime.
   You'll test whether the shared UI module is working before removing old code completely.
10. Everything in the new `App.kt` file should be working except for resource imports, which are now located in a
    different package.
    Reimport the `Res` object and all drawable resources with the correct path, for example:

    <compare type="top-bottom">
    <code-block lang="kotlin">
        import demo.composeapp.generated.resources.mx
    </code-block>
    <code-block lang="kotlin">
        import demo.sharedUI.generated.resources.mx
    </code-block>
    </compare>
11. Make the new `App()` composable available to the entry poins in the `composeApp` module.
    To do that, declare the dependency between `composeApp` and `sharedUI` in the `composeApp/build.gradle.kts` file:

     ```kotlin
     commonMain.dependencies {
         implementation(projects.sharedLogic)
         implementation(projects.sharedUI)
     }
     ```
12. Run your apps to check that the new module works to supply app entry points with shared UI code.
13. Remove the `composeApp/src/commonMain/.../App.kt` file.

You have successfully moved the cross-platform UI code to a dedicated module.
The only thing left is to create dedicated modules for every app you are producing with this project.


### Update the iOS integration

Since the iOS app entry point is not built as a separate Gradle module, you can embed the source code into any module.
In this example, `sharedUI` makes most sense:

1. Move the `composeApp/src/iosMain` directory into the `sharedUI/src` directory.
2. Configure the Xcode project to consume the framework produced by the `sharedUI` module:
    1. Select the **File | Open Project in Xcode** menu item.
    2. Click the **iosApp** project in the **Project navigator** tool window, then select the **Build Phases** tab.
    3. Find the **Compile Kotlin Framework** phase.
    4. Find the line starting with `./gradlew` and swap `composeApp` for `sharedUI`:

        ```text
        ./gradlew :sharedUI:embedAndSignAppleFrameworkForXcode
        ```
   
    5. Note that the import in the `ContentView.swift` file will stay the same, because it matches the `baseName` parameter from
       Gradle configuration of the iOS target,
       not actual name of the module.
       If you change the framework name in the `sharedUI/build.gradle.kts` file, you need to change the import directive accordingly.

3. Run the app from Xcode or using the **iosApp** run configuration in IntelliJ IDEA

### Remove `composeApp` and update the Android Gradle plugin version

When all code is working from correct new modules:

1. Remove the old module completely:
   1. Remove the `composeApp` dependency from the `settings.gradle.kts` file (the line `include(":composeApp")`).
   2. Remove the `composeApp` directory entirely.
   3. If you followed all instructions, you have working run configurations for the new project structure.
      You can remove old run configurations associated with the `composeApp` module.
2. In the `gradle/libs.versions.toml` file, update the AGP version to a 9.* version, for example:

    ```text
    [versions]
    agp = "9.0.0"
    ```
3. Remove this line from the `androidApp/build.gradle.kts` file, since [Kotlin support is built-in with AGP 9](https://developer.android.com/build/migrate-to-built-in-kotlin)
   and the Kotlin Android plugin is no longer necessary:

    ```kotlin
    alias(libs.plugins.kotlinAndroid)
    ```
4. Select **Build | Sync Project with Gradle Files** in the main menu, or click the Gradle refresh button in the
   build script editor.

5. That your apps build and run with the new AGP.

Congratulations, you modernized and optimized the structure of your project!

## What's next

TODO up for suggestions here — the first thought is to link platform-specific guidance.