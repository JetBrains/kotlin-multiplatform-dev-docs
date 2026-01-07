[//]: # (title: Recommended Kotlin Multiplatform project structure)
<show-structure for="chapter,procedure" depth="3"/>

The overviews of [basic](multiplatform-discover-project.md) and [advanced](multiplatform-advanced-project-structure.md)
project structure concepts should give you and understanding of source sets and dependency management.
What about modules which organize the source sets and rely on the dependencies?

> The article talks specifically about KMP projects.
> For a general understanding of modularization decision-making, see the [Android introduction to modularization](https://developer.android.com/topic/modularization).

## Optimal module structure

The optimal module structure can vary depending on your goals and necessary targets.
You can analyze the output of the [KMP IDE plugin wizard]() with different configurations and sets of targets
to see how we organize projects by default.

The general approach can be outlined as follows:
* The entry points for your apps should be contained in separate modules, each of which depends on necessary shared code modules.
* The shared code is generally divided into business logic and UI, and the strategy is to avoid unnecessary dependencies:
  * If all of your apps produced by the KMP project are using shared UI code as well as shared business logic,
    a single `shared` module for all of your shared code can be sufficient.
  * If UI for any one of your apps is written using native code (for example, you implemented the iOS UI in pure Swift),
    it makes sense to separate UI code from business logic to avoid Compose Multiplatform dependencies where they are not needed.
    So you can have a `sharedLogic` and a `sharedUI` module, and add them as dependencies to entry point modules as needed.
* If your project includes server code which should share logic with client apps, the recommended way to structure it is:
  * An `app` folder with entry point modules and client common code modules organized as described above.
  * A `server` module with the server-specific code.
  * A `core` module for code shared between the server and clients, such as models and validation.

If your project uses older structure, with app entry points and shared code contained in a single module,
you can follow guides below to extract the entry points into separate modules.

> It is mandatory to separate your Android app entry points from common code if you intend to use Android Gradle Plugin 9 or newer.
> See our [AGP 9 migration article](multiplatform-project-agp-9-migration.md) for details.
> 
{style="note"}

## Creating separate modules for app entry points

The example project that we will use to illustrate for a transition to the recommended structure is an older Compose Multiplatform sample
which can be found in the [old-project-structure](https://github.com/kotlin-hands-on/get-started-with-cm/tree/old-project-structure)
branch of the sample repository.

The example consists of a single Gradle module (`composeApp`) that contains all the shared code and KMP entry points,
and the `iosApp` folder with the iOS project code and configuration.

To extract an entry point to its own module, you need to create the module, move the code, and adjust
configurations accordingly both for the new module and the common code module.

<include from="multiplatform-project-agp-9-migration.md" element-id="android-app"/>

### Desktop JVM app

#### Create and configure the desktop app module

To create a desktop app module (`desktopApp`):

1. Create the `desktopApp` directory at the root of the project.
2. Inside that directory, create an empty `build.gradle.kts` file and the `src` directory.
3. Add the new module to project settings in the `settings.gradle.kts` file by adding this line:

    ```kotlin
    include(":desktopApp")
    ```

#### Configure the build script for the desktop app

To make the desktop app build script work:

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
    }
    ```

3. Make sure all of these plugins are mentioned in the **root** `build.gradle.kts` file:

    ```kotlin
    plugins {
        alias(libs.plugins.kotlinJvm) apply false
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

#### Move the code and run the desktop app

After the configuration is complete, move the code of the desktop app to the new directory:

1. In the `desktopApp/src` directory, create a new `main` directory.
2. Move the `composeApp/src/jvmMain/kotlin` directory into the `desktopApp/src/main/` directory:
   It's important that the package coordinates are aligned with the `compose.desktop {}` configuration.
3. If everything is configured correctly, the imports in the `desktopApp/src/main/.../main.kt` file are working
   and the code is compiling.
4. To run your desktop app, modify the **composeApp [jvm]** run configuration:
   1. In the run configuration dropdown, select **Edit Configurations**.
   2. Find the **composeApp [jvm]** configuration in the **Gradle** category.
   3. In the **Gradle project** field, change `ComposeDemo:composeApp` to `ComposeDemo:desktopApp`.
5. Start the updated configuration to make sure that the app runs as expected.
6. If everything works correctly:
   * Delete the `composeApp/src/jvmMain` directory.
   * In the `composeApp/build.gradle.kts` file, remove the desktop-related code:
       * the `compose.desktop {}` block,
       * the `jvmMain.dependencies {}` block inside the Kotlin `sourceSets {}` block,
       * the `jvm()` target declaration inside the `kotlin {}` block.

### Web app

#### Create and configure the web app module

To create a desktop app module (`webApp`):

1. Create the `webApp` directory at the root of the project.
2. Inside that directory, create an empty `build.gradle.kts` file and the `src` directory.
3. Add the new module to project settings in the `settings.gradle.kts` file by adding this line at the end of the file:

    ```kotlin
    include(":webApp")
    ```

#### Configure the build script for the web app

To make the desktop app build script work:

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

#### Move the code and run the web app

After the configuration is complete, move the code of the web app to the new directory:

1. Move the entire `composeApp/src/webMain` directory into the `webApp/src` directory.
   If everything is configured correctly, the imports in the `webApp/src/webMain/.../main.kt` file are working
   and the code is compiling.
2. In the `webApp/src/webMain/resources/index.html` file update the script name: from `composeApp.js` to `webApp.js`.
3. To run your web app, modify the **composeApp [wasmJs]** run configuration:
    1. In the run configuration dropdown, select **Edit Configurations**.
    2. Find the **composeApp [wasmJs]** configuration in the **Gradle** category.
    3. In the **Gradle project** field, change `ComposeDemo:composeApp` to `ComposeDemo:webApp`.
4. Repeat for **composeApp [js]** to be able to run the JavaScript version, too.
5. Start the run configurations to make sure that the app runs as expected.
6. If everything works correctly:
    * Delete the `composeApp/src/webMain` directory.
    * In the `composeApp/build.gradle.kts` file, remove the web-related code:
        * the `webMain.dependencies {}` block inside the Kotlin `sourceSets {}` block,
        * the `js {}` and `wasmJs {}` target declarations inside the `kotlin {}` block.

### Configure the shared module

In the example app, both UI and business logic code are being shared, so it only needs a single shared module
to hold all common code: you can simply repurpose `composeApp` as the common code module.

[//]: # (TODO For an overview of other project configurations and ways of dealing with them, see our blogpost about the new recommended project structure [link])

The only thing you need to adjust in the Gradle configuration that is not related to connections with entry point modules
is the new Android Library Gradle plugin.
The new plugin is built specifically for multiplatform projects and is required to use AGP 9 and newer.

Here are the necessary changes:

1. In `gradle/libs.versions.toml`,
   add the Android-KMP library plugin to your version catalog:

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
    }
    ```
3. In the root `build.gradle.kts` file, add the following line to avoid conflicts in applying the plugin:

    ```kotlin
    alias(libs.plugins.androidMultiplatformLibrary) apply false
    ```
4. In the `composeApp/build.gradle.kts` file, instead of the `kotlin.androidTarget {}` block add a `kotlin.androidLibrary {}` block:

    ```kotlin
    androidLibrary {
        namespace = "compose.project.demo.composedemo"
        compileSdk = libs.versions.android.compileSdk.get().toInt()
    
        compilerOptions {
            jvmTarget = JvmTarget.JVM_11
        }
    
        androidResources {
            enable = true
        }
    }
    ```
5. Remove the root `android {}` block from the `composeApp/build.gradle.kts` file.
6. Remove the `androidMain` dependencies, since all code was moved to the app module:
   delete the `kotlin.sourceSets.androidMain.dependencies {}` block.
7. Check that the Android app is running as expected.

### (Optional) Separate shared logic and shared UI {collapsible="true"}

If some of the targets in your project implement native UI, it may be a good idea to separate common code
into `sharedLogic` and a `sharedUI` modules, so that app modules with native UI don't need to depend on Compose Multiplatform
to use shared code.

Below is an example of how you can approach this, based on the same sample app.

#### Create a shared logic module

Before actually creating a module, you need to decide on what is business logic, which code is both UI- and platform-independent.
In this example, the only candidate is the `currentTimeAt()` function, which returns the exact time for a pair of
location and time zone.
By contrast, the `Country` data class relies on `DrawableResource` from Compose Multiplatform and cannot be separated from UI code.

> If your project already has a `shared` module, for example, because you don't share all UI code,
> then you can use this module instead of `sharedLogic`.
> It may be nice to rename it to clearer distinguish shared logic from UI.
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

    1. In the `gradle/libs.versions.toml` file, add the Android-KMP library plugin to your version catalog:

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
    4. In the `sharedLogic/build.gradle.kts` file, specify the targets that the common module should support in this example:

        ```kotlin
        kotlin {
            // There's no need for iOS framework configuration since sharedLogic
            // is not going to be exported as a framework, only 'sharedUI' is.
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
7. Run **Build | Sync Project with Gradle Files** again to apply the changes.
8. In the `composeApp/commonMain/.../App.kt` file, import the `currentTimeAt()` function to fix the code.
9. Run the application to make sure that your new module functions properly.

You have successfully isolated the shared logic into a separate module and used it cross-platform.
Next step: creating a shared UI module.

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
       add the Android-KMP library plugin to your version catalog:

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
        }
        ```

    3. Make sure all of these plugins are mentioned in the **root** `build.gradle.kts` file:

        ```kotlin
        plugins {
            alias(libs.plugins.androidMultiplatformLibrary) apply false
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
                namespace = "com.jetbrains.greeting.demo.sharedUI"
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
9. Temporarily comment out all code in the old `App.kt` file.
   This will allow you to test whether the shared UI module is working before removing the old code completely.
10. The new `App.kt` file should work as expected, except for resource imports, which are now located in a
    different package.
    Reimport the `Res` object and all drawable resources with the correct path, for example:

    <compare type="top-bottom">
    <code-block lang="kotlin">
        import demo.composeapp.generated.resources.mx
    </code-block>
    <code-block lang="kotlin">
        import demo.sharedui.generated.resources.mx
    </code-block>
    </compare>
11. To make the new `App()` composable available to the entry points in your app modules that rely on it,
    add the dependency to the corresponding `build.gradle.kts` files:

    ```kotlin
    kotlin {
        sourceSets {
            commonMain.dependencies {
                implementation(projects.sharedUI)
                // ...
            }
        }
    }
    ```
12. Run your apps to check that the new module works to supply app entry points with shared UI code.
13. Remove the `composeApp/src/commonMain/.../App.kt` file.

You have successfully moved the cross-platform UI code into a dedicated module.

### Update the iOS integration

Since the iOS app entry point is not built as a separate Gradle module, you can embed the source code into any module.
In this example, you can leave it inside `shared`:

1. Move the `composeApp/src/iosMain` directory into the `shared/src` directory.
2. Configure the Xcode project to consume the framework produced by the `shared` module:
    1. Select the **File | Open Project in Xcode** menu item.
    2. Click the **iosApp** project in the **Project navigator** tool window, then select the **Build Phases** tab.
    3. Find the **Compile Kotlin Framework** phase.
    4. Find the line starting with `./gradlew` and replace `composeApp` with `sharedUi`:

        ```text
        ./gradlew :shared:embedAndSignAppleFrameworkForXcode
        ```
   
    5. Note that the import in the `ContentView.swift` file needs to stay the same, because it matches the `baseName` parameter from
       Gradle configuration of the iOS target,
       not actual name of the module.
       If you change the framework name in the `shared/build.gradle.kts` file, you need to change the import directive accordingly.

3. Run the app from Xcode or using the **iosApp** run configuration in IntelliJ IDEA

[//]: # (TODO ## What's next: up for suggestions here — the first thought is to link platform-specific guidance)