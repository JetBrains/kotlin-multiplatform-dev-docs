[//]: # (title: What's new with KMP in Fleet 1.31)

Here are the highlights of changes related to Kotlin Multiplatform support in JetBrains Fleet 1.31:

* [Compose Multiplatform @Preview annotation support](#support-for-compose-multiplatform-preview-annotation)
* [Support for Compose Multiplatform resources API](#support-for-compose-multiplatform-resources-api)
* [Logcat support](#logcat-support)
* [Improvements in Swift support](#various-improvements-in-swift-support)

## Support for Compose Multiplatform @Preview annotation

JetBrains Fleet now allows you to build previews for composable functions annotated with `@Preview`.
Currently, Fleet does not rebuild previews automatically, so click the icon again to see the updated version:

![Compose Preview tool window in JetBrains Fleet](fleet-compose-preview-light.png){width="700"}

> The previews currently work for common code as well as for desktop source sets.
> Android native UI previews are not supported yet.
> 
{type="note"}

For more about the feature, see [Getting started with Kotlin Multiplatform](https://www.jetbrains.com/help/fleet/getting-started-with-kotlin-multiplatform.html#b65b852e_76)
in the Fleet documentation.

## Support for Compose Multiplatform resources API

The latest version of the [images and resources access API](compose-images-resources.md), revamped in Compose Multiplatform 1.6.0,
is supported in Fleet.

Starting with this version, JetBrains Fleet:

* Autocompletes resources calls
* Allows navigation to resources referenced in code
* Clearly reports generation errors and their possible source
* Monitors file system changes to regenerate the accessor class promptly

## Logcat support

JetBrains Fleet now allows you to see device logs for Android run configurations. In the run configuration 
output, click the line "Connected to process ... on device ...". This will open the Logcat tool window.

In the **Logcat** tab, you can see continuous device logs, filter text, and set the minimum log level:

![Logcat tool window in JetBrains Fleet](fleet-logcat-window-light.png){width="700"}

## Various improvements in Swift support

A number of small improvements in formatting, highlighting, and completion for Swift 5.9 features.