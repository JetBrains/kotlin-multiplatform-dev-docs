[//]: # (title: What's new in Compose Multiplatform 1.6.10-rc02)

Here are the highlights for this EAP feature release:

* [Support for multimodule projects with Compose Multiplatform resources](#support-for-multimodule-projects-with-compose-multiplatform-resources)
* [Experimental navigation library](#experimental-navigation-library)
* [Lifecycle library with experimental common ViewModel](#lifecycle-library)
* [Known issues](#known-issues)

See the full list of changes for this release [on GitHub](https://github.com/JetBrains/compose-multiplatform/blob/master/CHANGELOG.md#1610-beta01-april-2024).

## Dependencies

* Gradle plugin `org.jetbrains.compose`, version 1.6.10-rc01. Based on Jetpack Compose libraries:
  * [Compiler 1.5.13](https://developer.android.com/jetpack/androidx/releases/compose-compiler#1.5.13)
  * [Runtime 1.6.7](https://developer.android.com/jetpack/androidx/releases/compose-runtime#1.6.7)
  * [UI 1.6.7](https://developer.android.com/jetpack/androidx/releases/compose-ui#1.6.7)
  * [Foundation 1.6.7](https://developer.android.com/jetpack/androidx/releases/compose-foundation#1.6.7)
  * [Material 1.6.7](https://developer.android.com/jetpack/androidx/releases/compose-material#1.6.7)
  * [Material3 1.2.1](https://developer.android.com/jetpack/androidx/releases/compose-material3#1.2.1)
* Lifecycle libraries `org.jetbrains.androidx.lifecycle:lifecycle-*:2.8.0-rc02`. Based on [Jetpack Lifecycle 2.8.0-rc01](https://developer.android.com/jetpack/androidx/releases/lifecycle#2.8.0-rc01).
* Navigation libraries `org.jetbrains.androidx.navigation:navigation-*:2.7.0-alpha05`. Based on [Jetpack Navigation 2.7.7](https://developer.android.com/jetpack/androidx/releases/navigation#2.7.7).

## Breaking changes

### New Compose compiler Gradle plugin is required for Kotlin 2.0.0

Starting with Kotlin 2.0.0-RC2, Compose Multiplatform requires the new Compose compiler Gradle plugin.
See [the migration guide](https://www.jetbrains.com/help/kotlin-multiplatform-dev/compose-compiler.html#migrating-a-compose-multiplatform-project)
for details.

## Across platforms

### Resources

#### Stable resource library

The bulk of [the resource library API](compose-images-resources.md) is now considered stable.

#### Support for multimodule projects with Compose Multiplatform resources

Starting with Compose Multiplatform 1.6.10-beta01,
you can store resources in any Gradle module and in any source set, as well as publish projects and libraries
with resources included.

To enable multimodule support, update your project to Kotlin version 2.0.0-Beta5 or newer and Gradle 7.6 or newer.

#### Configuration DSL for multiplatform resources

You can now fine-tune the `Res` class generation in your project: alter the modality and the assigned package for
the class, as well as choose conditions for generating it: always, never or only with explicit dependency on the
resource library.

See [the documentation section](compose-images-resources.md#configuration) for details.

#### Public function for generating resource URI

The new `getUri()` function allows you to pass platform-dependent URI of a resource to external libraries,
so that they can access the file directly.
See [documentation](compose-images-resources.md#accessing-resources-from-external-libraries) for details.

#### Plurals for string resources

You can now define plurals (quantity strings) along with other multiplatform string resources.
See [documentation](compose-images-resources.md#plurals) for details.

#### Support three-letter locales

[Language qualifiers](compose-images-resources.md#language-and-regional-qualifiers) now support alpha-3 (ISO 639-2)
codes for locales. 

#### Experimental byte array functions for images and fonts

You can try out two functions that allow fetching fonts and images as byte arrays:
`getDrawableResourceBytes` and `getFontResourceBytes`.
These functions are aimed to help with accessing multiplatform resources from third-party libraries.

See details in the [pull request](https://github.com/JetBrains/compose-multiplatform/pull/4651).

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

Compose Multiplatform also provides a general `ViewModelStoreOwner` implementation.

### Support for Kotlin 2.0.0

Kotlin 2.0.0-RC2 came out along with the new Gradle plugin for the Compose compiler.
To use Compose Multiplatform with the latest compiler version, apply the plugin to the modules in your project
(see [the migration guide](https://www.jetbrains.com/help/kotlin-multiplatform-dev/compose-compiler.html#migrating-a-compose-multiplatform-project) for details).

## Desktop

### Basic support of BasicTextField2

The `BasicTextField2` Compose component is now supported on a base level for desktop targets.
Use it if your project absolutely requires it, or to test it out, but keep in mind that there may be uncovered edge cases.
For example, `BasicTextField2` does not support IME events right now, so you won't be able to use virtual keyboards
for Chinese, Japanese, or Korean.

Full support for the component and support for other platforms is planned in the Compose Multiplatform 1.7.0 release.

### alwaysOnTop flag for DialogWindow

To avoid your dialog windows being overwritten, you can now use the `alwaysOnTop` flag for a `DialogWindow`
composable.

See the [pull request](https://github.com/JetBrains/compose-multiplatform-core/pull/1120) for details.

## iOS

### Accessibility support improvements

In this release:

* dialogs and popups are properly integrated with accessibility features,
* interop views created using `UIKitView` and `UIKitViewController` are now reachable by Accessibility Services,
* `LiveRegion` semantics are supported by the accessibility API,
* [accessibility scrolling](https://github.com/JetBrains/compose-multiplatform-core/pull/1169) is supported,
* `HapticFeedback` is supported.

### Selection container magnifier for iOS 17 and higher

Compose Multiplatform selection containers on iOS now emulate the native magnifying tool.

![Screenshot of iPhone chat app with the text magnifier active](compose-1610-ios-magnifier.png){width=390}


### Software keyboard inset for Dialog centering

Behavior of `Dialog` composables is now aligned with Android: when a software keyboard appears on screen, dialogs
are centered considering the effective height of the application window.
There is an option to disable this, with the `DialogProperties.useSoftwareKeyboardInset` property.

## Web

### Basic IME keyboard support

The web target for Compose Multiplatform now has basic support for virtual (IME) keyboards.

## Gradle plugin

### Possibility to modify the macOS minimum version

In previous versions, it wasn't possible to upload a macOS app to the App Store without including an Intel version. 
You can now set a minimum macOS version for your app among platform-specific Compose Multiplatform options:

```kotlin
compose.desktop {
    application {
        nativeDistributions {
            macOS {
                minimumSystemVersion = "12.0"
            }
        }
    }
}
```

See the [pull request](https://github.com/JetBrains/compose-multiplatform/pull/4271) for details.

### Option to create uber JARs with Proguard support

You can now create uber JARs (complex packages with JARs of the application and all dependencies) using ProGuard
Gradle tasks.

See the [pull request](https://github.com/JetBrains/compose-multiplatform/pull/4136) for details.

<!--TODO add link to the GitHub tutorial mentioned in PR when it's updated  -->

## Known issues

### MissingResourceException

You may experience the `org.jetbrains.compose.resources.MissingResourceException: Missing resource with path: ...` error
after changing your Kotlin version from 1.9.x to 2.0.0 (or the other way around).
To resolve this, delete the `build` directories in your project: this includes folders located in the root and module folders of your project.

### NativeCodeGeneratorException

iOS compilation might fail for some projects with the following error:

```
org.jetbrains.kotlin.backend.konan.llvm.NativeCodeGeneratorException: Exception during generating code for following declaration: private fun $init_global()
```

Follow the [GitHub issue](https://github.com/JetBrains/compose-multiplatform/issues/4809) for details. 