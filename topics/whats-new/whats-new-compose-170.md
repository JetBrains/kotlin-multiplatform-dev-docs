[//]: # (title: What's new in Compose Multiplatform 1.7.0-beta01)

Here are the highlights for this feature release:

* [Type safe navigation is here](#type-safe-navigation)
* [Shared element transitions in Compose Multiplatform](#shared-element-transitions)
* [Compose Multiplatform resources are now packed into Android assets](#multiplatform-resources-are-now-packed-into-android-assets)

See the full list of changes for this release [on GitHub](https://github.com/JetBrains/compose-multiplatform/blob/master/CHANGELOG.md#170-beta01-august-2024). 

## Dependencies

* Gradle plugin `org.jetbrains.compose`, version 1.7.0-beta01. Based on Jetpack Compose libraries:
  * [Runtime 1.7.0](https://developer.android.com/jetpack/androidx/releases/compose-runtime#1.7.0)
  * [UI 1.7.0](https://developer.android.com/jetpack/androidx/releases/compose-ui#1.7.0)
  * [Foundation 1.7.0](https://developer.android.com/jetpack/androidx/releases/compose-foundation#1.7.0)
  * [Material 1.7.0](https://developer.android.com/jetpack/androidx/releases/compose-material#1.7.0)
  * [Material3 1.3.0](https://developer.android.com/jetpack/androidx/releases/compose-material3#1.3.0)
* Lifecycle libraries `org.jetbrains.androidx.lifecycle:lifecycle-*:2.8.0`. Based on [Jetpack Lifecycle 2.8.0](https://developer.android.com/jetpack/androidx/releases/lifecycle#2.8.0).
* Navigation libraries `org.jetbrains.androidx.navigation:navigation-*:2.7.0-alpha08`. Based on [Jetpack Navigation 2.8.0-beta05](https://developer.android.com/jetpack/androidx/releases/navigation#2.8.0-beta05).

## Breaking change: minimum Android Gradle plugin version is 8.1.0

Neither Jetpack Compose 1.7.0 nor Lifecycle 2.8.0, which are used by Compose Multiplatform 1.7.0, supports AGP 7.
So updating to the latest Compose Multiplatform will involve upgrading AGP as well.

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

### Resources

#### Multiplatform resources are now packed into Android assets

Multiplatform resources are now packed into Android assets, which helps Android Studio generate previews for Compose Multiplatform
composables in Android source sets.

Packing Android assets also fixes rendering multiplatform resources in WebViews and media player components.

#### Custom resource directories

With the new `customDirectory` setting in the configuration DSL you can associate a custom directory with a specific source
set. This allows, for example, using downloaded files as resources.

### BasicTextField2 stabilized and renamed into BasicTextField

The `BasicTextField2` component, introduced in Compose Multiplatform 1.6.0, is out of its transitional status and is renamed
into `BasicTextField`, replacing the old component.

The new `BasicTextField`:
* allows managing state more reliably
* offers new `TextFieldBuffer` API for programmatic changes to the text field content 
* contains several new APIs for visual transformations and styling
* gives access to `UndoState` with the return to previous states of the field.

### GraphicsLayer: new drawing API

The new drawing layer added in Jetpack Compose 1.7.0 is available in Compose Multiplatform.

Unlike `Modifier.graphicsLayer`, the new GraphicsLayer class allows for rendering of Composable content anywhere
and is useful in animated use cases where content is expected to be rendered in different scenes.

See the [reference documentation](https://developer.android.com/reference/kotlin/androidx/compose/ui/graphics/layer/GraphicsLayer)
for a more detailed description and example usage.

### LocalLifecycleOwner moved out of Compose UI

The `LocalLifecycleOwner` class is moved from the Compose UI package to the Lifecycle package.
This allows you to access the class and call its Compose-based helper APIs independently of a Compose UI.

## iOS

### Touch interop between Compose Multiplatform and native iOS is improved

In this release, the touch handling on iOS interop views becomes more sophisticated:
Compose Multiplatform now tries to detect whether a touch is meant for an interop view or should be processed by Compose.
This gives you a chance to process a touch event that happens in a UIKit area inside your Compose Multiplatform app.

This introduces a small delay (150 ms) in UI response, however.
If you’re certain you won’t need to process these touches in Compose, you can turn this behavior off with the new `areTouchesDelayed`
parameter of `UIKitView` and `UIKitViewController` APIs.

## Desktop

### Implemented drag and drop

When your want to provide to your users an ability to move things between composables, you can use the `dragAndDropSource`
`dragAndDropTarget` modifiers to specify potential sources and destinations for drag operations.

For details see the [Drag and drop](https://developer.android.com/develop/ui/compose/touch-input/user-interactions/drag-and-drop)
article about the corresponding modifiers in Jetpack Compose documentation.

### Render settings for ComposePanel

By specifying the new `RenderSettings` parameter in the `ComposePanel` constructor you can enable vertical synchronization
and thus try to reduce visual latency between input and changes in the UI.
Screen tearing is a potential side effect.

## iOS

