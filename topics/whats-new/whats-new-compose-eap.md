[//]: # (title: What's new in Compose Multiplatform 1.6.10-beta01)

Here are the highlights for this EAP feature release:

* [Support for multimodule projects with Compose Multiplatform resources](#support-for-multimodule-projects-with-compose-multiplatform-resources)
* [Experimental navigation library](#experimental-navigation-library)
* [Lifecycle library with experimental common ViewModel](#lifecycle-library)
* [Known issues: bugs](#known-issues-bugs)

See the full list of changes for this release [on GitHub](https://github.com/JetBrains/compose-multiplatform/blob/master/CHANGELOG.md#1610-beta01-april-2024).

## Support for multimodule projects with Compose Multiplatform resources

In %composeEapVersion%-beta01, you can store resources in any Gradle module and in any source set, as well as publish projects and libraries
with resources included.

To enable multimodule support, update your project to Kotlin version 2.0.0-Beta5 or newer and Gradle 7.6 or newer.

## Experimental navigation library

The common navigation library, based on Jetpack Compose, is now available.
For details, see [the documentation](compose-navigation-routing.md).

Key limitations in this version:
* [Deep links](https://developer.android.com/guide/navigation/design/deep-link) (handling or following them) are not supported yet.
* The [BackHandler](https://developer.android.com/develop/ui/compose/libraries#handling_the_system_back_button) function
  and [predictive back gestures](https://developer.android.com/guide/navigation/custom-back/predictive-back-gesture)
  are supported only on Android.

## Lifecycle library

The common lifecycle library based on Jetpack lifecycle is now available, see [the docs](compose-lifecycle.md)
for details.

The library was primarily introduced to support common navigation functionality, but it also offers an experimental
cross-platform `ViewModel` implementation, and includes a common `LifecycleOwner` interface you can implement for your
projects.

A notable limitation of this version: Compose Multiplatform does not provide a general `ViewModelStoreOwner` implementation.
To use it beyond a `NavHost`, you'll need to provide your own implementation. The issue is going to be resolved in the stable
%composeEapVersion% release.

## Known issues: bugs

* A crash at startup on devices with iOS version older than 17 due to a [bug](https://github.com/JetBrains/compose-multiplatform/issues/4644).
* `inline fun <reified VM> viewModel(...)` is not available in common code due to a [compiler bug](https://github.com/JetBrains/compose-multiplatform/issues/3147).

  You can use this overload instead:

    ```kotlin
    fun <VM> viewModel(KClass, ...)
    ```