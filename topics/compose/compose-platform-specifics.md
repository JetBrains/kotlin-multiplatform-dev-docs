[//]: # (title: Default UI behavior on different platforms)

Compose Multiplatform aims to help you produce apps which behave as similarly as possible on different platforms.
On this page, you can learn about unavoidable differences or temporary compromises which
you should expect when writing shared UI code for different platforms
with Compose Multiplatform.

## Project structure

Regardless of the platform you're targeting, each of them needs a dedicated entry point:

* For Android, that's the `Activity`, whose job is to show the main composable from common code.
* For an iOS app, that's the `@main` class or structure that initializes the app.
* For a JVM app, that's the `main()` function that starts the application which launches the main common composable.
* For a Kotlin/JS or Kotlin/Wasm app, that's the `main()` function that attaches the main common code composable
  to the web page.

Certain platform-specific APIs necessary for your app may not have multiplatform support,
and you will have to implement calling these APIs in platform-specific source sets.
Before doing so, check out [klibs.io](https://klibs.io/), a JetBrains project which aims to comprehensively
catalog all available Kotlin Multiplatform libraries.
There are already libraries available for network code, databases, coroutines, and much more.

## Input methods

### Software keyboards

Each platform may handle software keyboards slightly differently, including the way a keyboard appears when a text field becomes active.

Compose Multiplatform adopts the [Compose window insets approach](https://developer.android.com/develop/ui/compose/system/insets)
and imitates it on iOS
to account for [safe areas](https://developer.apple.com/documentation/UIKit/positioning-content-relative-to-the-safe-area).
Depending on your implementation, the software keyboard may be positioned a little differently on iOS.
Make sure to check that the keyboard does not cover important UI elements on both platforms.

### Touch and mouse support

The current desktop implementation interprets all pointer manipulation as mouse gestures
and, therefore, does not support multitouch gestures.
For example, the common pinch-to-zoom gesture cannot be implemented with Compose Multiplatform for desktop
as it requires processing two touches at once.

## UI behavior and appearance

### Platform-specific functionality

Some common UI elements are not covered by Compose Multiplatform and cannot be customized using the framework.
Therefore, you should expect them to look different on different platforms.

Native pop-up views are an example of this:
when you select text in a Compose Multiplatform text field, default suggested actions like **Copy** or **Translate**
are specific to the platform the app is running on.

### Scroll physics

For Android and iOS, the feel of scrolling is aligned with the platform.
For desktop, scrolling support is limited to the mouse wheel (as mentioned in [](#touch-and-mouse-support)). 

### Interop views

If you would like to embed native views within common composables, or vice versa,
you'll need to familiarize yourself with platform-specific mechanisms supported by Compose Multiplatform.

For iOS, there are separate guides for interop code with [SwiftUI](compose-swiftui-integration.md) and [UIKit](compose-uikit-integration.md).

For desktop, Compose Multiplatform supports [](compose-desktop-swing-interoperability.md).

### Back gesture

Android devices have back gesture support by default, and every screen reacts to the **Back** button in some fashion.

On iOS, there is no back gesture by default, although developers are encouraged to implement similar functionality
to meet user experience expectations.
Compose Multiplatform for iOS supports back gestures by default to mimic Android functionality.

On desktop, Compose Multiplatform uses the **Esc** key as the default back trigger.

For details, see the [](compose-navigation.md#back-gesture) section.

### Text

With text, Compose Multiplatform doesn't guarantee pixel-perfect correspondence between different platforms:

* If you don't set a font explicitly, each system assigns a different default font to your text.
* Even for the same font, letter aliasing mechanisms specific to each platform can lead to noticeable differences.

This doesn't have a significant impact on user experience. On the contrary, the default fonts appear as expected on each platform.
However, pixel differences may interfere with screenshot testing, for example.

<!-- this should be covered in benchmarking, not as a baseline Compose Multiplatform limitation 
### Initial performance

On iOS, you may notice a delay in the initial performance of individual screens compared to Android.
This can happen because Compose Multiplatform compiles UI shaders on demand.
So, if a particular shader is not cached yet, compiling it may delay rendering of a scene.

This issue affects only the first launch of each screen.
Once all necessary shaders are cached, subsequent launches are not delayed by compilation.
-->

## Developer experience

### Previews

_Previews_ are non-interactable layout presentations of composables available in the IDE.

To see a preview for a composable:

1. Add an Android target to your project if there isn't one (the preview mechanism uses an Android library).
2. Mark composables that you would like to be previewable with the `@Preview` annotation in common code.
3. Switch to the **Split** or **Design** view in the editor window.
     It prompts you to build the project for the first time if you haven't done that yet.

In both IntelliJ IDEA and Android Studio, you will be able to see the initial layout of every composable
annotated with `@Preview` in the current file.

### Hot reload

_Hot reload_ refers to the app reflecting code changes on the fly without requiring additional input.
In Compose Multiplatform, hot reload functionality is available only for JVM (desktop) targets.
However, you can use it to quickly troubleshoot issues before switching to your intended platforms for fine-tuning.

To learn more, see our [](compose-hot-reload.md) article. 

## What's next

Read more about Compose Multiplatform implementation for the following components:
  * [Resources](compose-multiplatform-resources.md)
  * [](compose-lifecycle.md)
  * [](compose-viewmodel.md)
  * [](compose-navigation-routing.md)
