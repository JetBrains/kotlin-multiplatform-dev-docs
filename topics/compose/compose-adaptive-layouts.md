# Adaptive layouts

To provide a consistent user experience on all types of devices, adapt your app's UI to different display sizes, orientations, 
and input modes.

## Designing adaptive layouts

Follow these key guidelines when designing adaptive layouts:

* Prefer [canonical layouts](https://developer.android.com/develop/ui/compose/layouts/adaptive/canonical-layouts) patterns 
  like list-detail, feed, and supporting pane.
* Maintain consistency by reusing shared styles for padding, typography, and other design elements. Keep navigation patterns 
consistent across devices while following platform-specific guidelines.
* Break complex layouts into reusable composables for flexibility and modularity.
* Adjust for screen density and orientation.

## Using window size classes

Window size classes are a set of predefined thresholds, also referred to as breakpoints, that categorize different screen 
sizes to help you design, develop, and test adaptive layouts.

The window size classes categorize the display area available to your app into three categories for both width and height:
compact, medium, and expanded. As you make layout changes, test the layout behavior across all window sizes,
especially at the different breakpoint widths.

With `WindowSizeClass`, you can change your app layout as the display space available to your app changes. For example,
when the device orientation changes, or the app window is resized in multi‑window mode.

<!--- waiting for a page about @Preview and hot reload
## Previewing layouts

We have three different @Preview:

* Android-specific, for `androidMain`, from Android Studio.
* Separate desktop annotation plugin with our own implementation (only for desktop source set) + uiTooling plugin.
* Common annotation, also supported in Android Studio, works for Android only but from common code.
-->

## What's next

Learn more about adaptive layouts in the [Jetpack Compose documentation](https://developer.android.com/develop/ui/compose/layouts/adaptive).