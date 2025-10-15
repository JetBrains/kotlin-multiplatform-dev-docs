[//]: # (title: Using multiplatform resources in your app)

<show-structure depth="2"/>

When you've [set up the resources for your project](compose-multiplatform-resources-setup.md),
build the project to generate the special `Res` class which provides access to resources.
To regenerate the `Res` class and all the resource accessors, build the project again or re-import the project in the IDE.

After that, you can use the generated class to access the configured multiplatform resources from your code or from external libraries.

## Importing the generated class

To use the prepared resources, import the generated class, for example:

```kotlin
import project.composeapp.generated.resources.Res
import project.composeapp.generated.resources.example_image
```

Here:
* `project` is the name of your project
* `composeapp` is the module where you placed the resource directories
* `Res` is the default name for the generated class
* `example_image` is the name of an image file in the `composeResources/drawable` directory (`example_image.png`, for example).

## Customizing accessor class generation

You can customize the generated `Res` class to suit your needs using Gradle settings.

In the `compose.resources {}` block of the `build.gradle.kts` file, you can specify several settings that affect the way
the `Res` class is generated for your project.
An example configuration looks like this:

```kotlin
compose.resources {
    publicResClass = false
    packageOfResClass = "me.sample.library.resources"
    generateResClass = auto
}
```

* `publicResClass` set to `true` makes the generated `Res` class public. By default, the generated class is [internal](https://kotlinlang.org/docs/visibility-modifiers.html).
* `packageOfResClass` allows you to assign the generated `Res` class to a particular package (to access within the code,
    as well as for isolation in a final artifact). By default, Compose Multiplatform assigns the
    `{group name}.{module name}.generated.resources` package to the class.
* `generateResClass` set to `always` makes the project unconditionally generate the `Res` class. This may be useful
    when the resource library is only available transitively. By default, Compose Multiplatform uses the `auto` value
    to generate the `Res` class only if the current project has an explicit `implementation` or `api` dependency on the resource library.

## Resource usage

### Images

You can access drawable resources as simple images, rasterized images or XML vectors.
SVG images are supported on all platforms **except** Android.

* To access drawable resources as `Painter` images, use the `painterResource()` function:

  ```kotlin
  @Composable
  fun painterResource(resource: DrawableResource): Painter {...}
  ```

  The `painterResource()` function takes a resource path and returns a `Painter` value. The function works synchronously
  on all targets except for web. For the web target, it returns an empty `Painter` for the first recomposition that is
  replaced with the loaded image in subsequent recompositions.

  * `painterResource()` loads either a `BitmapPainter` for rasterized image formats, such as `.png`, `.jpg`, `.bmp`, `.webp`,
    or a `VectorPainter` for the Android XML vector drawable format.
  * XML vector drawables have the same format as [Android](https://developer.android.com/reference/android/graphics/drawable/VectorDrawable),
    except that they don't support external references to Android resources.

* To access drawable resources as an `ImageBitmap` rasterized image, use the `imageResource()` function:

  ```kotlin
  @Composable
  fun imageResource(resource: DrawableResource): ImageBitmap {...}
  ```

* To access drawable resources as an `ImageVector` XML vector, use the `vectorResource()` function:

  ```kotlin
  @Composable
  fun vectorResource(resource: DrawableResource): ImageVector {...}
  ```

Here's an example of how you can access images in your Compose Multiplatform code:

```kotlin
Image(
    painter = painterResource(Res.drawable.my_icon),
    contentDescription = null
)
```

### Strings

Store all string resources in XML files in `composeResources/values` directories.
A static accessor is generated for each item in each file.

For more information on how to localize strings for different locales, refer to the 
[guide on localizing strings](compose-localize-strings.md).

#### Simple strings

To store a simple string, add a `<string>` element to your XML:

```XML
<resources>
    <string name="app_name">My awesome app</string>
    <string name="title">Some title</string>
</resources>
```

To get string resources as a `String`, use the following code:

<tabs>
<tab title= "From composable code">

```kotlin
@Composable
fun stringResource(resource: StringResource): String {...}

@Composable
fun stringResource(resource: StringResource, vararg formatArgs: Any): String {...}
```

For example:

```kotlin
Text(stringResource(Res.string.app_name))
```

</tab>
<tab title= "From non-composable code">

```kotlin
suspend fun getString(resource: StringResource): String

suspend fun getString(resource: StringResource, vararg formatArgs: Any): String
```

For example:

```kotlin
coroutineScope.launch {
    val appName = getString(Res.string.app_name)
}
```

</tab>
</tabs>

You can use special symbols in string resources:

* `\n` – for a new line
* `\t` – for a tab symbol
* `\uXXXX` – for a specific Unicode character

You don't need to escape special XML characters like "@" or "?"
as you do [for Android strings](https://developer.android.com/guide/topics/resources/string-resource#escaping_quotes). 

#### String templates

Currently, arguments have basic support for string resources.
When creating a template, use the `%<number>` format to place arguments within the string and include a `$d` or `$s` suffix
to indicate that it is a variable placeholder and not simple text.
For example:

```XML
<resources>
    <string name="str_template">Hello, %2$s! You have %1$d new messages.</string>
</resources>
```

After creating and importing the string template resource, you can refer to it while
passing the arguments for placeholders in the correct order:

```kotlin
Text(stringResource(Res.string.str_template, 100, "User_name"))
```

There is no difference between the `$s` and `$d` suffixes, and no others are supported.
You can put the `%1$s` placeholder in the resource string and use it to display a fractional number, for example:

```kotlin
Text(stringResource(Res.string.str_template, "User_name", 100.1f))
```

#### String arrays

You can group related strings into an array and automatically access them as a `List<String>` object:

```XML
<resources>
    <string name="app_name">My awesome app</string>
    <string name="title">Some title</string>
    <string-array name="str_arr">
        <item>item \u2605</item>
        <item>item \u2318</item>
        <item>item \u00BD</item>
    </string-array>
</resources>
```

To get the corresponding list, use the following code:

<tabs>
<tab title= "From composable code">

```kotlin
@Composable
fun stringArrayResource(resource: StringArrayResource): List<String> {...}
```

For example:

```kotlin
val arr = stringArrayResource(Res.array.str_arr)
if (arr.isNotEmpty()) Text(arr[0])
```

</tab>
<tab title= "From non-composable code">

```kotlin
suspend fun getStringArray(resource: StringArrayResource): List<String>
```

For example:

```kotlin
coroutineScope.launch {
    val appName = getStringArray(Res.array.str_arr)
}
```

</tab>
</tabs>

#### Plurals

When your UI displays quantities of something, you might want to support grammatical agreement for different numbers
of the same thing (one _book_, many _books_, and so on) without creating programmatically unrelated strings.

The concept and base implementation in Compose Multiplatform are the same as for quantity strings
on Android.
See [the Android documentation](https://developer.android.com/guide/topics/resources/string-resource#Plurals) for more
about best practices and nuances of using plurals in your project.

* The supported variants are `zero`, `one`, `two`, `few`, `many`, and `other`. Note that not all variants are even 
  considered for every language: for example, `zero` is ignored for English because it is the same as any other plural
  except 1. Rely on a language specialist to know what distinctions the language actually insists upon.
* It's often possible to avoid quantity strings by using quantity-neutral formulations such as "Books: 1".
  If this doesn't worsen the user experience, 

To define a plural, add a `<plurals>` element to any `.xml` file in your `composeResources/values` directory.
A `plurals` collection is a simple resource referenced using the name attribute (not the name of the XML file).
As such, you can combine `plurals` resources with other simple resources in one XML file under one `<resources>` element:

```xml
<resources>
    <string name="app_name">My awesome app</string>
    <string name="title">Some title</string>
    <plurals name="new_message">
        <item quantity="one">%1$d new message</item>
        <item quantity="other">%1$d new messages</item>
    </plurals>
</resources>
```

To access a plural as a `String`, use the following code:

<tabs>
<tab title= "From composable code">

```kotlin
@Composable
fun pluralStringResource(resource: PluralStringResource, quantity: Int): String {...}

@Composable
fun pluralStringResource(resource: PluralStringResource, quantity: Int, vararg formatArgs: Any): String {...}
```

For example:

```kotlin
Text(pluralStringResource(Res.plurals.new_message, 1, 1))
```

</tab>
<tab title= "From non-composable code">

```kotlin
suspend fun getPluralString(resource: PluralStringResource, quantity: Int): String

suspend fun getPluralString(resource: PluralStringResource, quantity: Int, vararg formatArgs: Any): String
```

For example:

```kotlin
coroutineScope.launch {
    val appName = getPluralString(Res.plurals.new_message, 1, 1)
}
```

</tab>
</tabs>

### Fonts

Store custom fonts in the `composeResources/font` directory as `*.ttf` or `*.otf` files.

To load a font as a `Font` type, use the `Font()` composable function:

```kotlin
@Composable
fun Font(
    resource: FontResource,
    weight: FontWeight = FontWeight.Normal,
    style: FontStyle = FontStyle.Normal
): Font
```

For example:

```kotlin
@Composable
private fun InterTypography(): Typography {
    val interFont = FontFamily(
        Font(Res.font.Inter_24pt_Regular, FontWeight.Normal),
        Font(Res.font.Inter_24pt_SemiBold, FontWeight.Bold),
    )

    return with(MaterialTheme.typography) {
        copy(
            displayLarge = displayLarge.copy(fontFamily = interFont, fontWeight = FontWeight.Bold),
            displayMedium = displayMedium.copy(fontFamily = interFont, fontWeight = FontWeight.Bold),
            displaySmall = displaySmall.copy(fontFamily = interFont, fontWeight = FontWeight.Bold),
            headlineLarge = headlineLarge.copy(fontFamily = interFont, fontWeight = FontWeight.Bold),
            headlineMedium = headlineMedium.copy(fontFamily = interFont, fontWeight = FontWeight.Bold),
            headlineSmall = headlineSmall.copy(fontFamily = interFont, fontWeight = FontWeight.Bold),
            titleLarge = titleLarge.copy(fontFamily = interFont, fontWeight = FontWeight.Bold),
            titleMedium = titleMedium.copy(fontFamily = interFont, fontWeight = FontWeight.Bold),
            titleSmall = titleSmall.copy(fontFamily = interFont, fontWeight = FontWeight.Bold),
            labelLarge = labelLarge.copy(fontFamily = interFont, fontWeight = FontWeight.Normal),
            labelMedium = labelMedium.copy(fontFamily = interFont, fontWeight = FontWeight.Normal),
            labelSmall = labelSmall.copy(fontFamily = interFont, fontWeight = FontWeight.Normal),
            bodyLarge = bodyLarge.copy(fontFamily = interFont, fontWeight = FontWeight.Normal),
            bodyMedium = bodyMedium.copy(fontFamily = interFont, fontWeight = FontWeight.Normal),
            bodySmall = bodySmall.copy(fontFamily = interFont, fontWeight = FontWeight.Normal),
        )
    }
}
```

{initial-collapse-state="collapsed" collapsible="true" collapsed-title="@Composable private fun InterTypography(): Typography { val interFont = FontFamily("}

> When `Font` is a composable, make sure its dependent components, such as `TextStyle` and `Typography`, are also composable.
>
{style="note"}

To support special characters like emojis or Arabic script in web targets, you need to add the corresponding fonts
to resources and [preload fallback fonts](#preload-resources-using-the-compose-multiplatform-preload-api).

### Raw files

To load any raw file as a byte array, use the `Res.readBytes(path)` function:

```kotlin
suspend fun readBytes(path: String): ByteArray
```

You can place raw files in the `composeResources/files` directory and create any hierarchy inside it.

For example, to access raw files, use the following code:

<tabs>
<tab title= "From composable code">

```kotlin
var bytes by remember {
    mutableStateOf(ByteArray(0))
}
LaunchedEffect(Unit) {
    bytes = Res.readBytes("files/myDir/someFile.bin")
}
Text(bytes.decodeToString())
```

</tab>
<tab title= "From non-composable code">

```kotlin
coroutineScope.launch {
    val bytes = Res.readBytes("files/myDir/someFile.bin")
}
```

</tab>
</tabs>

#### Convert byte arrays into images

If the file you are reading is a bitmap (JPEG, PNG, BMP, WEBP) or an XML vector image, you can use the following functions
to convert them into `ImageBitmap` or `ImageVector` objects suitable for the `Image()` composable.

Access the raw files as shown in the [Raw files](#raw-files) section, then pass the result to a composable:

```kotlin
// bytes = Res.readBytes("files/example.png")
Image(bytes.decodeToImageBitmap(), null)

// bytes = Res.readBytes("files/example.xml")
Image(bytes.decodeToImageVector(LocalDensity.current), null)
```

On every platform except Android, you can also turn an SVG file into a `Painter` object:

```kotlin
// bytes = Res.readBytes("files/example.svg")
Image(bytes.decodeToSvgPainter(LocalDensity.current), null)
```

### Generated maps for resources and string IDs

For ease of access, Compose Multiplatform also maps resources with string IDs. You can access them by using
the filename as the key:

```kotlin
val Res.allDrawableResources: Map<String, DrawableResource>
val Res.allStringResources: Map<String, StringResource>
val Res.allStringArrayResources: Map<String, StringArrayResource>
val Res.allPluralStringResources: Map<String, PluralStringResource>
val Res.allFontResources: Map<String, FontResource>
```

An example of passing a mapped resource to a composable:

```kotlin
Image(painterResource(Res.allDrawableResources["compose_multiplatform"]!!), null)
```

### Compose Multiplatform resources as Android assets

Starting with Compose Multiplatform 1.7.0, all multiplatform resources are packed into Android assets.
This enables Android Studio to generate previews for Compose Multiplatform composables in Android source sets.

> Android Studio previews are available only for composables in an Android source set.
> They also require one of the latest versions of AGP: 8.5.2, 8.6.0-rc01, or 8.7.0-alpha04.
>
{style="warning"}

Using Multiplatform resources as Android assets also makes possible direct access from WebViews and media player components
on Android, since resources can be reached by a simple path, for example `Res.getUri("files/index.html")`.

An example of an Android composable displaying a resource HTML page with a link to a resource image:

```kotlin
// androidMain/kotlin/com/example/webview/App.kt
@OptIn(ExperimentalResourceApi::class)
@Composable
@Preview
fun App() {
    MaterialTheme {
        val uri = Res.getUri("files/webview/index.html")

        // Adding a WebView inside AndroidView with layout as full screen.
        AndroidView(factory = {
            WebView(it).apply {
                layoutParams = ViewGroup.LayoutParams(
                    ViewGroup.LayoutParams.MATCH_PARENT,
                    ViewGroup.LayoutParams.MATCH_PARENT
                )
            }
        }, update = {
            it.loadUrl(uri)
        })
    }
}
```
{initial-collapse-state="collapsed" collapsible="true"  collapsed-title="AndroidView(factory = { WebView(it).apply"}

The example works with this simple HTML file:

```html
<html>
<header>
    <title>
        Cat Resource
    </title>
</header>
<body>
    <img src="cat.jpg">
</body>
</html>
```
{initial-collapse-state="collapsed" collapsible="true"  collapsed-title="<title>Cat Resource</title>"}

Both resource files in this example are located in the `commonMain` source set:

![File structure of the composeResources directory](compose-resources-android-webview.png){width="230"}

## Preloading of resources for web targets

The web resources like fonts and images are loaded asynchronously using the `fetch` API. During the initial load or with 
slower network connections, resource fetching can cause visual glitches, such as [FOUT](https://fonts.google.com/knowledge/glossary/fout) 
or displaying placeholders instead of images.

A typical example of this issue is when a `Text()` component contains text in a custom font, but the font with the 
necessary glyphs is still loading. In this case, users may temporarily see the text in default font or even empty boxes and 
question marks instead of characters. Similarly, for images or drawables, users may observe a placeholder like a blank or 
black box until the resource is fully loaded.

To prevent visual glitches, you can use built-in browser features for preloading of resources, 
the Compose Multiplatform preload API, or a combination of both.

### Preload resources using browser features

In modern browsers, you can preload resources using the `<link>` tag with the [`rel="preload"` attribute](https://developer.mozilla.org/en-US/docs/Web/HTML/Attributes/rel/preload).
This attribute instructs the browser to prioritize downloading and caching resources like fonts and images 
before the application starts, ensuring that these resources are available early. 

For example, to enable in-browser preloading of a font:

1. Build your application's web distribution:

```console
   ./gradlew :composeApp:wasmJsBrowserDistribution
```

2. Find the required resource in the generated `dist` directory and save the path.
3. Open the `wasmJsMain/resources/index.html` file and add a `<link>` tag inside the `<head>` element. 
4. Set the `href` attribute to the resource path:

```html
<link rel="preload" href="./composeResources/username.composeapp.generated.resources/font/FiraMono-Regular.ttf" as="fetch" type="font/ttf" crossorigin/>
```

### Preload resources using the Compose Multiplatform preload API
<primary-label ref="Experimental"/>

Even if you preloaded the resources in the browser, they are cached as raw bytes that still need to be converted into a 
format suitable for rendering, such as `FontResource` and `DrawableResource`. When the application requests the resource 
for the first time, the conversion is done asynchronously, which may again result in flickering. To further optimize the experience, 
Compose Multiplatform resources have their own internal cache for higher-level representations of the resources, that can also be preloaded.

Compose Multiplatform 1.8.0 introduced an experimental API for preloading font and image resources on 
web targets: `preloadFont()`, `preloadImageBitmap()`, and `preloadImageVector()`.

Additionally, you can set fallback fonts different from the default bundled option if you require special characters like emojis.
To specify a fallback font, use the `FontFamily.Resolver.preload()` method.

The following example demonstrates how to use preloading and a fallback font:

```kotlin
import androidx.compose.foundation.background
import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.material3.CircularProgressIndicator
import androidx.compose.runtime.*
import androidx.compose.ui.*
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.platform.LocalFontFamilyResolver
import androidx.compose.ui.text.font.FontFamily
import androidx.compose.ui.text.font.FontStyle
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.window.CanvasBasedWindow
import components.resources.demo.shared.generated.resources.*
import components.resources.demo.shared.generated.resources.NotoColorEmoji
import components.resources.demo.shared.generated.resources.Res
import components.resources.demo.shared.generated.resources.Workbench_Regular
import components.resources.demo.shared.generated.resources.font_awesome
import org.jetbrains.compose.resources.ExperimentalResourceApi
import org.jetbrains.compose.resources.configureWebResources
import org.jetbrains.compose.resources.demo.shared.UseResources
import org.jetbrains.compose.resources.preloadFont

@OptIn(ExperimentalComposeUiApi::class, ExperimentalResourceApi::class, InternalComposeUiApi::class)
fun main() {
    configureWebResources {
        // Overrides the resource location
        resourcePathMapping { path -> "./$path" }
    }
    CanvasBasedWindow("Resources + K/Wasm") {
        val font1 by preloadFont(Res.font.Workbench_Regular)
        val font2 by preloadFont(Res.font.font_awesome, FontWeight.Normal, FontStyle.Normal)
        val emojiFont = preloadFont(Res.font.NotoColorEmoji).value
        var fontsFallbackInitialized by remember { mutableStateOf(false) }

        // Uses the preloaded resource for the app's content
        UseResources()

        if (font1 != null && font2 != null && emojiFont != null && fontsFallbackInitialized) {
            println("Fonts are ready")
        } else {
            // Displays the progress indicator to address a FOUT or the app being temporarily non-functional during loading
            Box(modifier = Modifier.fillMaxSize().background(Color.White.copy(alpha = 0.8f)).clickable {  }) {
                CircularProgressIndicator(modifier = Modifier.align(Alignment.Center))
            }
            println("Fonts are not ready yet")
        }

        val fontFamilyResolver = LocalFontFamilyResolver.current
        LaunchedEffect(fontFamilyResolver, emojiFont) {
            if (emojiFont != null) {
                // Preloads a fallback font with emojis to render missing glyphs that are not supported by the bundled font
                fontFamilyResolver.preload(FontFamily(listOf(emojiFont)))
                fontsFallbackInitialized = true
            }
        }
    }
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="fontFamilyResolver.preload(FontFamily(listOf(emojiFont)))"}

## Interaction with other libraries and resources

### Accessing multiplatform resources from external libraries

If you want to process multiplatform resources using other libraries included in your project, you can pass platform-specific
file paths to these other APIs.
To get a platform-specific path, call the `Res.getUri()` function with the project path to the resource:

```kotlin
val uri = Res.getUri("files/my_video.mp4")
```

Now that the `uri` variable contains the absolute path to the file, any external library can use that path to access
the file in a manner that suits it.

For Android-specific uses, multiplatform resources are also [packed as Android assets](#compose-multiplatform-resources-as-android-assets).

### Remote files

In the context of the resource library, only files that are part of the application are considered resources.

You can load remote files from the internet using their URL using specialized libraries:

* [Compose ImageLoader](https://github.com/qdsfdhvh/compose-imageloader)
* [Kamel](https://github.com/Kamel-Media/Kamel)
* [Ktor client](https://ktor.io/)

### Using Java resources

While you can use Java resources with Compose Multiplatform, they don't benefit from extended features provided by
the framework: generated accessors, multimodule support, localization, and so on.
Consider transitioning fully to the multiplatform resource library to unlock that potential.

With Compose Multiplatform 1.7.0, the resources API available in the `compose.ui` package is deprecated.
If you still need to work with Java resources, copy the following implementation to your project to ensure that your code
works after you upgrade to Compose Multiplatform 1.7.0 or above:

```kotlin
@Composable
internal fun painterResource(
    resourcePath: String
): Painter = when (resourcePath.substringAfterLast(".")) {
    "svg" -> rememberSvgResource(resourcePath)
    "xml" -> rememberVectorXmlResource(resourcePath)
    else -> rememberBitmapResource(resourcePath)
}

@Composable
internal fun rememberBitmapResource(path: String): Painter {
    return remember(path) { BitmapPainter(readResourceBytes(path).decodeToImageBitmap()) }
}

@Composable
internal fun rememberVectorXmlResource(path: String): Painter {
    val density = LocalDensity.current
    val imageVector = remember(density, path) { readResourceBytes(path).decodeToImageVector(density) }
    return rememberVectorPainter(imageVector)
}

@Composable
internal fun rememberSvgResource(path: String): Painter {
    val density = LocalDensity.current
    return remember(density, path) { readResourceBytes(path).decodeToSvgPainter(density) }
}

private object ResourceLoader
private fun readResourceBytes(resourcePath: String) =
    ResourceLoader.javaClass.classLoader.getResourceAsStream(resourcePath).readAllBytes()
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="internal fun painterResource(resourcePath: String): Painter"}

## What's next?

* Check out the official [demo project](https://github.com/JetBrains/compose-multiplatform/tree/master/components/resources/demo)
that shows how resources can be handled in a Compose Multiplatform project targeting iOS, Android, and desktop.
* Learn how to manage the application's [resource environment](compose-resource-environment.md) like in-app theme and language.
