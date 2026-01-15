[//]: # (title: Preview your UI)

You can create _preview_ composables to be able to see your UI rendered in the IDE without running an emulator.
Previews are a [part of core functionality of Jetpack Compose](https://developer.android.com/develop/ui/compose/tooling/previews).

Compose Multiplatform first implemented a limited `@Preview` annotation as a custom library,
but with version 1.10.0 this implementation is deprecated as the original AndroidX annotation became fully multiplatform.

On this page, you can learn about possible project configurations that make previews for common code work
in IntelliJ IDEA and Android Studio.

## Preview setup

To add support for previews in your IDE, add necessary dependencies in the `build.gradle.kts` file of your KMP module:

* The annotation dependency for the `commonMain` source set.
* The tooling dependency on the classpath, whose declaration depends on the Android configuration.

The annotation dependency should point to [one of the `@Preview` implementations](#available-annotations), for example:

```kotlin
kotlin {
    sourceSets {
        commonMain.dependencies {
            implementation("org.jetbrains.compose.ui:ui-tooling-preview:1.10.0")
        }
    }
}
```

The tooling dependency should be declared in the root `dependencies {}` block in one of two ways,
depending on your [Android target configuration](#android-target-configurations):

* If using the `com.android.application` or `com.android.library` plugin:

    ```kotlin
    dependencies {
        debugImplementation("org.jetbrains.compose.ui:ui-tooling:1.10.0")
    }
    ```
* If using the `com.android.kotlin.multiplatform.library` plugin:

    ```kotlin
    dependencies {
        androidRuntimeClasspath("org.jetbrains.compose.ui:ui-tooling:1.10.0")
    }
    ```

## Possible configurations

Depending on the configuration of your project, there are several component combinations you can use to enable
Compose previews:

* Compose Multiplatform 1.9, with the old `@Preview` annotation and Android configured as `androidTarget {}`.
* Compose Multiplatform 1.10, with the old `@Preview` annotation and Android configured as `androidTarget {}`.
  You can upgrade to the 1.10 version without changing anything else.
* Compose Multiplatform 1.10, with the new `@Preview` annotation and Android configured as `androidTarget {}`.
* Compose Multiplatform 1.10, with the new `@Preview` annotation and Android configured as `androidLibrary {}` with AGP 9.

Read on to learn about the differences between annotations and Android configurations.

### Available annotations

There are two `@Preview` annotations available in Compose Multiplatform,
the old custom implementation, and the original implementation made multiplatform in version 1.10.0:

* `org.jetbrains.compose.ui.tooling.preview.Preview`
  * This was the first multiplatform implementation of the annotation, emulating the pure Android experience.
    It supports a limited number of parameters, but provides base preview functionality.
  * The necessary runtime dependency is `org.jetbrains.compose.components:components-ui-tooling-preview`.
* `androidx.compose.ui.tooling.preview.Preview`
  * This is the original Android Jetpack annotation, made multiplatform with Compose Multiplatform 1.10.
    It supports all parameters from the Android declaration in common code.
  * The necessary runtime dependency is `org.jetbrains.compose.ui:ui-tooling-preview`.

To use one of these annotations in your shared code, add the appropriate runtime dependency for your `commonMain`
source set, [as shown above](#preview-setup).

### Android target configurations

If your project uses Android Gradle plugin 8, the Kotlin Multiplatform part of the project should use either
the Android application (`com.android.application`) or the Android library (`com.android.library`) plugin,
and the Android configuration is contained in the `androidTarget {}` block of your `build.gradle.kts` file.

For Android Gradle plugin 9, there is the new [KMP Android library plugin](https://developer.android.com/kotlin/multiplatform/plugin)
(`com.android.kotlin.multiplatform.library`)
which introduces the `androidLibrary {}` block for Android configuration.
While it's possible to use this plugin with AGP 8, that combination has significant issues and is not recommended. 
