[//]: # (title: What's new in Compose Multiplatform %composeEapVersion%)
Here are the highlights for this EAP feature release:

* [Variable fonts](#variable-fonts)
* [Drag-and-drop on iOS](#drag-and-drop)
* [Improved accessibility on iOS](#accessibility-support-improvements)
* [Preloading of resources for web targets](#preloading-of-resources)
* [Integration with browser navigation controls](#browser-controls-supported-in-the-navigation-library)

See the full list of changes for this release [on GitHub](https://github.com/JetBrains/compose-multiplatform/blob/v1.8.0-beta01/CHANGELOG.md).

## Breaking changes

### Full migration of Compose Multiplatform to the K2 compiler

With this release, the Compose Multiplatform codebase has been fully migrated to the K2 compiler.
Starting with 1.8.0,
native and web klibs produced by projects that depend on Compose Multiplatform can be consumed
only when using Kotlin 2.1.0 or newer.

What this means for your projects, in addition to the underlying changes in the Compose compiler Gradle plugin:

* For a library that depends on Compose Multiplatform:
you need to update your project to Kotlin 2.1.x and Compose 1.8.0,
then recompile the library and publish the new version.
* For an app using such libraries:
it's recommended to update your project to Kotlin 2.1.20
and update the dependencies to versions compiled against Compose Multiplatform 1.8.0 and Kotlin 2.1.x.

If you encounter any compatibility issues when upgrading to Compose Multiplatform 1.8.0,
please let us know by filing an issue in [YouTrack](https://youtrack.jetbrains.com/newIssue?project=CMP).

### Migration from Bundle to SavedState in Navigation

Navigation in Compose Multiplatform %composeEapVersion%,
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

Starting with Compose Multiplatform %composeEapVersion%:

```kotlin
composable(Destinations.Followers.route) { navBackStackEntry ->
    val uId = navBackStackEntry.arguments?.read { getStringOrNull("userid") }
    val page = navBackStackEntry.arguments?.read { getStringOrNull("page") }
    if (uId != null && page != null) {
        FollowersMainComposable(navController, accountId = uId, page = page)
    }
}
```

## Across platforms

### Variable fonts

Compose Multiplatform %composeEapVersion% supports variable fonts on all platforms.
With variable fonts, you can keep one font file that contains all style preferences such as weight,
width, slant, italic, custom axes, visual weight with typographic color,
and adaptations to specific text sizes.

For details,
see the [Jetpack Compose documentation](https://developer.android.com/develop/ui/compose/text/fonts#variable-fonts).

### Skia updated to Milestone 132

The version of Skia used by Compose Multiplatform, via Skiko, has been updated to Milestone 132.

The previous version of Skia used was Milestone 126. You can see the changes
made between these versions in the [release notes](https://skia.googlesource.com/skia/+/main/RELEASE_NOTES.md#milestone-132).

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

### Accessibility support improvements

#### Support for right-to-left languages

Compose Multiplatform %composeEapVersion% introduces accessibility support for right-to-left languages,
including proper text direction handling for gestures.

#### Accessibility for scrollable lists

This version improves the performance and accuracy of scrolling boundary and element position calculations.
By carefully accounting for safe areas, such as notches and screen edges,
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

Starting with Compose Multiplatform %composeEapVersion%,
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

In this version, we've introduced support for text fields' accessibility traits.
When a text input field comes into focus, it is now marked as editable,
ensuring proper accessibility-state representation.

You can now also use accessible text input in UI testing.

#### Improved accuracy of accessibility property calculations

We’ve updated the accessibility properties
of Compose Multiplatform components to match the expected behavior of UIKit components.
The UI elements now provide extensive accessibility data,
and any transparent component with an alpha value of 0 no longer provides accessibility semantics.

Aligning semantics also allowed us
to fix several issues related to the incorrect calculation of accessibility properties,
such as missing hitboxes for `DropDown` elements,
mismatches between visible text and accessibility labels, and incorrect radio button states.

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

### Preloading of resources
<secondary-label ref="Experimental"/>

Compose Multiplatform %composeEapVersion% introduces a new experimental API
for preloading fonts and images for web targets.
Preloading helps
prevent visual issues such as flashes of unstyled text (FOUT) or the flickering of images and icons.

The following functions are now available for loading and caching resources:

* `preloadFont()`, which preloads fonts.
* `preloadImageBitmap()`, which preloads bitmap images.
* `preloadImageVector()`, which preloads vector images.

See the [documentation](compose-multiplatform-resources-usage.md#preload-resources-using-the-compose-multiplatform-preload-api) for details.

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

## Desktop

### Support for Windows for ARM64

Compose Multiplatform %composeEapVersion% introduces support for Windows for ARM64 on the JVM,
improving the overall experience of building and running applications on ARM-based Windows devices.
