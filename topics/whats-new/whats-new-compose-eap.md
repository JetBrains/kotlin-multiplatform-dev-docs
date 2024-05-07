[//]: # (title: What's new in Compose Multiplatform 1.6.10-rc01)

Here are the highlights for this EAP feature release:

* [Support for multimodule projects with Compose Multiplatform resources](#support-for-multimodule-projects-with-compose-multiplatform-resources)
* [Experimental navigation library](#experimental-navigation-library)
* [Lifecycle library with experimental common ViewModel](#lifecycle-library)
* [Known issues: bugs](#known-issues)

See the full list of changes for this release [on GitHub](https://github.com/JetBrains/compose-multiplatform/blob/master/CHANGELOG.md#1610-beta01-april-2024).

## Dependencies

* Gradle plugin `org.jetbrains.compose`, version 1.6.10-rc01. Based on Jetpack Compose libraries:
  * [Compiler 1.5.11](https://developer.android.com/jetpack/androidx/releases/compose-compiler#1.5.8)
  * [Runtime 1.6.7](https://developer.android.com/jetpack/androidx/releases/compose-runtime#1.6.1)
  * [UI 1.6.7](https://developer.android.com/jetpack/androidx/releases/compose-ui#1.6.1)
  * [Foundation 1.6.7](https://developer.android.com/jetpack/androidx/releases/compose-foundation#1.6.1)
  * [Material 1.6.7](https://developer.android.com/jetpack/androidx/releases/compose-material#1.6.1)
  * [Material3 1.2.1](https://developer.android.com/jetpack/androidx/releases/compose-material3#1.2.0)
* Lifecycle libraries `org.jetbrains.androidx.lifecycle:lifecycle-*:2.8.0-rc01`. Based on [Jetpack Lifecycle 2.8.0-rc01](https://developer.android.com/jetpack/androidx/releases/lifecycle#2.8.0-rc01).
* Navigation libraries `org.jetbrains.androidx.navigation:navigation-*:2.7.0-alpha04`. Based on [Jetpack Navigation 2.7.7](https://developer.android.com/jetpack/androidx/releases/navigation#2.7.7).

## Breaking changes

### New Compose compiler Gradle is required for Kotlin 2.0

Starting with Kotlin 2.0.0-RC2, Compose Multiplatform requires the Gradle plugin for the 2.0.0 version of the Compose compiler.
See [the migration guide](https://www.jetbrains.com/help/kotlin-multiplatform-dev/compose-compiler.html#migrating-a-compose-multiplatform-project)
for details.

## Across platforms

### Stable resource library

The bulk of [the resource library API](compose-images-resources.md) is now considered stable.

### Support for multimodule projects with Compose Multiplatform resources

Starting with Compose Multiplatform %composeEapVersion%-beta01,
you can store resources in any Gradle module and in any source set, as well as publish projects and libraries
with resources included.

To enable multimodule support, update your project to Kotlin version 2.0.0-Beta5 or newer and Gradle 7.6 or newer.

### Experimental navigation library

The common navigation library, based on Jetpack Compose, is now available.
For details, see [the documentation](compose-navigation-routing.md).

Key limitations in this version:
* [Deep links](https://developer.android.com/guide/navigation/design/deep-link) (handling or following them) are not supported yet.
* The [BackHandler](https://developer.android.com/develop/ui/compose/libraries#handling_the_system_back_button) function
  and [predictive back gestures](https://developer.android.com/guide/navigation/custom-back/predictive-back-gesture)
  are supported only on Android.

### Lifecycle library

The common lifecycle library based on Jetpack lifecycle is now available, see [the docs](compose-lifecycle.md)
for details.

The library was primarily introduced to support common navigation functionality, but it also offers an experimental
cross-platform `ViewModel` implementation, and includes a common `LifecycleOwner` interface you can implement for your
projects.

A notable limitation of this version: Compose Multiplatform does not provide a general `ViewModelStoreOwner` implementation.
To use it beyond a `NavHost`, you'll need to provide your own implementation. The issue is going to be resolved in the stable
%composeEapVersion% release.

### Support for Kotlin 2.0.0-RC2

Kotlin 2.0-RC2 came out along with the new Gradle plugin for the Compose compiler.
To use Compose Multiplatform with the latest compiler version, apply the plugin to the modules in your project
(see [the migration guide](https://www.jetbrains.com/help/kotlin-multiplatform-dev/compose-compiler.html#migrating-a-compose-multiplatform-project) for details).

## Known issues

### MissingResourceException

You may experience the `org.jetbrains.compose.resources.MissingResourceException: Missing resource with path: ...` error
after changing your Kotlin version from 1.9 to 2.0 (or the other way around).
To resolve this, delete the `build` directories in your project: this includes folders located in the root and module folders of your project.
