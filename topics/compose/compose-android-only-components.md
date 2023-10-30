[//]: # (title: Android-only components)

Compose Multiplatform builds on top of [Jetpack Compose](https://developer.android.com/jetpack/compose). Most of Compose
Multiplatform's functionality is available for all platforms. However, there are some APIs and libraries that you can
use only in the Android target. This is either because they are Android-specific, or because they haven't been ported to
other platforms yet. This page summarizes these parts of the Compose Multiplatform API.

> Sometimes, in [Jetpack Compose documentation](https://developer.android.com/jetpack/compose/documentation) or in
> articles created by the community, you may find an API that you can only use in the Android target.
> If you try to use it in `commonMain` code, your IDE will tell you that this API isn't available.
>
{type="note"}

## Android-only API

The Android-only API is Android-specific and isn't available on other platforms. This is because other platforms don't
need certain concepts that Android uses. The API usually uses classes from the `android.*` package or configures
Android-specific behavior. Here are some examples of parts of the Android-only API:

* [`android.context.Context`](https://developer.android.com/reference/android/content/Context) class
* [`LocalContext`](https://github.com/androidx/androidx/blob/41cb7d5c422180edd89efde4076f9dc724d3a313/compose/ui/ui/src/androidMain/kotlin/androidx/compose/ui/platform/AndroidCompositionLocals.android.kt)
  and [`LocalConfiguration`](https://github.com/androidx/androidx/blob/41cb7d5c422180edd89efde4076f9dc724d3a313/compose/ui/ui/src/androidMain/kotlin/androidx/compose/ui/platform/AndroidCompositionLocals.android.kt)
  variables
* [`android.graphics.BitmapFactory`](https://developer.android.com/reference/android/graphics/BitmapFactory)
  and [`android.graphics.Bitmap`](https://developer.android.com/reference/android/graphics/Bitmap) classes
* [`ImageBitmap.asAndroidBitmap()`](https://developer.android.com/reference/kotlin/androidx/compose/ui/graphics/ImageBitmap#(androidx.compose.ui.graphics.ImageBitmap).asAndroidBitmap())
  function
* [`android.app.Activity`](https://developer.android.com/reference/android/app/Activity) class
* [`android.app.Activity.setContent()`](https://github.com/androidx/androidx/blob/41cb7d5c422180edd89efde4076f9dc724d3a313/activity/activity-compose/src/main/java/androidx/activity/compose/ComponentActivity.kt)
  function
* [`ComposeView`](https://github.com/androidx/androidx/blob/41cb7d5c422180edd89efde4076f9dc724d3a313/compose/ui/ui/src/androidMain/kotlin/androidx/compose/ui/platform/ComposeView.android.kt)
  class
* [`LocalView`](https://github.com/androidx/androidx/blob/41cb7d5c422180edd89efde4076f9dc724d3a313/compose/ui/ui/src/androidMain/kotlin/androidx/compose/ui/platform/AndroidCompositionLocals.android.kt)
  variable
* [`Modifier.pointerInteropFilter()`](https://github.com/androidx/androidx/blob/41cb7d5c422180edd89efde4076f9dc724d3a313/compose/ui/ui/src/androidMain/kotlin/androidx/compose/ui/input/pointer/PointerInteropFilter.android.kt)
  function
* [Hilt](https://developer.android.com/jetpack/compose/libraries#hilt) dependency injection library

Usually, there isn't a strong reason to commonize parts of an API like this, so it's best to keep it in `androidMain`
only.

## API with Android classes in their signatures

There are parts of the API in Compose Multiplatform that use `android.*`, `androidx.*` (excluding `androidx.compose.*`)
in their signatures, but their behavior is applicable for other platforms as well:

* [Resource management](https://developer.android.com/jetpack/compose/resources): `stringResource`, `animatedVectorResource`, `Font`,
  and `*.R` class for resource management.
  <!-- For more information,see [Images and resources](compose-images-resources.md). -->
* [Navigation](https://developer.android.com/jetpack/compose/navigation).
  For more information, see [Navigation and routing](compose-navigation-routing.md).
* [`ViewModel`](https://developer.android.com/jetpack/compose/libraries#viewmodel) class.
* [Paging](https://developer.android.com/jetpack/compose/libraries#paging) library.
* [`ConstraintLayout`](https://developer.android.com/jetpack/compose/layouts/constraintlayout) layout.
* [Maps](https://developer.android.com/jetpack/compose/libraries#maps) library.
* [Preview](https://developer.android.com/jetpack/compose/libraries#maps) tool.
  ([Desktop support](https://plugins.jetbrains.com/plugin/16541-compose-multiplatform-ide-support)).
* [`WebView`](https://developer.android.com/reference/android/webkit/WebView) class.
* Other Jetpack Compose libraries that haven't been ported to Compose Multiplatform.

They may be ported to `commonMain` in the future, depending on complexity and demand.

APIs that are often used when developing applications, such as permissions, devices (Bluetooth, GPS, Camera),
and IO (network, files, databases), are out of the scope of Compose Multiplatform.
<!-- To find alternative solutions, see [Search for Multiplatform libraries](search-libs.md). -->

## API without Android classes in their signatures

Some parts of the API can be available only for the Android target, even if their signatures don't contain `android.*`
or `androidx.*` classes, and the API is applicable to other platforms. The reason behind this is usually that the
implementation uses many platform specifics and it takes time to write other implementations for other platforms.

Normally, parts of the API like this are ported to Compose Multiplatform after they are introduced in Jetpack Compose
for the Android target.

In Compose Multiplatform 1.5.10, the following parts of the API are **not** available in `commonMain`:

* [`Modifier.imeNestedScroll()`](https://github.com/androidx/androidx/blob/0e8dd4edd03f6e802303e5325ad11e89292c26c3/compose/foundation/foundation-layout/src/androidMain/kotlin/androidx/compose/foundation/layout/WindowInsetsConnection.android.kt)
  function
* [`Modifier.systemGestureExclusion()`](https://github.com/androidx/androidx/blob/0e8dd4edd03f6e802303e5325ad11e89292c26c3/compose/foundation/foundation/src/androidMain/kotlin/androidx/compose/foundation/SystemGestureExclusion.kt)
  function
* [`Modifier.magnifier()`](https://github.com/androidx/androidx/blob/41cb7d5c422180edd89efde4076f9dc724d3a313/compose/foundation/foundation/src/androidMain/kotlin/androidx/compose/foundation/Magnifier.kt)
  function
* [`LocalOverscrollConfiguration`](https://github.com/androidx/androidx/blob/41cb7d5c422180edd89efde4076f9dc724d3a313/compose/foundation/foundation/src/androidMain/kotlin/androidx/compose/foundation/OverscrollConfiguration.kt)
  variable
* [`AnimatedImageVector.animatedVectorResource` API](https://developer.android.com/jetpack/compose/resources#animated-vector-drawables)
* [material3-adaptive](https://github.com/androidx/androidx/tree/41cb7d5c422180edd89efde4076f9dc724d3a313/compose/material3/material3-adaptive)
  library
* [material3-window-size-class](https://github.com/androidx/androidx/tree/41cb7d5c422180edd89efde4076f9dc724d3a313/compose/material3/material3-window-size-class)
  library

## Request to port Android API

For each API that can be ported from Android, there
is [an open issue](https://github.com/JetBrains/compose-multiplatform/labels/commonization) in the Compose Multiplatform
repository. If you see that an API can be ported from Android and commonized, and there is no existing issue for
it, [create one](https://github.com/JetBrains/compose-multiplatform/issues/new?assignees=&labels=submitted%2C+enhancement&projects=&template=enhancement.md&title=).
