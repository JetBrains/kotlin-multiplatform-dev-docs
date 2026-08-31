[//]: # (title: Desktop-only API)

You can use Compose Multiplatform to create macOS, Linux, and Windows desktop applications. This page gives a short
overview of the desktop-specific components and events. Each section includes a link to a detailed tutorial.

## Components

<!-- * [Images and icons](#images-and-icons) -->
* [Windows and dialogs](compose-desktop-top-level-windows-management.md)
* [Context menus](compose-desktop-context-menus.md)
* [Tray and notifications](compose-desktop-tray.md)
* [Menu bar](compose-desktop-menu-bar.md)
* [Scrollbars](compose-desktop-scrollbars.md)
* [Tooltips](compose-desktop-tooltips.md)

<!-- ### Images and icons

You can use the `Image` composable and the `painterResource()` function to display images stored as resources in your
application:

```kotlin
import androidx.compose.foundation.Image
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.ui.Modifier
import androidx.compose.ui.res.painterResource
import androidx.compose.ui.window.singleWindowApplication

fun main() = singleWindowApplication {
    Image(
        painter = painterResource("sample.png"),
        contentDescription = "Sample",
        modifier = Modifier.fillMaxSize()
    )
}
```

`painterResource()` supports rasterized image formats, such as `.png`, `.jpg`, `.bmp`, `.webp`, and the Android XML vector
drawable format. You can also use images stored in the device memory, load images from the network,
or create them in your project using `Canvas()`.

With Compose Multiplatform, you can set the application window icon and the application tray icon as well.

* For more information on working with images using Compose Multiplatform in desktop projects, see
  the [Image and in-app icon manipulations](https://github.com/JetBrains/compose-multiplatform/tree/master/tutorials/Image_And_Icons_Manipulations)
  tutorial.
* For more information on using resources in common code in Compose Multiplatform projects, see [Images and resources](compose-multiplatform-resources.md). -->

## Events

* [Mouse events](compose-desktop-mouse-events.md)
* [Keyboard events](compose-desktop-keyboard.md)
* [Tabbing navigation](compose-desktop-tabbing.md)

## What's next

* Learn how to [create unit tests for your Compose Multiplatform desktop project](compose-desktop-ui-testing.md).
* Learn how to [create native distributions, installers, and packages for desktop platforms](compose-native-distribution.md).
* Set up [interoperability with Swing and migrate your Swing applications to Compose Multiplatform](compose-desktop-swing-interoperability.md).
* Learn about [accessibility support on different platforms](compose-desktop-accessibility.md).
