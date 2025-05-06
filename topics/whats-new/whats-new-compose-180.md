[//]: # (title: What's new in Compose Multiplatform %composeVersion%)
Here are the highlights for this feature release:

* [Variable fonts](#variable-fonts)
* [Drag-and-drop on iOS](#drag-and-drop)
* [Deep linking on iOS](#deep-linking)
* [Improved accessibility on iOS](#accessibility-support-improvements)
* [Preloading of resources for web targets](#preloading-of-resources)
* [Integration with browser navigation controls](#browser-controls-supported-in-the-navigation-library)

See the full list of changes for this release [on GitHub](https://github.com/JetBrains/compose-multiplatform/releases/tag/v1.8.0).

## Dependencies

* Gradle Plugin `org.jetbrains.compose`, version 1.8.0. Based on Jetpack Compose libraries:
    * [Runtime 1.8.0](https://developer.android.com/jetpack/androidx/releases/compose-runtime#1.8.0)
    * [UI 1.8.0](https://developer.android.com/jetpack/androidx/releases/compose-ui#1.8.0)
    * [Foundation 1.8.0](https://developer.android.com/jetpack/androidx/releases/compose-foundation#1.8.0)
    * [Material 1.8.0](https://developer.android.com/jetpack/androidx/releases/compose-material#1.8.0)
    * [Material3 1.3.2](https://developer.android.com/jetpack/androidx/releases/compose-material3#1.3.2)
* Lifecycle libraries `org.jetbrains.androidx.lifecycle:lifecycle-*:2.9.0-beta01`. Based on [Jetpack Lifecycle 2.9.0-beta01](https://developer.android.com/jetpack/androidx/releases/lifecycle#2.9.0-beta01)
* Navigation libraries `org.jetbrains.androidx.navigation:navigation-*:2.9.0-beta01`. Based on [Jetpack Navigation 2.9.0-beta01](https://developer.android.com/jetpack/androidx/releases/navigation#2.9.0-beta01)
* Material3 Adaptive libraries `org.jetbrains.compose.material3.adaptive:adaptive*:1.1.0`. Based on [Jetpack Material3 Adaptive 1.1.0](https://developer.android.com/jetpack/androidx/releases/compose-material3-adaptive#1.1.0)

## Breaking changes

### Full migration of Compose Multiplatform to the K2 compiler

With this release, the Compose Multiplatform codebase has been fully migrated to the K2 compiler.
Starting with 1.8.0,
native and web klibs produced by projects that depend on Compose Multiplatform can be consumed
only when using Kotlin 2.1.0 or newer.

What this means for your projects, in addition to the underlying changes in the Compose compiler Gradle plugin:

* For an app using libraries that depend on Compose Multiplatform:
  it's recommended to update your project to Kotlin 2.1.20
  and update the dependencies to versions compiled against Compose Multiplatform 1.8.0 and Kotlin 2.1.x.
* For a library that depends on Compose Multiplatform:
    you need to update your project to Kotlin 2.1.x and Compose 1.8.0,
    then recompile the library and publish the new version.

If you encounter any compatibility issues when upgrading to Compose Multiplatform 1.8.0,
please let us know by filing an issue in [YouTrack](https://youtrack.jetbrains.com/newIssue?project=CMP).

### Implicit dependency on `material-icons-core` removed

Compose Multiplatform %composeVersion% incorporates a [change made in Material](https://android.googlesource.com/platform/frameworks/support/+/1d1abef790da93325a83fe19b50ccdec06be6956):
there is no longer a transitive dependency on `material-icons-core`.
This aligns with [moving away from dependencies built with K1](#full-migration-of-compose-multiplatform-to-the-k2-compiler).

If you need to continue using the `material-icons-core` library in your project,
add the dependency explicitly to your `build.gradle.kts`, for example:

```kotlin
implementation("org.jetbrains.compose.material:material-icons-core:1.6.11")
```

### Migration from Bundle to SavedState in Navigation

Navigation in Compose Multiplatform %composeVersion%,
along with the Android Navigation component, is transitioning to using the `SavedState` class to store UI states.
This breaks the pattern of accessing state data when you declare destinations in a navigation graph.
When upgrading to a 2.9.* version of the [Navigation library](compose-navigation-routing.md),
make sure to update such code to use accessors of `SavedState`.

> For a more robust architecture,
> use the [type-safe approach to navigation](https://developer.android.com/guide/navigation/design/type-safety),
> avoiding string routes.
>
{style="note"}

Before:

```kotlin
composable(Destinations.Followers.route) { navBackStackEntry ->
    val uId = navBackStackEntry.arguments?.getString("userid")
    val page = navBackStackEntry.arguments?.getString("page")
    if (uId != null && page != null) {
        FollowersMainComposable(navController, accountId = uId, page = page)
    }
}
```

Starting with Compose Multiplatform %composeVersion%:

```kotlin
composable(Destinations.Followers.route) { navBackStackEntry ->
    val uId = navBackStackEntry.arguments?.read { getStringOrNull("userid") }
    val page = navBackStackEntry.arguments?.read { getStringOrNull("page") }
    if (uId != null && page != null) {
        FollowersMainComposable(navController, accountId = uId, page = page)
    }
}
```

### Deprecated `ComposeUIViewControllerDelegate` on iOS

The `ComposeUIViewControllerDelegate` API has been deprecated in favor of the parent view controller. 
If you use the deprecated API with Compose Multiplatform %composeVersion%, you will encounter a deprecation error indicating 
that you should override the `UIViewController` class methods via the parent view controller.

Read more about child-parent view controller relationships in Apple’s developer [documentation](https://developer.apple.com/documentation/uikit/uiviewcontroller).

### Removed obsolete `platformLayers` option on iOS

The `platformLayers`
experimental option [was introduced in 1.6.0](https://www.jetbrains.com/help/kotlin-multiplatform-dev/whats-new-compose.html#separate-platform-views-for-popups-dialogs-and-dropdowns-ios-desktop)
to allow enabling alternative layering mode and drawing popups and dialogs outside the bounds of the parent container.

This mode is now the default behavior on iOS, and the option to enable it has been removed as obsolete.

### Breaking changes in tests

#### New handling of coroutine delays in tests

Previously, Compose Multiplatform tests would not consider side effects with `delay()` calls idle.
Because of that, the following test, for example, would hang indefinitely:

```kotlin
@Test
fun loopInLaunchedEffectTest() = runComposeUiTest {
    setContent {
        LaunchedEffect(Unit) {
            while (true) {
                delay(1000)
                println("Tick")
            }
        }
    }
}
```

When coroutines call the `delay()` function after being launched in a composition scope, the `waitForIdle()`, `awaitIdle()`,
and `runOnIdle()` functions now consider Compose to be idle.
This change fixes the hanging test above but breaks tests that rely on `waitForIdle()`, `awaitIdle()`, and `runOnIdle()`
to execute coroutines with `delay()`.

To produce the same results in these cases, advance time artificially:

```kotlin
var updateText by mutableStateOf(false)
var text by mutableStateOf("0")
setContent {
    LaunchedEffect(updateText) {
        if (updateText) {
            delay(1000)
            text = "1"
        }
    }
}
updateText = true
waitForIdle()
// Since waitForIdle() no longer waits for the delayed LaunchedEffect() to complete,
// the test needs to advance time to make the following assertion correct:
mainClock.advanceTimeBy(1001)

assertEquals("1", text)
```

Tests that already use `mainClock.advanceTimeBy()` calls to advance the test clock may behave differently
with recomposition, layout, drawing, and effects.

#### Implementation of `runOnIdle()` aligned with Android

To bring the Compose Multiplatform implementation of the `runOnIdle()` test function in line with Android behavior,
we’ve introduced the following changes:

* `runOnIdle()` now executes its `action` on the UI thread.
* `runOnIdle()` does not call `waitForIdle()` after executing `action` anymore.

If your tests rely on that extra `waitForIdle()` call after the `runOnIdle()` action,
add the call to your tests as needed when you update them for Compose Multiplatform %composeVersion%.

#### Advancing time in tests is decoupled from rendering

In Compose Multiplatform %composeVersion%, the `mainClock.advanceTimeBy()` function no longer causes recomposition, layout,
or drawing if the time was not advanced past the point of rendering the next frame (virtual test frames are rendered every 16 ms).

This may break tests that rely on rendering being triggered by every `mainClock.advanceTimeBy()` call.
See the [PR description](https://github.com/JetBrains/compose-multiplatform-core/pull/1618) for details.

## Across platforms

### Variable fonts

Compose Multiplatform %composeVersion% supports variable fonts on all platforms.
With variable fonts, you can keep one font file that contains all style preferences such as weight,
width, slant, italic, custom axes, visual weight with typographic color,
and adaptations to specific text sizes.

For details,
see the [Jetpack Compose documentation](https://developer.android.com/develop/ui/compose/text/fonts#variable-fonts).

### Skia updated to Milestone 132

The version of Skia used by Compose Multiplatform, via Skiko, has been updated to Milestone 132.

The previous version of Skia used was Milestone 126. You can see the changes
made between these versions in the [release notes](https://skia.googlesource.com/skia/+/main/RELEASE_NOTES.md#milestone-132).

### New Clipboard interface

Compose Multiplatform has adopted Jetpack Compose’s new `Clipboard` interface.

The previously used `ClipboardManager` interface, which was inaccessible on web targets due to the asynchronous nature 
of the [Clipboard API on web](https://developer.mozilla.org/en-US/docs/Web/API/Clipboard_API),
has been deprecated in favor of `Clipboard`. The new interface supports `suspend` functions and is compatible with all targets,
including web.

Clipboard interactions from common code are currently limited by the API design.
For more details, see [CMP-7624](https://youtrack.jetbrains.com/issue/CMP-7624).

### Line-height alignment

The common API for line-height alignment,
previously supported by Compose Multiplatform on Android only, is now supported on all platforms.
Using `LineHeightStyle.Alignment`, you can configure how a text line aligns within the space provided by the line height.
Text lines can be aligned to the bottom, center, or top of the reserved space,
or adjusted proportionally based on their ascent and descent values.

<img src="compose-180-LineHeightStyle.png" alt="Line-height alignment" width="508"/>

Note that in Material3, the default value for line-height alignment is `Center`,
meaning central alignment will be applied to text with `lineHeight`
in Material3 components across all platforms unless otherwise specified.

## iOS

### Deep linking

By using Compose Multiplatform %composeVersion% along with [org.jetbrains.androidx.navigation.navigation-compose](compose-navigation-routing.md)
%composeNavigationVersion%,
you can implement deep linking on iOS in the usual Compose manner:
assigning deep links to destinations and navigating to them using a `NavController`.

For a guide on introducing deep links to common code, see [](compose-navigation-deep-links.md).

### Accessibility support improvements

#### Support for right-to-left languages

Compose Multiplatform %composeVersion% introduces accessibility support for right-to-left languages,
including proper text direction handling for gestures.

To learn more about RTL support, refer to [Right-to-Left languages](compose-rtl.md).

#### Accessibility for scrollable lists

This version improves the performance and accuracy of scrolling boundary and element position calculations.
By accounting for safe areas, such as notches and screen edges,
we ensure precise accessibility properties for scrolling near gaps and margins.

We’ve also introduced support for scrolling state announcements.
With VoiceOver enabled, you will hear list status updates when performing three-finger scrolling gestures.
The announcements include:

* "First page" when at the top of the list.
* "Next page" when scrolling forward.
* "Previous page" when scrolling backward.
* "Last page" upon reaching the end.

Localized versions of these announcements are also provided, allowing VoiceOver to read them in your language of choice.

#### Accessibility for container views

By default, screen readers navigate through UI elements in a fixed order,
following their layout from left to right and top to bottom.
For complex layouts,
screen readers cannot determine the correct reading order without properly defined traversal semantics.
This is particularly relevant for layouts with container views, such as tables and nested views,
that support the scrolling and zooming of contained views.

Starting with Compose Multiplatform %composeVersion%,
you can define traversal semantic properties for containers
to ensure the correct reading order when scrolling and swiping through complex views.

For example, if you want the screen reader to first focus on a floating action button,
you can set its traversal index to `-1f`.
The default value of the index is `0f`,
which means that the lower specified value causes the element to be read before others on the screen.

```kotlin
@Composable
fun FloatingBox() {
    Box(
        modifier =
        Modifier.semantics {
            isTraversalGroup = true
            // Sets a negative index to prioritize over elements with the default index
            traversalIndex = -1f
        }
    ) {
        FloatingActionButton(onClick = {}) {
            Icon(
                imageVector = Icons.Default.Add,
                contentDescription = "Icon of floating action button"
            )
        }
    }
}
```

In addition to proper sorting elements for screen readers,
support for traversal properties enables navigation between different traversal groups with the swipe-up or swipe-down accessibility gestures.
To switch to accessible navigation mode for containers, rotate two fingers on the screen while VoiceOver is active.

#### Accessible text input

In Compose Multiplatform %composeVersion% we've introduced support for text fields' accessibility traits.
When a text input field comes into focus, it is now marked as editable,
ensuring proper accessibility-state representation.

You can now also use accessible text input in UI testing.

#### Support for control via trackpad and keyboard
Compose Multiplatform for iOS now supports two additional input methods to control your device. Instead of relying on the touchscreen, 
you can enable either AssistiveTouch to use a mouse or trackpad, or Full Keyboard Access to use a keyboard:

* AssistiveTouch (**Settings** | **Accessibility** | **Touch** | **AssistiveTouch**) allows you to control your iPhone or 
 iPad with a pointer from a connected mouse or trackpad. You can use the pointer to click icons on your screen, 
 navigate through the AssistiveTouch menu, or type using the onscreen keyboard.
* Full Keyboard Access (**Settings** | **Accessibility** | **Keyboards** | **Full Keyboard Access**) enables device control 
 with a connected keyboard. You can navigate with keys like **Tab** and activate items using **Space**.

#### Loading accessibility tree on demand

Instead of setting a specific mode of syncing the Compose semantic tree with the iOS accessibility tree,
you can now rely on Compose Multiplatform to handle this process lazily.
The tree is fully loaded after the first request from the iOS accessibility engine
and is disposed of when the screen reader stops interacting with it.

This allows fully supporting iOS Voice Control, VoiceOver,
and other accessibility tools that rely on the accessibility tree.

The `AccessibilitySyncOptions` class, which was [used to configure accessibility tree sync](compose-ios-accessibility.md#choose-the-tree-synchronization-option),
has been removed as no longer necessary.

#### Improved accuracy of accessibility property calculations

We’ve updated the accessibility properties
of Compose Multiplatform components to match the expected behavior of UIKit components.
The UI elements now provide extensive accessibility data,
and any transparent component with an alpha value of 0 no longer provides accessibility semantics.

Aligning semantics also allowed us
to fix several issues related to the incorrect calculation of accessibility properties,
such as missing hitboxes for `DropDown` elements,
mismatches between visible text and accessibility labels, and incorrect radio button states.

### Stable API for iOS logging

The API that enables operating system logging on iOS is now stable. The `enableTraceOSLog()` function no longer requires 
experimental opt-in and now aligns with Android-style logging. This logging provides trace information that can be analyzed 
using Xcode Instruments for debugging and performance analysis.

### Drag-and-drop
<secondary-label ref="Experimental"/>

Compose Multiplatform for iOS introduces support for drag-and-drop functionality,
allowing you to drag content into or out of a Compose application
(see pull request [1690](https://github.com/JetBrains/compose-multiplatform-core/pull/1690) for a demo video).
To define draggable content and drop targets, use the `dragAndDropSource` and `dragAndDropTarget` modifiers.

On iOS, drag-and-drop session data is represented by [`UIDragItem`](https://developer.apple.com/documentation/uikit/uidragitem).
This object contains information about cross-process data transfer and an optional local object for in-app usage.
For example, you can use `DragAndDropTransferData(listOf(UIDragItem.fromString(text)))` to drag text,
where `UIDragItem.fromString(text)` encodes the text into a format suitable for drag-and-drop operations.
Currently, only the `String` and `NSObject` types are supported.

For common use cases,
see the [dedicated article](https://developer.android.com/develop/ui/compose/touch-input/user-interactions/drag-and-drop) in the Jetpack Compose documentation.

### Improved touch handling for scrolling interop views

In this release:

* A Compose view with non-scrollable content that is presented as a modal `UIViewController` can be closed with a swipe-down gesture.
* Nested scrollable views work correctly within the general [interop touch framework](compose-ios-touch.md):
  when scrolling native content within a scrollable Compose view, or Compose content within a scrollable native view,
  the UI closely follows iOS logic to resolve ambiguous touch sequences.

### Opt-in concurrent rendering
<secondary-label ref="Experimental"/>

Compose Multiplatform for iOS now supports offloading rendering tasks to a dedicated render thread.
Concurrent rendering may improve performance in scenarios without UIKit interop.

Opt in to encode the rendering commands on a separate render thread by enabling the `useSeparateRenderThreadWhenPossible`
flag of the `ComposeUIViewControllerConfiguration` class, or the `parallelRendering`
property directly within the `ComposeUIViewController` configuration block:

```kotlin
@OptIn(ExperimentalComposeUiApi::class)
fun main(vararg args: String) {
    UIKitMain {
        ComposeUIViewController(configure = { parallelRendering = true }) {
            // ...
        }
    }
}
```

## Web

### Browser controls supported in the Navigation library

In Kotlin/Wasm and Kotlin/JS applications built with Compose Multiplatform,
navigation now correctly works with basic browser controls.
To enable this, link the browser window to the main navigation graph with the `window.bindToNavigation()` method.
Once you do, the web app will correctly react to using the **Back** and **Forward** buttons to move through the history in the browser
(see pull request [1621](https://github.com/JetBrains/compose-multiplatform-core/pull/1621) for a demo video).

The web app will also manipulate the browser address bar to reflect the current destination route
and navigate to a destination directly when the user pastes a URL with the correct route encoded
(see pull request [1640](https://github.com/JetBrains/compose-multiplatform-core/pull/1640) for a demo video).
The `window.bindToNavigation()` method has the optional `getBackStackEntryPath` parameter,
which allows you to customize the translation of route strings into URL fragments.

### Setting the browser cursor
<secondary-label ref="Experimental"/>

We introduced an experimental `PointerIcon.Companion.fromKeyword()` function to manage icons that can be used as mouse 
pointers on a browser page. By passing a keyword as a parameter, you can specify the type of cursor to display based on the context. 
For example, you can assign different pointer icons to select text, open a context menu, or indicate a loading process.

Check out the full list of available [keywords](https://developer.mozilla.org/en-US/docs/Web/CSS/cursor).

### Preloading of resources
<secondary-label ref="Experimental"/>

Compose Multiplatform %composeVersion% introduces a new experimental API
for preloading fonts and images for web targets.
Preloading helps
prevent visual issues such as flashes of unstyled text (FOUT) or the flickering of images and icons.

The following functions are now available for loading and caching resources:

* `preloadFont()`, which preloads fonts.
* `preloadImageBitmap()`, which preloads bitmap images.
* `preloadImageVector()`, which preloads vector images.

See the [documentation](compose-multiplatform-resources-usage.md#preload-resources-using-the-compose-multiplatform-preload-api) for details.

## Desktop

### Software rendering improvement on Windows

Switching to the recommended clang compiler for Skia on Windows sped up rendering that relies on the CPU.
This mostly affects pure software rendering, as rendering generally relies on the GPU, with only some computations done on the CPU.
Therefore the improvement is very noticeable on some virtual machines
and with a few old graphics cards that [aren't supported by Skia](https://github.com/JetBrains/skiko/blob/30df516c1a1a25237880f3e0fe83e44a13821292/skiko/src/jvmMain/kotlin/org/jetbrains/skiko/GraphicsApi.jvm.kt#L13):
Windows apps produced by Compose Multiplatform are now up to 6 times as fast in those environments
compared to Compose Multiplatform 1.7.3.

This improvement, in addition to the Windows for ARM64 support, makes Compose Multiplatform UIs on virtual Windows systems
under macOS significantly more performant.

### Support for Windows for ARM64

Compose Multiplatform %composeVersion% introduces support for Windows for ARM64 on the JVM,
improving the overall experience of building and running applications on ARM-based Windows devices.

## Gradle plugin

### Support for multiplatform resources in the `androidLibrary` target
<secondary-label ref="Experimental"/>

Starting with the Android Gradle plugin version 8.8.0, you can use generated assets in the new `androidLibrary` target. 
To align Compose Multiplatform with these changes, we’ve introduced support for a new target configuration to work with 
multiplatform resources packed into Android assets.

If you are using the `androidLibrary` target, enable resources in your configuration:

```
kotlin {
  androidLibrary {
    experimentalProperties["android.experimental.kmp.enableAndroidResources"] = true
  }
}
```

Otherwise, you’ll encounter the following exception: `org.jetbrains.compose.resources.MissingResourceException: Missing resource with path: …`.