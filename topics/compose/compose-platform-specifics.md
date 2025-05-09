[//]: # (title: Default UI behavior on different platforms)

Compose Multiplatform aims to help you produce apps which behave as similar as possible on different platforms.
On this page, you can learn about unavoidable differences or temporary compromises which
you should expect when writing shared UI code for different platforms
with Compose Multiplatform.

## Project structure

Regardless of the platform you're targeting, each of them needs a dedicated entry point:

* For Android, that's the Activity whose job it is to show the main composable from common code.
* For an iOS app, that's the `@main` class or structure that initializes the app.
* For a JVM app, that's the `main()` function that starts the application which launches the main common composable.
* For a Kotlin/JS or Kotlin/Wasm app, that's the `main()` function that attaches the main common code composable
  to the web page.

### Platform-specific APIs



## Input methods

### IME keyboards

### Touch and mouse support

Current desktop implementation interprets all pointer manipulation as mouse gestures
and, therefore, does not support multitouch gestures.
For example, the common pinch-to-zoom gesture cannot be implemented with Compose Multiplatform for desktop
as it requires processing two touches at once.

## UI behavior and appearance

### System functionality

Some UI elements depend on system integration and may look different in default configuration,
for example:

* The "Share" actions use system APIs to offer a list of specific ways to share or send content.
* The popup-views (for example, for selected text) are fully native and currently cannot be customized
    through Compose Multiplatform.

### Scroll physics

For Android and iOS, the feel of scrolling is aligned with the platform.
For desktop, scrolling support is limited since the JVM target by default doesn't support
common mobile gesture patterns like fling or overscroll (often used for the "pull to refresh" action).

### Interop views

### Back gesture

Android devices have back gesture support by default, and every screen reacts to the **Back** button in some fashion.

On iOS, there is no back gesture by default, although developers 

### Text

With text, Compose Multiplatform doesn't guarantee pixel-perfect correspondence between different platforms:

* If you don't set a font explicitly, each system assigns a different default font to your text.
* Even for the same font, letter aliasing mechanisms specific to each platform can lead to noticeable differences.

This doesn't have a significant impact on user experience, but may get in the way of screenshot testing, for example.

### Start times

On iOS, you may notice a delay in initial loading of the app compared to Android.
This can happen if you have a lot of shaders to be compiled.
Shaders are not only relevant for apps using 3D models:
if your app is complicated enough, it may include a sufficient amount of ordinary UI widgets that need to be pre-built
at startup.

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

_Hot reload_ is the experience of the app reflecting changes in code without additional input, on the fly.
For Compose Multiplatform, the hot reload functionality is available only for JVM (desktop) targets,
but you can use it to quickly iron out problems before switching to your actual target platforms for fine-tuning.

TODO: link the CMP hot reload repo, or the doc page if it's there already 

## What's next

Read more about Compose Multiplatform implementation for the following components:
  * [Resources](compose-multiplatform-resources.md)
  * [](compose-lifecycle.md)
  * [](compose-viewmodel.md)
  * [](compose-navigation-routing.md)
