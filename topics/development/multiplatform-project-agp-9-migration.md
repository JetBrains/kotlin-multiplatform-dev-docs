[//]: # (title: Updating multiplatform projects with Android apps to use AGP 9)
<show-structure for="chapter,procedure" depth="3"/>

When used along with Android Gradle plugin 9.0 or newer,
the Kotlin Multiplatform Gradle plugin stops being compatible with the `com.android.application` and the `com.android.library` plugins.

To update your project:
* If your Android entry point is currently implemented in a shared code module,
  extract it into a separate module to avoid Gradle plugin conflicts.
* Migrate your shared code module to use the new [Android-KMP library plugin](https://developer.android.com/kotlin/multiplatform/plugin)
  built specifically for multiplatform projects.

> Android Studio supports AGP 9.0.0 starting with Otter 3 Feature Drop 2025.2.3.
> IntelliJ IDEA support for AGP 9.0.0 is expected in Q1 2026.
> 
{style="note"}

## Migrate to the Android-KMP library plugin

Previously, to configure an Android target in a multiplatform module you needed to use
the KMP plugin (`org.jetbrains.kotlin.multiplatform`)
together with either
the Android application (`com.android.application`) or the Android library (`com.android.library`) plugin.

With AGP 9.0, these plugins are no longer compatible with KMP,
so you need to migrate to the new Android-KMP library plugin built specifically for KMP.

> To make your project work with AGP 9.0 in the short term, you can manually enable the deprecated APIs.
> To do this, in the `gradle.properties` file of your project, add the following property:
> `android.enableLegacyVariantApi=true`.
>
> The legacy APIs are going to be [removed completely in AGP 10](https://developer.android.com/build/releases/gradle-plugin-roadmap#agp-10),
> which is likely to come out in the second half of 2026.
> Make sure you finish the migration before then.
>
{style="note"}

For library migration steps, see the [guide in Android documentation](https://developer.android.com/kotlin/multiplatform/plugin#migrate).

To migrate an Android app project, you need to have the Android entry point and the shared code in properly configured separate modules.
Below is a general tutorial for migrating a sample app, where you can see:
* [How to extract the Android app entry point into a separate module](#android-app)
* [How to update the configuration of a shared module](#configure-the-shared-module-to-use-the-android-kmp-library-plugin)

## Migration of a sample app

The example project that you will prepare for the migration is a Compose Multiplatform app that is the result of the
[](compose-multiplatform-new-project.md)
tutorial.
* The sample with the example of an app that needs to be updated is in the [main](https://github.com/kotlin-hands-on/get-started-with-cm/tree/main)
  branch of the sample repository.
* A final state of the app, with `androidApp` isolated, is available in the [new-project-structure](https://github.com/kotlin-hands-on/get-started-with-cm/tree/new-project-structure)
  branch.
  (There you can also see examples of isolated app modules for other platforms.)

<!-- When the new structure is implemented in the wizard, this is going to change: 
     following the tutorial will bring you to the new structure already.
     So when the update hits we update with the following:

The sample with an example of older structure is in the [old-project-structure](https://github.com/kotlin-hands-on/get-started-with-cm/tree/old-project-structure)
branch of the sample repository. -->

The sample consists of a single Gradle module (`composeApp`) that contains all the shared code and KMP entry
points,
and the `iosApp` project with the iOS-specific code and configuration.

To prepare for the AGP 9.0 migration, you will:

* [Extract the Android app entry point](#android-app) into a separate `androidApp` module.
* [Reconfigure the module with shared code](#configure-the-shared-module-to-use-the-android-kmp-library-plugin) (`composeApp`) to use the Android-KMP library plugin 

### Module for the Android app entry point {id="android-app"}

#### Create and configure the Android app module

To create an Android app module (`androidApp`):

1. Create the `androidApp` directory at the root of the project.
2. Inside that directory, create an empty `build.gradle.kts` file and the `src` directory.
3. Add the new module to project settings in the `settings.gradle.kts` file by adding this line at the end of the file:

    ```kotlin
    include(":androidApp")
    ```
4. Select **Build | Sync Project with Gradle Files** in the main menu, or click the Gradle refresh button in the
   editor.

#### Configure the build script for the Android app

Configure the Gradle build script for the new module:

1. In the `gradle/libs.versions.toml` file, add the Kotlin Android Gradle plugin to your version catalog:

    ```text
    [plugins]
    kotlinAndroid = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
    ```

2. In the `androidApp/build.gradle.kts` file, specify the plugins necessary for the Android app module:

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
   of the `composeApp` build script and add the dependency on the `composeApp` module itself. 
   In this example, the result should look like this:

   ```kotlin
   kotlin {
       dependencies { 
           implementation(projects.composeApp)
           implementation(libs.androidx.activity.compose)
           implementation(libs.compose.uiToolingPreview)
       }
   }
   ```

5. Copy the entire `android {}` block with Android-specific configuration from the `composeApp/build.gradle.kts`
   file to the `androidApp/build.gradle.kts` file. 

6. Copy the compiler options from the `androidTarget {}` block of the `composeApp/build.gradle.kts` file to 
   the `target {}` block of the `androidApp/build.gradle.kts` file:

    ```kotlin
    target {
        compilerOptions {
            jvmTarget.set(JvmTarget.JVM_11)
        }
    }
    ```

   > If there are any other plugins or properties set up in the `composeApp` build script, 
   > make sure to migrate those to the `androidApp` build script as well.
   >
   {style="note"}

7. Change the configuration of the `composeApp` module from an Android application to an Android library,
   since that's what it effectively becomes. In `composeApp/build.gradle.kts`:
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
   
8. Select **Build | Sync Project with Gradle Files** in the main menu, or click the Gradle refresh button in the
   editor.

#### Move the code and run the Android app

1. Move the `composeApp/src/androidMain` directory into the `androidApp/src/` directory,
   but mind the code that should remain cross-platform:
   
   * Entry point code, like `MainActivity.kt` in our sample, needs to be in the `androidApp` module to properly build the Android app.
   * All [expected and actual declarations](https://kotlinlang.org/docs/multiplatform/multiplatform-expect-actual.html)
     need to stay in the source sets of the common module (`composeApp` in our example) to be available for all platforms.
     As you set up the dependency of `androidApp` on `composeApp`, the declarations are going to be available in entry point
     code as well.
   
2. Rename the `androidApp/src/androidMain` directory to `main`.
3. If everything is configured correctly, the imports in the `androidApp/src/main/.../MainActivity.kt` file work
   and the code compiles.
4. When you're using IntelliJ IDEA or Android Studio, the IDE recognizes the new module and automatically creates a new
   run configuration, **androidApp**.
   If that doesn't happen, modify the **composeApp** Android run configuration manually:
   1. In the run configuration dropdown, select **Edit Configurations**.
   2. Find the **composeApp** configuration in the **Android** category.
   3. In the **General | Module** field, change `demo.composeApp` to `demo.androidApp`.
5. Start the new run configuration to make sure that the app runs as expected.
6. If everything works correctly, in the `composeApp/build.gradle.kts` file, remove the `kotlin.sourceSets.androidMain.dependencies {}` block.

You have extracted the Android entry point to a separate module.
Now update the common code module to use the new Android-KMP library plugin. 

### Configure the shared module to use the Android-KMP library plugin

To simply extract the Android entry point, you applied the `com.android.library` plugin for the shared `composeApp` module.
Now migrate to the new multiplatform library plugin:

1. In `gradle/libs.versions.toml`,
   add the Android-KMP library plugin to your version catalog:

    ```text
    [plugins]
    androidMultiplatformLibrary = { id = "com.android.kotlin.multiplatform.library", version.ref = "agp" }
    ```

2. In the `composeApp/build.gradle.kts` file, swap the old Android library plugin for the new one:

    <compare type="top-bottom">
        <code-block lang="kotlin">
            alias(libs.plugins.androidLibrary)
        </code-block>
        <code-block lang="kotlin">
            alias(libs.plugins.androidMultiplatformLibrary)
        </code-block>
    </compare>
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
            jvmTarget.set(JvmTarget.JVM_11)
        }
    
        androidResources {
            enable = true
        }
    }
    ```
5. Remove both the `android {}` block from the `composeApp/build.gradle.kts` file as it's replaced with the `kotlin.androidLibrary {}` configuration.
6. In the `dependencies {}` block, replace the `debugImplementation(libs.compose.uiTooling)` line with
   `androidRuntimeClasspath(libs.compose.uiTooling)` as the new Android KMP library plugin doesn't
   support build variants.
7. Select **Build | Sync Project with Gradle Files** in the main menu, or click the Gradle refresh button in the
   editor.
8. Check that the Android app is running as expected.

### Update the Android Gradle plugin version

When all of your code works with the new configuration:

1. If you followed the instructions, you have working run configurations for the new app modules.
      You can delete obsolete run configurations associated with the `composeApp` module.
2. In the `gradle/libs.versions.toml` file, update the AGP to a 9.* version, for example:

    ```text
    [versions]
    agp = "9.0.0"
    ```
3. Update the Gradle version in the `gradle/wrapper/gradle-wrapper.properties` file to at least 9.1.0:

    ```text
    distributionUrl=https\://services.gradle.org/distributions/gradle-9.1.0-bin.zip
    ```
4. Remove this line from the `androidApp/build.gradle.kts` file, since [Kotlin support is built-in with AGP 9.0](https://developer.android.com/build/migrate-to-built-in-kotlin)
   and applying the Kotlin Android plugin is no longer necessary:

    ```kotlin
    alias(libs.plugins.kotlinAndroid)
    ```
5. In the `composeApp/build.gradle.kts` file update the namespace in the `kotlin.androidLibrary {}` block
   so that it doesn't conflict with the app's namespace. For example:

    ```kotlin
    kotlin {
        androidLibrary {
            namespace = "compose.project.demo.composedemolibrary"
            // ...
    ```
   
6. Select **Build | Sync Project with Gradle Files** in the main menu, or click the Gradle refresh button in the
   build script editor.

7. Check that your app builds and runs with the new AGP version.

Congratulations! You have upgraded your project to be compatible with AGP 9.0.

<!-- Commented out for now
## What's next

Check out the [recommended project structure](multiplatform-project-recommended-structure.md)
which follows the logic of separating entry points for any app target you might have. -->