[//]: # (title: Adding Swift packages as dependencies to KMP modules)
<primary-label ref="alpha"/>

<tldr>
   Swift Package Manager fulfills the same role as <a href="multiplatform-cocoapods-overview.md">CocoaPods</a>:
   it lets you transparently orchestrate native iOS dependencies of your iOS app.
   Here you can learn how to set up an SPM dependency in your KMP project,
   and how to migrate your KMP setup from CocoaPods to SPM if necessary.
</tldr>

## Environment setup

Since SPM import functionality is still in the Alpha stage, you need to request an EAP Kotlin version to try it out.

To do that:

1. In your `settings.gradle.kts` file, add the development package repository for dependencies and plugins:

    ```kotlin
    dependencyResolutionManagement {
        repositories {
            maven("https://packages.jetbrains.team/maven/p/kt/dev")
            mavenCentral()
        }
    }
    
    pluginManagement {
        repositories {
            maven("https://packages.jetbrains.team/maven/p/kt/dev")
            mavenCentral()
            gradlePluginPortal()
        }
    }
    ```
   
2. In your version catalog, apply the EAP version of the Kotlin Multiplatform Gradle plugin, for example:
   TODO check the final release version

    ```text
    kotlin = "2.4.0-titan-214"
   
    [plugins]
    kotlin-multiplatform = "2.4.0-titan-214"
    ```

3. Sync the Gradle files and try adding a `kotlin.swiftPMDependencies {}` block to the `build.gradle.kts` file
   of your KMP module.
   If the `swiftPMDependencies` name cannot be resolved, add the following block to the root `build.gradle.kts` file
   to add the necessary symbols to the classpath directly:

    ```kotlin
    buildscript {
        dependencies.constraints {
            "classpath"("org.jetbrains.kotlin:kotlin-gradle-plugin:2.4.0-titan-214!!")
        }
    }
    ```
   
### Setup the KMP IDE plugin

If you are using the [Kotlin Multiplatform IDE plugin]() recommended for any KMP project,
explicitly specify the path to the iOS project that is built from the KMP module.

In the `build.gradle.kts` file where you call the `iosTarget.binaries.framework` API,
add the API call that sets the path:

```kotlin
kotlin {
    // Example of iOS targets configuration
    listOf(
        iosArm64(),
        iosSimulatorArm64(),
        iosX64(),
    ).forEach { iosTarget ->
            iosTarget.binaries.framework { 
                baseName = "Shared"
                isStatic = false
            } 
    }
      
    swiftPMDependencies { 
        // Specify the path to the .xcodeproj file that uses
        // the ':embedAndSignAppleFrameworkForXcode' integration
        xcodeProjectPathForKmpIJPlugin.set(
            layout.projectDirectory.file("../iosApp/iosApp.xcodeproj")
        )
    }
}

```

