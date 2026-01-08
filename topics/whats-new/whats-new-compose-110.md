[//]: # (title: What's new in Compose Multiplatform 1.10.0-rc02)

Here are the highlights for this EAP feature release:
 * [Unified `@Preview` annotation](#unified-preview-annotation)
 * [Support for Navigation 3](#support-for-navigation-3)
 * [Bundled Compose Hot Reload](#compose-hot-reload-integration)

You can find the full list of changes for this release on [GitHub](https://github.com/JetBrains/compose-multiplatform/releases/tag/v1.10.0-beta01).

## Dependencies

* Gradle Plugin `org.jetbrains.compose`, version `1.10.0-rc02`. Based on Jetpack Compose libraries:
    * [Runtime 1.10.0](https://developer.android.com/jetpack/androidx/releases/compose-runtime#1.10.0)
    * [UI 1.10.0](https://developer.android.com/jetpack/androidx/releases/compose-ui#1.10.0)
    * [Foundation 1.10.0](https://developer.android.com/jetpack/androidx/releases/compose-foundation#1.10.0)
    * [Material 1.10.0](https://developer.android.com/jetpack/androidx/releases/compose-material#1.10.0)

* Compose Material3 library `org.jetbrains.compose.material3:material3*:1.10.0-alpha05`. Based on [Jetpack Compose Material3 1.5.0-alpha08](https://developer.android.com/jetpack/androidx/releases/compose-material3#1.5.0-alpha08).

  To use the [Expressive theme](whats-new-compose-190.md#material-3-expressive-theme), include the experimental version of Material 3:
    ```kotlin
    implementation("org.jetbrains.compose.material3:material3:1.9.0-alpha05")
    ```
* Compose Material3 Adaptive libraries `org.jetbrains.compose.material3.adaptive:adaptive*:1.3.0-alpha02`. Based on [Jetpack Compose Material3 Adaptive 1.3.0-alpha03](https://developer.android.com/jetpack/androidx/releases/compose-material3-adaptive#1.3.0-alpha03)
* Lifecycle libraries `org.jetbrains.androidx.lifecycle:lifecycle-*:2.10.0-alpha06`. Based on [Jetpack Lifecycle 2.10.0](https://developer.android.com/jetpack/androidx/releases/lifecycle#2.10.0)
* Navigation libraries `org.jetbrains.androidx.navigation:navigation-*:2.9.1`. Based on [Jetpack Navigation 2.9.4](https://developer.android.com/jetpack/androidx/releases/navigation#2.9.4)
* Navigation 3 libraries `org.jetbrains.androidx.navigation3:navigation3-*:1.0.0-alpha06`. Based on [Jetpack Navigation3 1.0.0](https://developer.android.com/jetpack/androidx/releases/navigation3#1.0.0)
* Navigation Event library `org.jetbrains.androidx.navigationevent:navigationevent-compose:1.0.0-rc02`. Based on [Jetpack Navigation Event 1.0.1](https://developer.android.com/jetpack/androidx/releases/navigationevent#1.0.1)
* Savedstate library `org.jetbrains.androidx.savedstate:savedstate*:1.4.0`. Based on [Jetpack Savedstate 1.4.0](https://developer.android.com/jetpack/androidx/releases/savedstate#1.4.0)
* WindowManager Core library `org.jetbrains.androidx.window:window-core:1.5.1`. Based on [Jetpack WindowManager 1.5.1](https://developer.android.com/jetpack/androidx/releases/window#1.5.1)

## Breaking changes and deprecations

### Deprecated dependency aliases

Dependency aliases supported by the Compose Multiplatform Gradle plugin (`compose.ui` and others)
are deprecated with the 1.10.0-beta01 release.
We encourage you to add direct library references to your version catalogs.
Specific references are suggested in the corresponding deprecation notices.

This change should make dependency management for Compose Multiplatform libraries a bit more transparent.
In the future, we hope to provide a BOM for Compose Multiplatform to simplify setting up compatible versions.

### Deprecated `PredictiveBackHandler()`

The `PredictiveBackHandler()` function was introduced in Compose Multiplatform to bring native Android back navigation
gesture to other platforms.
With the release of Navigation 3, the old implementation has been deprecated in favor of the new [Navigation Event](https://developer.android.com/jetpack/androidx/releases/navigationevent)
library and its APIs.
Specifically, instead of using the `PredictiveBackHandler()` function, you should now use the new `NavigationBackHandler()` function that wraps
the more general `NavigationEventHandler()` implementation.

The simplest migration can look like this:

<compare type="top-bottom">
    <code-block lang="kotlin">
         PredictiveBackHandler(enabled = true) { progress ->
            try {
                progress.collect { event ->
                    // Animate the back gesture progress
                }
                // Process the completed back gesture
            } catch(e: Exception) {
                // Process the canceled back gesture
            }
        }
    </code-block>
    <code-block lang="kotlin">
        // Use an empty state as a stub to satisfy the required argument
        val navState = rememberNavigationEventState(NavigationEventInfo.None)
        NavigationBackHandler(
            state = navState,
            isBackEnabled = true,
            onBackCancelled = {
                // Process the canceled back gesture
            },
            onBackCompleted = {
              // Process the completed back gesture
            }
        )
        LaunchedEffect(navState.transitionState) {
            val transitionState = navState.transitionState
            if (transitionState is NavigationEventTransitionState.InProgress) {
                val progress = transitionState.latestEvent.progress
                // Animate the back gesture progress
            }
        }
    </code-block>
</compare>

Here:

* The `state` parameter is mandatory: `NavigationEventInfo` is designed to hold contextual information about the UI state.
  If you don't have any information to store for now, you can use `NavigationEventInfo.None` as a stub.
* The `onBack` parameter has been split into `onBackCancelled` and `onBackCompleted`, so you don't need to track canceled
  gestures separately.
* The `NavigationEventState.transitionState` property helps to track the progress of the physical gesture.

For details on the implementation, see the [NavigationEventHandler page in the Navigation Event API reference](https://developer.android.com/reference/kotlin/androidx/navigationevent/NavigationEventHandler).

### Minimum Kotlin version increased for web

If your project includes a web target, the latest features require upgrading to Kotlin 2.2.20.

## Across platforms

### Unified `@Preview` annotation

We've unified the approach to previews across platforms. 
You can now use the `androidx.compose.ui.tooling.preview.Preview` annotation for all target platforms in the `commonMain` source set.

All other annotations, such as `org.jetbrains.compose.ui.tooling.preview.Preview` and 
the desktop-specific `androidx.compose.desktop.ui.tooling.preview.Preview`, have been deprecated.

### Autosizing interop views

Compose Multiplatform now supports automatic resizing for native interop elements on both desktop and iOS.
These elements can now adapt their layout to their content,
eliminating the need to calculate exact sizes manually and specify fixed dimensions in advance.

* On desktop, `SwingPanel` automatically adjusts its size based on the embedded component's minimum, preferred, and maximum sizes.
* On iOS, UIKit interop views now support sizing according to the view's fitting size (intrinsic content size).
  This enables proper wrapping of SwiftUI views (via `UIHostingController`)
  and basic `UIView` subclasses that do not depend on `NSLayoutConstraints`.

### Stable `Popup` and `Dialog` properties

The following properties in `DialogProperties` have been promoted to stable and are no longer experimental: 
`usePlatformInsets`, `useSoftwareKeyboardInset`, and `scrimColor`.

Similarly, the `usePlatformDefaultWidth` and `usePlatformInsets` properties 
in `PopupProperties` have also been promoted to stable.

The deprecation level for `Popup` overloads without the `PopupProperties` parameter 
has been changed to `ERROR` to enforce the use of the updated API.

### Skia updated to Milestone 138

The version of Skia used by Compose Multiplatform, via Skiko, has been updated to Milestone 138.

The previous version of Skia used was Milestone 132.
You can see the changes made between these versions in the [release notes](https://skia.googlesource.com/skia/+/refs/heads/chrome/m138/RELEASE_NOTES.md).

### Support for Navigation 3
<primary-label ref="Experimental"/>

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

## iOS

### Window insets

Compose Multiplatform now supports `WindowInsetsRulers`, 
which provides functionality to position and size UI elements based on window insets, such as the status bar, 
navigation bar, or on-screen keyboard.

This new approach to managing window insets uses a single implementation for retrieving platform-specific window inset data. 
This means both `WindowInsets` and `WindowInsetsRulers` use a common mechanism to manage insets consistently.

> Previously, `WindowInsets.Companion.captionBar` was not marked as `@Composable`. 
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

### Overlay placement for interop views
<primary-label ref="Experimental"/>

You can now place `UIKitView` and `UIKitViewController` views above the Compose UI using the experimental `placedAsOverlay` flag.
This flag allows interop views to support transparent backgrounds and native shader effects.

To render an interop view as an overlay, use the `@OptIn(ExperimentalComposeUiApi::class)` annotation and 
set the `placedAsOverlay` parameter to `true` in `UIKitInteropProperties`:

```kotlin
UIKitViewController(
    modifier = modifier,
    update = {},
    factory = { factory.createNativeMap() },
    properties = UIKitInteropProperties(placedAsOverlay = true)
)
```

Keep in mind that this configuration renders the view on top of the Compose UI layer; 
consequently, it will visually cover any other composables located in the same area.

## Web

### Resource caching
<primary-label ref="Experimental"/>

Compose Multiplatform now uses the [Web Cache API](https://developer.mozilla.org/en-US/docs/Web/API/Cache) 
to cache successful responses for static assets and string resources.
This approach avoids the delays associated with the browser's default cache, 
which validates stored content through repeated HTTP requests and can be particularly slow on low-bandwidth connections. 
The cache is cleared on every app launch or page refresh to ensure resources remain consistent with the application's current state.

For more details, see the [pull request](https://github.com/JetBrains/compose-multiplatform/pull/5379)
and [Caching web resources](compose-multiplatform-resources-usage.md#caching-web-resources).

## Desktop

### Compose Hot Reload integration

The Compose Hot Reload plugin is now bundled with the Compose Multiplatform Gradle plugin. 
You no longer need to configure the Hot Reload plugin separately, 
as it is enabled by default for Compose Multiplatform projects targeting desktop.

What this means for the projects that explicitly declare the Compose Hot Reload plugin:

 * You can safely remove the declaration in order to use the version provided by the Compose Multiplatform Gradle plugin.
 * If you choose to keep a specific version declaration, that version will be used instead of the bundled one.

The minimum Kotlin version for the bundled Compose Hot Reload Gradle plugin is 2.1.20.
If an older version of Kotlin is detected, the hot reload functionality will be disabled.

## Gradle

### Support for AGP 9.0.0

Compose Multiplatform introduces support for version 9.0.0 of the Android Gradle Plugin (AGP).
For compatibility with the new AGP version, make sure you upgrade to Compose Multiplatform 1.9.3 or 1.10.0.

To make the update process smoother in the long term,
we recommend changing your project structure to use a dedicated Android application module.
