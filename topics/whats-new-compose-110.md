[//]: # (title: What's new in Compose Multiplatform 1.10.0-beta01)

Here are the highlights for this EAP feature release:
 * [Unified `@Preview` annotation](#unified-preview-annotation)
 * [Support for Navigation3](#support-for-navigation-3)
 * [Bundled hot reload](#compose-hot-reload-integration)

You can find the full list of changes for this release on [GitHub](https://github.com/JetBrains/compose-multiplatform/releases/tag/v1.10.0-beta01).

## Dependencies

* Gradle Plugin `org.jetbrains.compose`, version `1.10.0-beta01`. Based on Jetpack Compose libraries:
    * [Runtime 1.10.0-beta01](https://developer.android.com/jetpack/androidx/releases/compose-runtime#1.10.0-beta01)
    * [UI 1.10.0-beta01](https://developer.android.com/jetpack/androidx/releases/compose-ui#1.10.0-beta01)
    * [Foundation 1.10.0-beta01](https://developer.android.com/jetpack/androidx/releases/compose-foundation#1.10.0-beta01)
    * [Material 1.10.0-beta01](https://developer.android.com/jetpack/androidx/releases/compose-material#1.10.0-beta01)
    * [Material3 1.4.0](https://developer.android.com/jetpack/androidx/releases/compose-material3#1.4.0)

* Compose Material3 libraries `org.jetbrains.compose.material3:material3*:1.10.0-alpha04`. Based on [Jetpack Compose Material3 1.5.0-alpha07](https://developer.android.com/jetpack/androidx/releases/compose-material3#1.5.0-alpha07)
* Compose Material3 Adaptive libraries `org.jetbrains.compose.material3.adaptive:adaptive*:1.3.0-alpha01`. Based on [Jetpack Compose Material3 Adaptive 1.3.0-alpha02](https://developer.android.com/jetpack/androidx/releases/compose-material3-adaptive#1.3.0-alpha02)
* Lifecycle libraries `org.jetbrains.androidx.lifecycle:lifecycle-*:2.10.0-alpha04`. Based on [Jetpack Lifecycle 2.10.0-beta01](https://developer.android.com/jetpack/androidx/releases/lifecycle#2.10.0-beta01)
* Navigation libraries `org.jetbrains.androidx.navigation:navigation-*:2.9.1`. Based on [Jetpack Navigation 2.9.4](https://developer.android.com/jetpack/androidx/releases/navigation#2.9.4)
* Navigation 3 libraries `org.jetbrains.androidx.navigation:navigation3-*:1.0.0-alpha04`. Based on [Jetpack Navigation 3](https://developer.android.com/jetpack/androidx/releases/navigation3#1.0.0-beta01)
* Navigation Event library `org.jetbrains.androidx.navigationevent:navigationevent-compose:1.0.0-beta01`. Based on [Jetpack Navigation Event 1.0.0-beta01](https://developer.android.com/jetpack/androidx/releases/navigationevent#1.0.0-beta01)
* Savedstate library `org.jetbrains.androidx.savedstate:savedstate*:1.4.0-beta01`. Based on [Jetpack Savedstate 1.4.0-rc01](https://developer.android.com/jetpack/androidx/releases/savedstate#1.4.0-rc01)
* WindowManager Core library `org.jetbrains.androidx.window:window-core:1.5.0-rc01`. Based on [Jetpack WindowManager 1.5.0](https://developer.android.com/jetpack/androidx/releases/window#1.5.0)

## Across platforms

### Unified `@Preview` annotation

We've unified the approach to previews across platforms. 
You can now use the `androidx.compose.ui.tooling.preview.Preview` annotation for all target platforms in the `commonMain` source set.

All other annotations, such as `org.jetbrains.compose.ui.tooling.preview.Preview` and 
the desktop-specific `androidx.compose.desktop.ui.tooling.preview.Preview`, have been deprecated.

### Support for Navigation 3

Navigation 3 is a new navigation library designed to work with Compose. 
With Navigation 3, you have full control over your back stack, 
and navigating to and from destinations is as simple as adding and removing items from a list. 
You can read about the new guiding principles and decisions in the [Navigation 3 documentation](https://developer.android.com/guide/navigation/navigation-3), 
as well as in the announcement [blog post](https://android-developers.googleblog.com/2025/05/announcing-jetpack-navigation-3-for-compose.html).

Compose Multiplatform 1.10.0-beta01 provides Alpha support for using new navigation APIs on non-Android targets. 
The released multiplatform artifacts are:

* Navigation 3 UI library, `org.jetbrains.androidx.navigation3:navigation3-ui`
* ViewModel for Navigation 3, `org.jetbrains.androidx.lifecycle:lifecycle-viewmodel-navigation3`
* Material 3 adaptive layouts for Navigation 3, `org.jetbrains.compose.material3.adaptive:adaptive-navigation3`

You can find an example of a multiplatform Navigation 3 implementation in the [nav3-recipes](https://github.com/terrakok/nav3-recipes) sample mirrored from the original Android repository.

Some platform-specific implementation details:

* On iOS, you can now manage navigation for end edge [pan gestures](https://developer.apple.com/documentation/uikit/handling-pan-gestures) using the [EndEdgePanGestureBehavior](https://github.com/JetBrains/compose-multiplatform-core/pull/2519) option (`Disabled` by default). 
  "End edge" here refers to the right edge of the screen in LTR interfaces and the left edge in RTL ones. 
  The start edge is the opposite to the end edge and is always bound to the back gesture.
* In web apps, pressing the **Esc** key in a desktop browser now returns the user to the previous screen 
  (and closes dialogs, popups and some widgets like Material 3's `SearchBar`), 
  just as it already does in desktop apps.
* Support for [browser history navigation](compose-navigation-routing.md#support-for-browser-navigation-in-web-apps) and using a destination in the address bar will not be extended 
  to Navigation 3 with Compose Multiplatform 1.10. 
  This has been postponed until a later version of the multiplatform library.

### Skia updated to Milestone 138

The version of Skia used by Compose Multiplatform, via Skiko, has been updated to Milestone 138.

The previous version of Skia used was Milestone 132. 
You can see the changes made between these versions in the [release notes](https://skia.googlesource.com/skia/+/refs/heads/chrome/m138/RELEASE_NOTES.md).

## iOS

### Window insets

Compose Multiplatform now supports `WindowInsetsRulers`, 
which provides functionality to position and size UI elements based on window insets, such as the status bar, 
navigation bar, or on-screen keyboard.

This new approach to managing window insets uses a single implementation for retrieving platform-specific window inset data. 
This means both `WindowInsets` and `WindowInsetsRulers` use a common mechanism to manage insets consistently.
Accordingly, platform-specific locals, including `LocalLayoutMargins`, `LocalSafeArea`, 
`LocalKeyboardOverlapHeight`, and `LocalInterfaceOrientation`, were removed in favor of a new unified API.

>Previously, `WindowInsets.Companion.captionBar` was not marked as `@Composable`. 
> We added the `@Composable` attribute to align its behavior across platforms.
> 
{style="note"}

### Improved IME configuration

Following the iOS-specific IME customization [introduced in 1.9.0](whats-new-compose-190.md#ime-options), 
this release adds new APIs for configuring text input views with `PlatformImeOptions`.

These new APIs allow customization of the input interface when a field gains focus and triggers the IME:

 * `UIResponder.inputView` specifies a custom input view to replace the default system keyboard.
 * `UIResponder.inputAccessoryView` defines a custom accessory view that attaches to the system keyboard 
    or a custom `inputView` upon IME activation.

## Desktop

### Compose Hot Reload integration

The Compose Hot Reload plugin is now bundled with the Compose Multiplatform Gradle plugin. 
You no longer need to configure the Hot Reload plugin separately, 
as it is enabled by default for Compose Multiplatform projects targeting desktop.

What this means for the projects that explicitly declare the Compose Hot Reload plugin:

 * You can safely remove the declaration in order to use the version provided by the Compose Multiplatform Gradle plugin.
 * If you choose to keep a specific version declaration, that version will be used instead of the bundled one.

> The bundled Compose Hot Reload Gradle plugin raises the Kotlin version necessary for 
> a Compose Multiplatform project to 2.1.20.
>
{style="warning"}

### Autosizing `SwingPanel`

`SwingPanel` now automatically adjusts its size based on the content's minimum, preferred, and maximum sizes. 
This removes the need to calculate exact sizes and specify fixed dimensions in advance.

```kotlin
val label = JLabel("Hello Swing!")

singleWindowApplication {
    SwingPanel(factory = { label })

    LaunchedEffect(Unit) {
        delay(500)
        // Grows the text
        repeat(2) { 
            label.text = "#${label.text}#"
            delay(200)
        }
        // Shrinks the text
        repeat(2) { 
            label.text = label.text.substring(1, label.text.length - 1)
            delay(200)
        }
    }
}
```

## Gradle

### Deprecated dependency aliases

Dependency aliases supported by the Compose Multiplatform Gradle plugin (`compose.ui` and others)
are deprecated with the 1.10.0-beta01 release.
We encourage you to add direct library references to your version catalogs.
Specific references are suggested in the corresponding deprecation notices.

This change should make dependency management for Compose Multiplatform libraries a bit more transparent.
In the future, we hope to provide a BOM for Compose Multiplatform to simplify setting up compatible versions.

### Support for AGP 9.0.0

Compose Multiplatform introduces support for version 9.0.0 of the Android Gradle Plugin (AGP).
For compatibility with the new AGP version, make sure you upgrade to Compose Multiplatform 1.10.0.

To make the update process smoother in the long term, 
we recommend changing your project structure to isolate AGP usage to a dedicated Android module.
