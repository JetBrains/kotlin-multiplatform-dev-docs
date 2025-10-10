[//]: # (title: Migrating a Kotlin Multiplatform project to support AGP 9.0)

Kotlin Multiplatform projects with Android targets which were created using the Android Gradle plugin version earlier than 9.0
need to be restructured to upgrade to AGP 9.0.
Several APIs needed for KMP configuration are hidden in AGP 9.0 and eventually are going to be removed.
To solve this in the long term, we recommend updating your project structure to isolate AGP usage to an Android module.

On top of AGP compatibility, there is an issue of migrating to the new [Android Gradle Library plugin](https://developer.android.com/kotlin/multiplatform/plugin).
In the following guide, we highlight changes related to this as well.

> To make your project work with AGP 9.0 in the short term, you can manually enable the hidden APIs.
> TODO: explain how to do that.
> 
{style="note"}

## Migration guide

In this guide, we show how to restructure a combined multiplatform module into discrete modules clearly delineating
shared logic, shared UI, and individual entry points.

The example project is a Compose Multiplatform app that is the result of the [](compose-multiplatform-new-project.md)
tutorial.
You can check out the initial state of the project in the [update_october_2025 branch](https://github.com/kotlin-hands-on/get-started-with-cm/tree/update_october_2025)
of the sample project.

The example consists of a single Gradle module (`composeApp`) that contains all the shared code and all of the KMP entry points.
You will extract shared code and entry points into separate modules to reach two goals:

* Create a more flexible and scalable project structure that allows managing shared logic, shared UI, and different entry points
  separately.
* Isolate the Android module (that uses the `androidApplication` Gradle plugin) from KMP modules (that use the `androidLibrary`
  Gradle plugin).

For general modularization advice, see [Android modularization intro](https://developer.android.com/topic/modularization).
In these terms, you are going to create several **app modules**, for each platform, and shared **feature modules**, for UI and business logic.

> If your project is simple enough, it might suffice to combine all shared code (shared logic and UI) in a single module.
> We'll separate them to illustrate the modularisation pattern.
>
{style="note"}

### Create a shared logic module

Before actually creating a module, you need to decide on what is business logic, which code is both UI- and platform-independent.
In this example, the only clear candidate is the `currentTimeAt()` function that returns exact time for a pair of location and time zone.
The `Country` data class, for example, relies on `DrawableResource` from Compose Multiplatform and can't be separated from UI code.

Isolate the corresponding code in a `sharedLogic` module:

1. Create the `sharedLogic` directory at the root of the project.
2. Inside that directory, create the `src` directory and an empty `build.gradle.kts` file.
3. Add the new module to `settings.gradle.kts` by adding this line at the end of the file:

    ```kotlin
    include(":sharedLogic")
    ```
4. Configure the Gradle build script for the new module.

   1. In `gradle/libs.versions.toml`, add the Android Gradle Library plugin to your version catalog:

       ```text
       [plugins]
       androidMultiplatformLibrary = { id = "com.android.kotlin.multiplatform.library", version.ref = "agp" }
       ```
   
   2. In `sharedLogic/build.gradle.kts`, specify the plugins necessary for the shared logic module:
         
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
           // There's no need for framework configuration since sharedLogic is not going to be exported as a framework,
           // only sharedUi is.
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
   5. Add the `androidLibrary` configuration to the `kotlin {}` block. This is basically an `androidApplication` configuration
      with unnecessary parts removed:

       ```kotlin
       kotlin {
           // ...
           androidLibrary {
               namespace = "com.jetbrains.greeting.demo.sharedLogic"
               compileSdk = libs.versions.android.compileSdk.get().toInt()
      
               defaultConfig {
                   minSdk = libs.versions.android.minSdk.get().toInt()
               }     
             
               compilerOptions {
                   sourceCompatibility = JavaVersion.VERSION_11
                   targetCompatibility = JavaVersion.VERSION_11
               }
           }
       }
       ```

5. Move the business logic code identified in the beginning:
   1. Create a `commonMain/kotlin` directory inside `sharedLogic/src`.
   2. Inside `commonMain/kotlin` create the `CurrentTime.kt` file.
   3. Copy the `currentTimeAt` function to that file. You'll see that imports are missing: there are no dependencies
      yet declared for the `sharedLogic` module.
6. In `sharedLogic/build.gradle.kts`, add the necessary time dependencies for the common and JavaScript source sets in the same
   way they are declared for `composeApp`:

```kotlin
kotlin {
    sourceSets {
        commonMain.dependencies {
            implementation("org.jetbrains.kotlinx:kotlinx-datetime:0.7.1")
        }
        wasmJsMain.dependencies {
            implementation(npm("@js-joda/timezone", "2.22.0"))
        }
    }
}
```

   1.  `currentTimeAt`, but not `Country` which depends on CMP resources.
   2. Add the dependency for datetime, import the imports.
   3. Depend composeApp on sharedLogic to properly reference the function in its new place.
6. Greeting and Platform are not used, but they are strictly speaking logic, so let's move them to sharedLogic. 
   7. expect-actuals will break, so we need to copy the actuals to sharedLogic as well — but not entry points.
   8. So delete everything except the necessary actuals.
   9. webMain is not necessary at all, because it only serves web entry points.
10. Sync Gradle check that there are no expect-actual errors.

### Create a shared UI module

You will isolate code implementing shared business logic in a `sharedUi` module:

1. Create the `sharedUi` directory at the root of the project.
2. Inside that directory, create the `src` directory and an empty `build.gradle.kts` file.
3. Add the new module to `settings.gradle.kts`.
4. TODO removing unneeded lines from the old script seems like more work.
   Probably easier to give a ready-made script and comment on the changes.
   The most notable change is androidApplication → androidLibrary.
   Also we don't have any platform-specific code here, so we can remove the platform-specific dependencies.
   Maybe notable changes should be summarized in the beginning.
5. Move resource files to sharedUi.
6. Now we move code.
    1. Create commonMain, create App.kt with an intermediary App2() function.
    1. Move everything that was left after removing logic.
    2. Reimport the resources.
    3. Change the function call to App2() at entry points, check that everything is working. 
4. Remove the App.kt file in composeApp.
5. Change the function name App2 → App.

### Create modules for each app entry point

As stated in the beginning of this page, the only module that needs isolating is the Android app entry point.
But if you have other targets enabled, to make the structure transparent it's more straightforward to keep all of them
on the same level of the project hierarchy.

#### androidApp

1. Navigate to the composeApp/src/androindMain directory.
1. Delete Platform.android.kt, since you moved it to sharedLogic already.
2. Create a new `androidApp` directory at the root.
2. Inside that directory, create the `src` directory and an empty `build.gradle.kts` file.
3. For the build script:
   4. kotlinAndroid, androidApplication, composeMultiplatform, composeCompiler, no HotReload
4. There is no Kotlin Android plugin explicitly defined, so add it to gradle/libs.versions.toml.
5. And add the Kotlin Android plugin to the root build.gradle.kts.
1. sharedUi and sharedLogic dependencies (not `implementation(project((":sharedUi")))`, but `implementation(project.sharedUi)`
6. To androidApp/build.gradle.kts, copy everything Android-specific:
   7. activity dependency
   8. uiToolingPreview 
   8. android {} root block
   9. Make sure that kotlin {} and android {} have the same JVM version to avoid mismatch.
7. Copy the entire androidMain to androidApp/src
9. Rename androidMain to `main`.
8. Sync Gradle.
9. Run MainActivity from the gutter.
10. Remove androidMain from composeApp.

#### desktopApp

1. Navigate to the composeApp/src/jvmMain directory.
1. Delete Platform.jvm.kt, since you moved it to sharedLogic already.
2. Create a new `desktopApp` directory at the root.
2. Inside that directory, create the `src` directory and an empty `build.gradle.kts` file.
3. Register the desktopApp module in the root settings.gradle.kts.
3. For the build script, we keep all plugins — Hot Reload, Compose, and JVM.
   4. There is no JVM plugin explicitly defined, so you need to add it to gradle/libs.versions.toml.
   5. And apply the JVM plugin in the root build.gradle.kts.
6. Create main/kotlin inside src.
7. Copy the package with main.kt there.
9. Copy the compose.desktop {} block (here it's important that you kept the package name the same).
9. Add the necessary dependencies to the build script (copy them from jvmMain.dependencies in composeApp).
8. sharedUi and sharedLogic (not `implementation(project((":sharedUi")))`, but `implementation(project.sharedUi)`
10. Run the desktop app from the gutter.
11. Remove the jvmMain directory from composeApp.
12. Remove the `jvm()` target from composeApp and corresponding dependencies from sourceSets{} (23:49).

#### webApp

1. You don't need to keep composeApp/jsMain and composeApp/wasmMain at all, since all they contain is actuals for Platform.
   Delete the corresponding directories.
2. Create a new `webApp` directory at the root.
2. Inside that directory, create the `src` directory and an empty `build.gradle.kts` file.
3. Register the webApp module in the root settings.gradle.kts.
3. For the build script, we keep all plugins — Multiplatform and Compose.
    4. Copy js and wasmJs targets into the new build.gradle.kts.
6. Copy webMain to webApp/src
7. Copy dependencies for webMain, and depend on sharedUi and sharedLogic + compose.ui.
8. In the webMain/resources/index.html file, rename the script to webApp.js, to reflect the new module that will be compiled into JavaScript.
8. Run the app.
9. Delete the webMain directory in the composeApp folder.

### Update the iOS integration

For iOS, you don't need a separate module, so the source set can be embedded in `sharedUi`: 

1. Move the `iosMain` directory into the `sharedUi/src` directory.
2. Open the 
2. Then rewrite the build script in Xcode to use the Gradle task for the correct module:
   3. Open Xcode project for iosApp.
   4. Navigate to Build Phases.
   5. Find the Compile Kotlin Framework step.
   6. Change `./gradlew :composeApp:...` to `./gradlew :sharedUi:...`
   7. Note that the import in ContentView.swift will stay the same, because it matches the baseName parameter from Gradle configuration,
      not actual name of the module.

### Remove composeApp

Now that me moved all code outside composeApp, check that there are no dependencies left. Then remove composeApp entirely.

If you already have a `shared` module, then this will be the `sharedLogic` module,  

## Anything else?