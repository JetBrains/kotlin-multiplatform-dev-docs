[//]: # (title: What's new with KMP in Fleet 1.31)

Here are the highlights of changes related to Kotlin Multiplatform support in JetBrains Fleet 1.31:

* [Compose Multiplatform @Preview annotation support](#compose-multiplatform-preview-annotation-support)
* [Support for Compose Multiplatform resources API](#support-for-compose-multiplatform-resources-api)
* [Logcat support](#logcat-support)
* [Improvements in Swift support](#various-improvements-in-swift-support)

## Compose Multiplatform @Preview annotation support

JetBrains Fleet now allows you to build previews for composable functions annotated with `@Preview`.
For now, Fleet only supports static (noninteractive) previews:

![Compose Preview tool window in JetBrains Fleet](fleet-compose-preview.png)

> The previews currently work for common code as well as for desktop source sets.
> Android native UI previews are not supported yet.
> 
{type="note"}

For more about the feature, see [Getting started with Kotlin Multiplatform](https://www.jetbrains.com/help/fleet/getting-started-with-kotlin-multiplatform.html#b65b852e_76)
in Fleet documentation.

## Support for Compose Multiplatform resources API

The latest version of the [images and resources access API](compose-images-resources.md), revamped in Compose Multiplatform 1.6.0,
is supported in Fleet.

The IDE:

* autocompletes resources calls,
* allows navigation to resources referenced in code,
* clearly reports generation errors and their possible source,
* monitors file system changes to promptly regenerate the accessor class.

## Logcat support

JetBrains Fleet now allows you to see device logs for Android run configurations. In the run configuration 
output, click the line "Connected to process ... on device ...". This will open the Logcat tool window.

In the **Logcat** tab, you can see continuous device logs, filter text and set the minimum log level:

![Logcat tool window in JetBrains Fleet](fleet-logcat-window.png){width="700"}

## Various improvements in Swift support

A number of small improvements in formatting, highlighting, and completion for Swift 5.9 features.