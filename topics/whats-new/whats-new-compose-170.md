[//]: # (title: What's new in Compose Multiplatform 1.7.0-beta01)

Here are the highlights for this feature release:

* [Type safe navigation](#type-safe-navigation)
* [Shared element transitions](#shared-element-transitions)
* [Multiplatform resources are now packed into Android assets](#resources-packed-into-android-assets)
* [Custom resource directories](#custom-resource-directories)
* [Improved touch interop on iOS](#new-default-behavior-for-processing-touch-in-ios-native-elements)
* [Material3 `adaptive` now in common code](#material3-adaptive-adaptive)
* [Drag and drop implemented on desktop](#drag-and-drop-implemented)
* [`BasicTextField` adopted on desktop](#basictextfield2-stabilized-and-renamed-into-basictextfield)

See the full list of changes for this release [on GitHub](https://github.com/JetBrains/compose-multiplatform/blob/master/CHANGELOG.md#170-beta01-september-2024).

## Dependencies

* Gradle plugin `org.jetbrains.compose`, version 1.7.0-beta01. Based on Jetpack Compose libraries:
  * [Runtime 1.7.0-rc01](https://developer.android.com/jetpack/androidx/releases/compose-runtime#1.7.0-rc01)
  * [UI 1.7.0-rc01](https://developer.android.com/jetpack/androidx/releases/compose-ui#1.7.0-rc01)
  * [Foundation 1.7.0-rc01](https://developer.android.com/jetpack/androidx/releases/compose-foundation#1.7.0-rc01)
  * [Material 1.7.0-rc01](https://developer.android.com/jetpack/androidx/releases/compose-material#1.7.0-rc01)
  * [Material3 1.3.0-rc01](https://developer.android.com/jetpack/androidx/releases/compose-material3#1.3.0-rc01)
* Lifecycle libraries `org.jetbrains.androidx.lifecycle:lifecycle-*:2.8.1`. Based on [Jetpack Lifecycle 2.8.4](https://developer.android.com/jetpack/androidx/releases/lifecycle#2.8.4).
* Navigation libraries `org.jetbrains.androidx.navigation:navigation-*:2.8.0-alpha09`. Based on [Jetpack Navigation 2.8.0-beta05](https://developer.android.com/jetpack/androidx/releases/navigation#2.8.0-beta05).
* Material3 Adaptive libraries `org.jetbrains.compose.material3.adaptive:adaptive-*:1.0.0-rc01`. Based on [Jetpack Material3 Adaptive 1.0.0-rc01](https://developer.android.com/jetpack/androidx/releases/compose-material3-adaptive#1.0.0-rc01)

## Breaking changes

### Minimum AGP version raised to 8.1.0

Neither Jetpack Compose 1.7.0 nor Lifecycle 2.8.0, which are used by Compose Multiplatform 1.7.0, supports AGP 7.
So when you update to Compose Multiplatform 1.7.0, you may have to upgrade your AGP dependency as well.

> Newly implemented previews for Android composables in Android Studio [require one of the latest AGP versions](#resources-packed-into-android-assets).
>
{type="note"}

### New default behavior for processing touch in iOS native elements

Before, Compose Multiplatform did not (and could not) respond to touch events that landed in interop UI views.
So interop views handled these touch sequences entirely.

In 1.7.0, Compose Multiplatform implements more sophisticated logic for handling interop touch sequences.
By default, there is a delay after the initial touch that helps the parent composable understand whether
the touch sequence was meant to interact with the native view and react accordingly.

See the more detailed explanation in the [iOS section of this page](#ios-touch-interop) 
or read the [documentation for this feature](compose-ios-touch.md).

### Disabling minimum frame duration on iOS is mandatory

Developers often failed to notice the printed warning about the high refresh rate displays,
and users were deprived of smooth animations on their 120-Hz-enabled devices.
So we are enforcing this check to be strict: apps built with Compose Multiplatform will now crash
if the `CADisableMinimumFrameDurationOnPhone` property in the `Info.plist` file is absent or set to `false`.

You can disable this behavior by setting the `ComposeUIViewControllerConfiguration.enforceStrictPlistSanityCheck` property to `false`.

## Across platforms

### Shared element transitions

Compose Multiplatform implements the API for seamless transition between composables that should have consistent elements between them.
These transitions are often useful in navigation to help users follow their trajectory in the UI.

For a deep dive into the API, see the [Jetpack Compose documentation](https://developer.android.com/develop/ui/compose/animation/shared-elements).

### Type safe Navigation

The safe Jetpack Compose approach to passing objects along a navigation route is implemented in Compose Multiplatform as well.
New APIs implemented in Navigation 2.8.0 allow Compose to provide compile-time safety for your navigation graph.
These APIs achieve the same result as the [Safe Args](https://developer.android.com/guide/navigation/use-graph/pass-data#Safe-args)
plugin for XML-based navigation.

For details, see the [Google’s Navigation docs about type safety](https://developer.android.com/guide/navigation/design/type-safety).

### Multiplatform resources

#### Resources packed into Android assets

All multiplatform resources are now packed into Android assets. This helps Android Studio generate previews for Compose Multiplatform composables in Android source sets.

This also allows direct access to multiplatform resources from WebViews and media player components on Android, since resources can be reached by a simple path, for example `Res.getUri(“files/index.html”)`

> Android Studio previews are available only for composables in an Android source set.
> They also require one of the latest versions of AGP: 8.5.2, 8.6.0-rc01, or 8.7.0-alpha04.
>
{type="note"}

#### Custom resource directories

With the new `customDirectory` setting in the configuration DSL you can [associate a custom directory](compose-images-resources.md#custom-resource-directories) with a specific source
set. This allows, for example, using downloaded files as resources.

#### Multiplatform font cache

Compose Multiplatform brings the Android font cache functionality to other platforms, too,
to avoid excessive byte-reading of `Font` resources.

#### Multiplatform resources available in test source sets

Now you can:

* Add resources to test source sets.
* Use generated accessors that are available only in corresponding source sets.
* Pack test resources into the app only for test runs.

#### Resources mapped to string IDs for easy access

Resources of each type are mapped with their file names: for example, you can use the `Res.allDrawableResources` function
to get the map of all `drawable` resources and access the necessary resource by passing the string ID:

```kotlin
Image(painterResource(Res.allDrawableResources["compose_multiplatform"]!!), null)
```

#### Functions for converting byte arrays into ImageBitmap or ImageVector

There are new functions for converting a `ByteArray` into an image resource:

* `decodeToImageBitmap()` for turning a JPEG, PNG, BMP, or WEBP file into an `ImageBitmap` object.
* `decodeToImageVector()` for turning an XML vector file into an `ImageVector` object.
* `decodeToSvgPainter()` for turning an SVG file into a `Painter` object. This function is not available on Android.

See the [documentation](compose-images-resources.md#convert-byte-arrays-into-images) for details.

### New common modules

#### material3.adaptive:adaptive*

Material3 adaptive modules are available in common code with Compose Multiplatform.
You need to explicitly set corresponding dependencies to use them:

```kotlin
commonMain.dependencies {
  implementation("org.jetbrains.compose.material3.adaptive:adaptive:1.0.0-rc01")
  implementation("org.jetbrains.compose.material3.adaptive:adaptive-layout:1.0.0-rc01")
  implementation("org.jetbrains.compose.material3.adaptive:adaptive-navigation:1.0.0-rc01")
}
````

#### material3:material3-window-size-class

The `material3-window-size-class` dependency should be explicitly added to the list of common dependencies:

```kotlin
commonMain.dependencies {
  implementation("org.jetbrains.compose.material3:material3-window-size-class:1.7.0-beta01")
}
```

The `calculateWindowSizeClass` function is not available in common code yet.
But you can import and call it in platform-specific code, for example:

```kotlin
// desktopMain/kotlin/main.kt
import androidx.compose.material3.windowsizeclass.calculateWindowSizeClass

// ...

val size = calculateWindowSizeClass()
```

#### material-navigation

The material-navigation library is available in common code alongside Compose Multiplatform navigation.
To use it, add these explicit dependencies to your common source set:

```kotlin
commonMain.dependencies {
  implementation("org.jetbrains.androidx.navigation:navigation-compose:2.8.0-alpha09")
  implementation("org.jetbrains.compose.material:material-navigation:1.7.0-beta01")
}
```

### Skia updated to Milestone 126

The version of Skia, used by Compose Multiplatform via [Skiko](https://github.com/JetBrains/skiko), has been updated to Milestone 126.

The previous version of Skia used was Milestone 116. You can see the changes made between these versions in the [release notes](https://skia.googlesource.com/skia/+/refs/heads/main/RELEASE_NOTES.md#milestone-126).

### GraphicsLayer: new drawing API

The new drawing layer added in Jetpack Compose 1.7.0 is available in Compose Multiplatform.

Unlike `Modifier.graphicsLayer`, the new `GraphicsLayer` class allows for rendering of Composable content anywhere
and is useful in animated use cases where content is expected to be rendered in different scenes.

See the [reference documentation](https://developer.android.com/reference/kotlin/androidx/compose/ui/graphics/layer/GraphicsLayer)
for a more detailed description and example usage.

### LocalLifecycleOwner moved out of Compose UI

The `LocalLifecycleOwner` class is moved from the Compose UI package to the Lifecycle package.

This allows you to access the class and call its Compose-based helper APIs independently of a Compose UI.
But keep in mind that without the Compose UI bindings a `LocalLifecycleOwner` instance will have no platform integration
and thus no platform-specific events to listen to.

## iOS

### Touch interop between Compose Multiplatform and native iOS is improved {id="ios-touch-interop"}

In this release, the touch handling on iOS interop views becomes more sophisticated:
Compose Multiplatform now tries to detect whether a touch is meant for an interop view or should be processed by Compose.
This gives you a chance to process a touch event that happens in a UIKit or a SwiftUI area inside your Compose Multiplatform app.

By default, Compose Multiplatform will delay transmitting touch events to interop views by 150 ms:

* If within this time frame there is movement over a minor distance threshold,
  the parent composable will intercept the touch sequence and not forward it to the interop view.
* If there is no noticeable movement, Compose will resign from processing the rest of the touch sequence.
  The rest of the sequence will be processed solely by the interop view.

This behavior aligns with how native [UIScrollView](https://developer.apple.com/documentation/uikit/uiscrollview) works.
It helps to avoid situations where a touch sequence that started in the interop view ends up intercepted by the interop view
without a chance for Compose Multiplatform to process it. This can lead to frustrating user experience, for example,
when large interop views (such as video players) are used in a scrollable context such as lazy list: it is tricky
to scroll the list when most of the screen is taken by the interop view that intercepts all touches without Compose Multiplatform being aware of them.

## Desktop

### Drag and drop implemented

The drag and drop mechanism, allowing for dragging content into or out of your Compose application,
is implemented in Compose Multiplatform for desktop.
You can use the `dragAndDropSource` and `dragAndDropTarget` modifiers to specify potential sources and destinations for drag operations.

> While these modifiers are available in common code, for now they will only work in desktop and Android source sets.
> Stay tuned for future releases.
> 
{type="note"}

For common use cases, see the [Drag and drop](https://developer.android.com/develop/ui/compose/touch-input/user-interactions/drag-and-drop)
article in Jetpack Compose documentation.

### BasicTextField2 stabilized and renamed into BasicTextField

Jetpack Compose stabilized and renamed the `BasicTextField2` component into `BasicTextField`.
In this release, Compose Multiplatform adopted the change for desktop targets, planning to cover iOS as well
in the stable 1.7.0 version.

The new `BasicTextField`:
* Allows managing state more reliably.
* Offers the new `TextFieldBuffer` API for programmatic changes to the text field content.
* Contains several new APIs for visual transformations and styling.
* Provides access to `UndoState` with the ability to return to previous states of the field.

### Render settings for ComposePanel

By specifying the new `RenderSettings.isVsyncEnabled` parameter in the `ComposePanel` constructor you can hint to the backend
rendering implementation to disable vertical synchronization.
This can reduce visual latency between input and changes in the UI but can also lead to screen tearing.

The default behavior stays the same: attempt to synchronize drawable presentations with VSync.
