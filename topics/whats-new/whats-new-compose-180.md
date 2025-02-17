[//]: # (title: What's new in Compose Multiplatform %composeEapVersion%)

Here are the highlights for this EAP feature release:

* [Variable fonts](#variable-fonts)
* [Preloading resources on web](#preloading-resources)

See the full list of changes for this release [on GitHub](https://github.com/JetBrains/compose-multiplatform/blob/v1.8.0-beta01/CHANGELOG.md). 

## Dependencies

[//]: # (TODO other sections)

## Across platforms

### Variable fonts

Compose Multiplatform now supports variable fonts on all platforms. With variable fonts, you can only keep one font file 
with all style preferences such as weight, width, slant, italic, custom axes, visual weight with typographic color, 
and style adaptable to specific text sizes.

For details, see the [Jetpack Compose documentation](https://developer.android.com/develop/ui/compose/text/fonts#variable-fonts).

## Desktop

### Support for Windows for ARM64

Compose Multiplatform introduces support for Windows for ARM64, allowing you to build and run applications natively on 
ARM-based Windows devices.

## Web

### Preloading resources
<secondary-label ref="Experimental"/>

Compose Multiplatform introduces a new experimental API for preloading fonts and images for web targets. Preloading helps 
prevent visual issues such as Flash of Unstyled Text (FOUT) or flickering of images and icons.

The following functions are now available for loading and caching resources:

* `preloadFont()` preloads fonts
* `preloadImageBitmap()` preloads bitmap images
* `preloadImageVector()` preloads vector images

See the [documentation](compose-multiplatform-resources-usage.md#preload-resources-using-the-compose-multiplatform-preload-api) for details. 