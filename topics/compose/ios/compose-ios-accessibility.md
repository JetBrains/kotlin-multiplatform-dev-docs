[//]: # (title: Support for iOS accessibility features)

Compose Multiplatform accessibility support allows people with disabilities to interact with the Compose Multiplatform UI
as comfortably as with the native iOS UI:

* Screen readers and VoiceOver can access the content of the Compose Multiplatform UI.
* The Compose Multiplatform UI supports the same gestures as the native iOS UI for navigation and interaction.

This is possible because semantics data produced by Compose APIs is now mapped to native objects and properties
that are consumed by iOS Accessibility Services. For most interfaces built with Material widgets, this should happen
automatically.

You can also use this semantic data in testing and other automation: properties such as `testTag` will correctly map
to native accessibility properties such as `accessibilityIdentifier`. This makes semantic data from Compose Multiplatform 
available to Accessibility Services and XCTest framework.

## High-contrast theme

Compose Multiplatform uses the [`ColorScheme`](https://kotlinlang.org/api/compose-multiplatform/material3/androidx.compose.material3/-color-scheme/)
class from the material3 library, which currently lacks out-of-the-box support for high-contrast colours. 
For a high-contrast theme on iOS, you need to add an extra set of colours 
to the application palette. For each custom colour, its high-contrast version should be specified manually.

When you provide an additional high-contrast color palette, users need to manually select it, as it will not be 
automatically triggered by iOS's system-level accessibility settings.

While defining a color palette, use a WCAG-compliant contrast checker tool to verify that your chosen `onPrimary` color has 
sufficient contrast with your primary color, `onSurface` with a surface color, and so on.
Ensure the contrast ratio between colors is at least 4.5:1. For custom foreground and background colors, 
the contrast ratio should be 7:1, especially for small text. This applies to both your `lightColorScheme` 
and `darkColorScheme`.

This code shows how to define high-contrast color palettes for light and dark themes:

```kotlin
import androidx.compose.ui.graphics.Color

// Defines a data class to hold the color palette for high-contrast themes
data class HighContrastColors(
    val primary: Color, // Main interactive elements, primary text, top app bars
    val onPrimary: Color, // Content displayed on top of a 'primary' color
    val secondary: Color, // Secondary interactive elements, floating action buttons
    val onSecondary: Color, // Content displayed on top of a 'secondary' color
    val tertiary: Color, // An optional third accent color
    val onTertiary: Color, // Content displayed on top of a 'tertiary' color
    val background: Color, // Main background of the screen
    val onBackground: Color, // Content displayed on top of a 'background' color
    val surface: Color, // Card backgrounds, sheets, menus, elevated surfaces
    val onSurface: Color, // Content displayed on top of a 'surface' color
    val error: Color, // Error states and messages
    val onError: Color, // Content displayed on top of an 'error' color
    val success: Color, // Success states and messages
    val onSuccess: Color, // Content displayed on top of a 'success' color
    val warning: Color, // Warning states and messages
    val onWarning: Color, // Content displayed on top of a 'warning' color
    val outline: Color, // Borders, dividers, disabled states
    val scrim: Color // Dimming background content behind modals/sheets
)

// Neutral colors
val Black = Color(0xFF000000)
val White = Color(0xFFFFFFFF)
val DarkGrey = Color(0xFF1A1A1A)
val MediumGrey = Color(0xFF888888)
val LightGrey = Color(0xFFE5E5E5)

// Primary accent colors
val RoyalBlue = Color(0xFF0056B3)
val SkyBlue = Color(0xFF007AFF)

// Secondary  and tertiary accent colors
val EmeraldGreen = Color(0xFF28A745)
val GoldenYellow = Color(0xFFFFC107)
val DeepPurple = Color(0xFF6F42C1)

// Status colors
val ErrorRed = Color(0xFFD32F2F)
val SuccessGreen = Color(0xFF388E3C)
val WarningOrange = Color(0xFFF57C00)

// Light high-contrast palette, dark content on light backgrounds
val LightHighContrastPalette =
    HighContrastColors(
        primary = RoyalBlue,
        onPrimary = White,
        secondary = EmeraldGreen,
        onSecondary = White,
        tertiary = DeepPurple,
        onTertiary = White,
        background = White,
        onBackground = Black,
        surface = LightGrey,
        onSurface = DarkGrey,
        error = ErrorRed,
        onError = White,
        success = SuccessGreen,
        onSuccess = White,
        warning = WarningOrange,
        onWarning = White,
        outline = MediumGrey,
        scrim = Black.copy(alpha = 0.6f)
    )

// Dark high-contrast palette, light content on dark backgrounds
val DarkHighContrastPalette =
    HighContrastColors(
        primary = SkyBlue,
        onPrimary = Black,
        secondary = EmeraldGreen,
        onSecondary = White,
        tertiary = GoldenYellow,
        onTertiary = Black,
        background = Black,
        onBackground = White,
        surface = DarkGrey,
        onSurface = LightGrey,
        error = ErrorRed,
        onError = White,
        success = SuccessGreen,
        onSuccess = White,
        warning = WarningOrange,
        onWarning = White,
        outline = MediumGrey,
        scrim = Black.copy(alpha = 0.6f)
    )
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="val LightHighContrastPalette = HighContrastColors( primary = RoyalBlue,"}

## Control via trackpad and keyboard

Compose Multiplatform for iOS supports additional input methods to control your device. Instead of relying on the 
touchscreen, you can enable either AssistiveTouch to use a mouse or trackpad, or Full Keyboard Access to use a keyboard:

* AssistiveTouch (**Settings** | **Accessibility** | **Touch** | **AssistiveTouch**) allows you to control your 
  iPhone or iPad with a pointer from a connected mouse or trackpad. You can use the pointer to click icons on your 
  screen, navigate through the AssistiveTouch menu, or type using the onscreen keyboard.
* Full Keyboard Access (**Settings** | **Accessibility** | **Keyboards** | **Full Keyboard Access**) enables device 
  control with a connected keyboard. You can navigate with keys like **Tab** and activate items using **Space**.

## Customize synchronization of the accessibility tree 

With default settings:
* The iOS accessibility tree is synchronized with the UI only when Accessibility Services are running.
* Synchronization events are not logged.

You can customize these settings with the new Compose Multiplatform API.

### Choose the tree synchronization option

> In Compose Multiplatform %composeVersion%, [this option has been removed](whats-new-compose-180.md#loading-accessibility-tree-on-demand)
> as the accessibility tree is synchronized lazily and no longer requires additional configuration.
>
{style="tip"}

To debug and test events and interactions, you can change the synchronization mode to:
* Never synchronize the tree with UI, for example, to temporarily disable accessibility mapping.
* Always synchronize the tree so that it is rewritten every time the UI updates to test the accessibility integration thoroughly.

> Remember that synchronizing the tree after each UI event can be quite useful for debugging and testing but can degrade
> the performance of your app.
>
{style="note"}

An example of enabling the option to always synchronize the accessibility tree:

```kotlin
ComposeUIViewController(configure = {
    accessibilitySyncOptions = AccessibilitySyncOptions.Always(debugLogger = null)
}) {
    // your @Composable content
}
```

The `AccessibilitySyncOptions` class contains all available options:

```kotlin
// package androidx.compose.ui.platform

@ExperimentalComposeApi
sealed class AccessibilitySyncOptions {
    
    // Option to never synchronize the accessibility tree
    object Never: AccessibilitySyncOptions

    // Option to synchronize the tree only when Accessibility Services are running
    //
    // You can include an AccessibilityDebugLogger to log interactions and tree syncing events
    class WhenRequiredByAccessibilityServices(debugLogger: AccessibilityDebugLogger?)

    // Option to always synchronize the accessibility tree
    //
    // You can include an AccessibilityDebugLogger to log interactions and tree syncing events
    class Always(debugLogger: AccessibilityDebugLogger?)
}
```

### Implement the logging interface

You can implement the `AccessibilityDebugLogger` interface to write custom messages to an output of your choosing:

```kotlin
ComposeUIViewController(configure = {
    accessibilitySyncOptions = AccessibilitySyncOptions.WhenRequiredByAccessibilityServices(object: AccessibilityDebugLogger {
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

## Test accessibility with XCTest framework

You can use the semantic accessibility data in testing and other automation. Properties such as `testTag` correctly map
to native accessibility properties such as `accessibilityIdentifier`. This makes semantic data from Compose Multiplatform
available to Accessibility Services and the XCTest framework.

You can use automated accessibility audits in your UI tests.
Calling `performAccessibilityAudit()` for your `XCUIApplication` will audit the current view for accessibility
issues just as the Accessibility Inspector does.

```swift
func testAccessibilityTabView() throws {
	let app = XCUIApplication()
	app.launch()
	app.tabBars.buttons["MyLabel"].tap()
	
	try app.performAccessibilityAudit()
}
```

## What's next?

* Learn more in the [Apple accessibility](https://developer.apple.com/accessibility/) guide.
* Try out the project generated by [the Kotlin Multiplatform wizard](https://kmp.jetbrains.com/) in your usual 
  iOS accessibility workflow.