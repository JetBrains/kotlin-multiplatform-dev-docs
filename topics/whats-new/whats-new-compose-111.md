[//]: # (title: What's new in Compose Multiplatform 1.11.0-beta02)

Here are the highlights for this EAP feature release:

* [Native iOS text input](#native-text-input)
* [v2 version of Compose UI testing API](#compose-ui-tests-v2)
* [Improved scroll on web targets](#scroll-on-web-targets-brought-in-line-with-native-ui)

You can find the full list of changes on [GitHub](https://github.com/JetBrains/compose-multiplatform/releases/tag/v1.11.0-beta02).
Specific component versions for this release are listed in the [Dependencies](#dependencies) section.

## Breaking changes and deprecations

### Shader wrapper for non-Android targets

For non-Android targets, the `Shader` type has been refactored from an `actual typealias` of `org.jetbrains.skia.Shader` 
to a Compose-specific wrapper class. This change decouples the common API from direct Skia/Skiko dependencies.

Migration steps:

* To use Skia/Skiko shaders in Compose APIs, wrap them using `SkShader.asComposeShader()`.
* To access low-level Skia types from a Compose `Shader`, use the `Shader.skiaShader` extension property.
* If you use third-party libraries that rely on the `Shader` API, update them to newer compatible versions.

### Minimum Kotlin version increased

If your project includes native or web targets, the latest features require an upgrade to Kotlin 2.3.10.

### Dropped support for Apple x86_64 targets

Compose Multiplatform no longer supports Apple x86_64 targets, as they were deprecated in Kotlin. 
As a result, the `iosX64` and `macosX64` targets have been fully removed from all modules.

### Deprecations

* In Compose Multiplatform 1.9.0, we [introduced the `WebElementView`](https://kotlinlang.org/docs/multiplatform/whats-new-compose-190.html#new-api-for-embedding-html-content) 
  composable to seamlessly integrate HTML elements into your web application. 
  It turned out that the chosen name was a bit unclear, and we renamed it to `HtmlElementView` to better reflect its 
  HTML-specific purpose. The `WebElementView` version has been deprecated in favor of `HtmlElementView`.
* `Key.Home` has been deprecated as it was incorrectly mapped. 
  Use `Key.MoveHome` for keyboard navigation or `Key.SystemHome` for system-level actions.

## Across platforms

### Compose UI tests v2

Compose Multiplatform introduces support for [v2 `ComposeUiTest` APIs](https://developer.android.com/develop/ui/compose/testing/migrate-v2) 
on non-Android targets. These new APIs use `StandardTestDispatcher` as the default test dispatcher instead of `UnconfinedTestDispatcher`. 
This change ensures coroutines are executed in order based on the event queue, improving test reliability and 
alignment with production behavior.

We've also added support for the `effectContext` parameter in Compose UI tests v2.
This parameter allows you to specify a custom coroutine context when running compositions.

The previously offered test APIs, such as `runComposeUiTest`, `runSkikoComposeUiTest`, and `runDesktopComposeUiTest`, 
are now deprecated in favor of their v2 versions.

### Skia updated to Milestone 144

The version of Skia used by Compose Multiplatform, via Skiko, has been updated to Milestone 144.

The previous version of Skia used was Milestone 138.
You can see the changes made between these versions in the [release notes](https://skia.googlesource.com/skia/+/refs/heads/chrome/m144/RELEASE_NOTES.md).

## iOS

### Native text input
<primary-label ref="Experimental"/>

Compose Multiplatform introduces a new text input implementation that uses native iOS `UIView` to manage input via 
the `UITextInput` and `UIKeyInput` protocols. This enables fully native iOS text editing behavior, including precise 
caret movement, native gestures, native selection handling, and system context menus with items 
like `Autofill`, `Translate`, and `Search`. The new approach aligns with the native iOS look and feel while also 
improving compatibility with future Apple updates.

While the existing Compose Multiplatform text input remains the stable choice for consistency across platforms, 
the native approach focuses on a user experience specifically tailored for iOS.

To enable the new text input, use the `usingNativeTextInput` option in the iOS source set:

```kotlin
@ExperimentalComposeUiApi
BasicTextField(
    value = state,
    keyboardOptions = KeyboardOptions(
        platformImeOptions = PlatformImeOptions {
            usingNativeTextInput(true)
        }
    )
)
```

The new native text input supports both the `BasicTextField(TextFieldValue)` and `BasicTextField(TextFieldState)` APIs, 
and is also compatible with the new context menu API enabled via the `isNewContextMenuEnabled` flag.

## Web

### Scroll on web targets brought in line with native UI

In Compose Multiplatform, scrolling performance on the web has been lagging behind that of native UIs. 
For the 1.11.0 release, a lot has been reworked and fixed in touch processing, bringing scrolling for Compose web apps 
in line with the other available targets. You can see the effect of these improvements in the 
latest [web version of the KotlinConf App](https://jetbrains.github.io/kotlinconf-app/).

As part of this work, [Coil image decoding on web has also been improved](https://github.com/coil-kt/coil/pull/3305). 
If you use Coil, make sure to update to version 3.4.0 for the best experience.

The list of fixes, along with the explanations and demos of improvements, is available in the issue [CMP-9727](https://youtrack.jetbrains.com/issue/CMP-9727).

## Dependencies

| Library            | Maven coordinates                                                              | Based on Jetpack version                                                                                                             |
|--------------------|--------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------|
| Runtime            | `org.jetbrains.compose.runtime:runtime*:1.11.0-beta02`                         | [Runtime 1.11.0-beta02](https://developer.android.com/jetpack/androidx/releases/compose-runtime#1.11.0-beta02)                       |
| UI                 | `org.jetbrains.compose.ui:ui*:1.11.0-beta02`                                   | [UI 1.11.0-beta02](https://developer.android.com/jetpack/androidx/releases/compose-ui#1.11.0-beta02)                                 |
| Foundation         | `org.jetbrains.compose.foundation:foundation*:1.11.0-beta02`                   | [Foundation 1.11.0-beta02](https://developer.android.com/jetpack/androidx/releases/compose-foundation#1.11.0-beta02)                 |
| Material           | `org.jetbrains.compose.material:material*:1.11.0-beta02`                       | [Material 1.11.0-beta02](https://developer.android.com/jetpack/androidx/releases/compose-material#1.11.0-beta02)                     |
| Material3          | `org.jetbrains.compose.material3:material3*:1.11.0-alpha06`                    | [Material3 1.5.0-alpha16](https://developer.android.com/jetpack/androidx/releases/compose-material3#1.5.0-alpha16)                   |
| Material3 Adaptive | `org.jetbrains.compose.material3.adaptive:adaptive*:1.3.0-alpha06`             | [Material3 Adaptive 1.3.0-alpha09](https://developer.android.com/jetpack/androidx/releases/compose-material3-adaptive#1.3.0-alpha09) |
| Lifecycle          | `org.jetbrains.androidx.lifecycle:lifecycle-*:2.11.0-alpha03`                  | [Lifecycle 2.11.0-alpha03](https://developer.android.com/jetpack/androidx/releases/lifecycle#2.11.0-alpha03)                         |
| Navigation         | `org.jetbrains.androidx.navigation:navigation-*:2.9.2`                         | [Navigation 2.9.7](https://developer.android.com/jetpack/androidx/releases/navigation#2.9.7)                                         |
| Navigation3        | `org.jetbrains.androidx.navigation3:navigation3-*:1.1.0-rc01`                  | [Navigation3 1.1.0-rc01](https://developer.android.com/jetpack/androidx/releases/navigation3#1.1.0-rc01)                             |
| Navigation Event   | `org.jetbrains.androidx.navigationevent:navigationevent-compose:1.1.0-alpha01` | [Navigation Event 1.1.0-alpha01](https://developer.android.com/jetpack/androidx/releases/navigationevent#1.1.0-alpha01)              |
| Savedstate         | `org.jetbrains.androidx.savedstate:savedstate*:1.4.0`                          | [Savedstate 1.4.0](https://developer.android.com/jetpack/androidx/releases/savedstate#1.4.0)                                         |
| WindowManager Core | `org.jetbrains.androidx.window:window-core:1.5.1`                              | [WindowManager 1.5.1](https://developer.android.com/jetpack/androidx/releases/window#1.5.1)                                          |