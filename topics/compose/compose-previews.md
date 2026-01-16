[//]: # (title: Compose UI previews)

You can create _preview_ composables to see your UI rendered in the IDE (IntelliJ IDEA and Android Studio) without running an emulator.
Previews are a [part of core functionality of Jetpack Compose](https://developer.android.com/develop/ui/compose/tooling/previews).

> To enable Compose previews in common code of your Kotlin Multiplatform project,
> you need an Android target, since previews rely on Android libraries.
> 
{style="note"}

Compose Multiplatform initially implemented a limited `@Preview` annotation as a custom library,
but starting with version 1.10.0, this implementation has been deprecated as the original AndroidX annotation is now fully multiplatform.

On this page, you can find:
* [how to enable previews](#preview-setup) in common code for different project configurations,
* [overview of supported combinations](#supported-configurations) of Compose Multiplatform, AGP, and annotations.

## Preview setup

To enable preview support in your IDE, add the necessary dependencies to the `build.gradle.kts` file of your KMP module:

1. The annotation dependency for the `commonMain` source set: the old or the new one, depending on the Compose Multiplatform version.
2. The tooling dependency on the classpath, whose declaration depends on the Android configuration.

The annotation dependency should point to one of the `@Preview` implementations, for example:

```kotlin
kotlin {
    sourceSets {
        commonMain.dependencies {
            // The new annotation, available in CMP 1.10.0 and later
            implementation("org.jetbrains.compose.ui:ui-tooling-preview:1.10.0")
            // To import the new annotation:
            // import androidx.compose.ui.tooling.preview.Preview

            // The old annotation, deprecated in CMP 1.10.0
            implementation("org.jetbrains.compose.components:components-ui-tooling-preview:1.10.0")
            // To import the old annotation:
            // import org.jetbrains.compose.ui.tooling.preview.Preview
        }
    }
}
```

The tooling dependency should be declared in the root `dependencies {}` block of the `build.gradle.kts` file in your KMP module
in one of two ways,
depending on your [Android target configuration](#android-target-configurations):

* If you're using the `com.android.application` or `com.android.library` plugin:

    ```kotlin
    dependencies {
        debugImplementation("org.jetbrains.compose.ui:ui-tooling:1.10.0")
    }
    ```
* If you're using the `com.android.kotlin.multiplatform.library` plugin:

    ```kotlin
    dependencies {
        androidRuntimeClasspath("org.jetbrains.compose.ui:ui-tooling:1.10.0")
    }
    ```

## Supported configurations

Depending on the version of your dependencies and the configuration style of your project, there are several supported 
combinations that you can use to enable Compose previews:

* Compose Multiplatform 1.9, with the old `@Preview` annotation and Android configured with `androidTarget {}`.
* Compose Multiplatform 1.10, with the old `@Preview` annotation and Android configured with `androidTarget {}`.
* Compose Multiplatform 1.10, with the new `@Preview` annotation and Android configured with `androidTarget {}`.
* Compose Multiplatform 1.10, with the new `@Preview` annotation and Android configured with `androidLibrary {}` with AGP 9.0.
  See our [AGP 9.0 migration guide](multiplatform-project-agp-9-migration.md) for details on upgrading your KMP apps.

> Support for AGP 9.0 is coming soon to IntelliJ IDEA, expected to arrive in Q1 2026.
>
{style="note"}

### Available annotations

There are two `@Preview` annotations available in Compose Multiplatform:

* `androidx.compose.ui.tooling.preview.Preview`
  * This is the original Android Jetpack annotation, made multiplatform with Compose Multiplatform 1.10.
    It supports all parameters from the Android declaration in common code.
  * The necessary runtime dependency is `org.jetbrains.compose.ui:ui-tooling-preview`.
  * This is the recommended annotation to use going forward. 
* `org.jetbrains.compose.ui.tooling.preview.Preview`
  * This was the first multiplatform implementation of the annotation, emulating the Android-only experience.
    It supports a limited number of parameters, but provides basic preview functionality.
  * The necessary runtime dependency is `org.jetbrains.compose.components:components-ui-tooling-preview`.
  * This annotation is now deprecated in Compose Multiplatform 1.10.

To use one of these annotations in your shared code, add the appropriate runtime dependency for your `commonMain`
source set, [as shown above](#preview-setup).

### Android target configurations

If your project uses Android Gradle plugin 8.x, the Kotlin Multiplatform part of the project should use either
the Android application (`com.android.application`) or the Android library (`com.android.library`) plugin,
and the Android configuration is contained in the `androidTarget {}` block of your `build.gradle.kts` file.

For Android Gradle plugin 9.0, there is a new [KMP Android library plugin](https://developer.android.com/kotlin/multiplatform/plugin)
(`com.android.kotlin.multiplatform.library`)
which introduces the `androidLibrary {}` block for Android configuration.
While it's also possible to use this plugin with AGP 8.x, that combination has known issues and is not recommended. 

> AGP 9.0 is supported in the latest stable Android Studio,
> but not supported in IntelliJ IDEA yet and expected to arrive in Q1 2026.
> For details on upgrading to AGP 9.0, see [our migration page](multiplatform-project-agp-9-migration.md).
> 
{style="note"}