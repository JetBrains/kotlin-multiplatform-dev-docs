[//]: # (title: What's new in Compose Multiplatform 1.7.0-beta01)

Here are the highlights for this feature release:

* [Breaking change: Android Gradle plugin minimum version raised to 8.1.0](#breaking-change-minimum-android-gradle-plugin-version-is-8-1-0)
* [Type safe navigation is here](#lifecycle-library)
* [Shared element transitions in Compose Multiplatform](#experimental-navigation-library)
* [Compose Multiplatform resources are now packed into Android assets](#support-for-multimodule-projects-with-compose-multiplatform-resources)
* [The new BasicTextField is stable](#experimental-navigation-library)
* [Known issue: MissingResourceException](#known-issue-missingresourceexception)

See the full list of changes for this release [on GitHub](https://github.com/JetBrains/compose-multiplatform/blob/master/CHANGELOG.md#170-beta01-august-2024). 

## Dependencies

TODO

* Gradle plugin `org.jetbrains.compose`, version 1.6.10. Based on Jetpack Compose libraries:
  * [Compiler 1.5.14](https://developer.android.com/jetpack/androidx/releases/compose-compiler#1.5.14)
  * [Runtime 1.6.7](https://developer.android.com/jetpack/androidx/releases/compose-runtime#1.6.7)
  * [UI 1.6.7](https://developer.android.com/jetpack/androidx/releases/compose-ui#1.6.7)
  * [Foundation 1.6.7](https://developer.android.com/jetpack/androidx/releases/compose-foundation#1.6.7)
  * [Material 1.6.7](https://developer.android.com/jetpack/androidx/releases/compose-material#1.6.7)
  * [Material3 1.2.1](https://developer.android.com/jetpack/androidx/releases/compose-material3#1.2.1)
* Lifecycle libraries `org.jetbrains.androidx.lifecycle:lifecycle-*:2.8.0`. Based on [Jetpack Lifecycle 2.8.0](https://developer.android.com/jetpack/androidx/releases/lifecycle#2.8.0).
* Navigation libraries `org.jetbrains.androidx.navigation:navigation-*:2.7.0-alpha07`. Based on [Jetpack Navigation 2.7.7](https://developer.android.com/jetpack/androidx/releases/navigation#2.7.7).

## Breaking change: minimum Android Gradle plugin version is 8.1.0

Neither Jetpack Compose 1.7.0 nor Lifecycle 2.8.0, which are used by Compose Multiplatform 1.7.0, supports AGP 7.
So updating to the latest Compose Multiplatform will involve upgrading AGP as well.

## Across platforms

### Shared element transitions

You can now implement transitions between composables with consistent content in exactly the way it is [implemented
in Jetpack Compose](https://developer.android.com/develop/ui/compose/animation/shared-elements).

TODO: link to an example?

### Resources

#### Multiplatform resources are now packed into Android assets

Now all multiplatform resources are packaged into Android assets, which allows using these resources in WebViews and other
external consumers.

TODO is this working for AS previews in the end?

#### Custom resource directories

With the new `customDirectory` setting in the configuration DSL you can associate a custom directory with a specific source
set. This allows, for example, using downloaded files as resources.

TODO: this needs documentation and a usage example for sure

### Type safe Navigation

New APIs implemented in Navigation 2.8.0 allow Compose to provide compile-time safety for your navigation graph.
These APIs achieve the same result as the [Safe Args](https://developer.android.com/guide/navigation/use-graph/pass-data#Safe-args)
plugin for XML-based navigation.

For details, see the [Navigation docs](https://developer.android.com/guide/navigation/design/type-safety).

### BasicTextField

The `BasicTextField2` component, introduced in Compose Multiplatform 1.6.0, is out of its transitional status and is renamed
into `BasicTextField`, replacing the old component.

The new `BasicTextField`:
* allows to manage state more reliably
* offers new `TextFieldBuffer` API for programmatic changes to the text field content 
* contains several new APIs for visual transformations and styling
* gives access to `UndoState` with the return to previous states of the field.

TODO should we mention that only one overload is getting renamed? https://android-review.googlesource.com/c/platform/frameworks/support/+/2953777

### dragAndDrop (Android, iOS, desktop)

When your want to provide to your users an ability to move things between composables, you can use the `dragAndDropSource`
`dragAndDropTarget` modifiers to specify potential sources and destinations for drag operations.

## Desktop

### Render settings for ComposePanel

TODO

https://github.com/JetBrains/compose-multiplatform-core/pull/1377

By specifying the new `RenderSettings` parameter in the `ComposePanel` constructor you can try to reduce visual latency
between input and changes in the UI by enabling vertical synchronization. Screen tearing is a potential side effect.

## iOS

## Web

## Gradle plugin

### Known issues