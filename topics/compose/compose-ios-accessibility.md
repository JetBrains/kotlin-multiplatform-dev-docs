[//]: # (title: Support for iOS accessibility features)

Compose Multiplatform for iOS allows people with disabilities to interact with Compose UI with the same level of comfort
as with native UIs:
* screen readers and VoiceOver can access the content of the Compose UI,
* Compose UI supports the same gestures as the native UIs for navigation and interaction. 

This is possible because semantics data produced by Compose APIs is now mapped to native accessibility objects and properties
that are consumed by iOS Accessibility Services. For most interfaces built with Material widgets, this should happen
automatically.

You can also use this semantic data in testing and other automation: properties such as `testTag` will correctly map
to native accessibility properties such as `accessibilityIdentifier`.

## Customizing accessibility tree synchronization

By default, the iOS accessibility tree is synchronized with the UI only when the accessibility services are running;
no accessibility logs are written. You can customize this behavior, for example, when you want to debug interactions:
set the option to always synchronize the accessibility tree, so that the tree is rewritten every time the UI updates.

Compose Multiplatform provides an API to allow this: you can configure the tree sync behavior
by modifying `accessibilitySyncOptions` inside the `configure` block of a `ComposeUIViewController` call:

```kotlin
 ComposeUIViewController(configure = {
     accessibilitySyncOptions = AccessibilitySyncOptions.Always(object: AccessibilityDebugLogger {
         override fun log(message: Any?) {
             if (message == null) {
                 println()
             } else {
                 println("[a11y]: $message")
             }
         }
     })
 }) {
    // your @Composable content
 }
```

> AccessibilitySyncOptions.Always can be quite handy for debugging and testing, but be aware that there is a significant
> overhead associated with it. Synchronizing the tree with the UI regardless of Accessibility Services can degrade
> the performance of the app.
>
> {type="tip"}

## Accessibility APIs

The following APIs provided by Compose Multiplatform help you customize iOS accessibility behavior.

### AccessibilityDebugLogger

The `log` function of the `AccessibilityDebugLogger` interface adds a message to the debug logger, or null
as a separator of log blobs.

```kotlin
// package androidx.compose.ui.platform

@ExperimentalComposeApi
interface AccessibilityDebugLogger {

    // Logs the message to the debug logger, or null as a separator of log blobs
    fun log(message: Any?)
}
```

### AccessibilitySyncOptions

The class contains available options for synchronizing the accessibility tree.

```kotlin
// package androidx.compose.ui.platform

@ExperimentalComposeApi
sealed class AccessibilitySyncOptions {
    
    // Option to never synchronize the accessibility tree.
    object Never: AccessibilitySyncOptions

    // Option to synchronize the tree only when the accessibility services are running.
    // You can include an AccessibilityDebugLogger to log interactions and tree syncing events.
    class WhenRequiredByAccessibilityServices(debugLogger: AccessibilityDebugLogger?)

    // Option to synchronize the tree only when the accessibility services are running.
    // You can include an AccessibilityDebugLogger to log interactions and tree syncing events.
    class Always(debugLogger: AccessibilityDebugLogger?)
}
```

