[//]: # (title: Default UI behavior on different platforms)

Compose Multiplatform aims to help you produce apps which behave as similar as possible on different platforms.
On this page, you can learn about unavoidable differences or temporary compromises which
you should expect when writing shared UI code for different platforms
with Compose Multiplatform.

## Project structure

Regardless of the platform you're targeting, each of them needs a dedicated entry point:

* For Android, that's the Activity whose job it is to show the main composable from common code.
* For an iOS app, that's the `@main` class or structure that initializes the app.
* For a JVM app, that's the `main()` function that starts the application.
* For a Kotlin/JS or Kotlin/Wasm app, that's the `main()` function that attaches the main common code composable
  to the web page.

## UI look and feel

### Scroll physics

### Shadows

### Interop views

### Text

## Input methods

### IME keyboards

### Desktop specific

Current desktop implementation interprets all pointer manipulation as mouse gestures
and, therefore, does not support multitouch gestures.
For example, the common pinch-to-zoom gesture cannot be implemented with Compose Multiplatform for desktop
as it requires processing two touches at once.

## Developer experience

### Previews

_Previews_ are non-interactable layout presentations of composables available in the IDE.

To see a preview for a composable:

1. Add an Android target to your project if there isn't one (the preview mechanism uses an Android library).
2. Mark composables that you would like to be previewable with the `@Preview` annotation in common code.
3. Switch to the **Split** or **Design** view in the editor window, which will prompt you to build the project for the
    project for the first time if you haven't done that yet.

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
