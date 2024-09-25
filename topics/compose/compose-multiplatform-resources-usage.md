[//]: # (title: Using multiplatform resources in your app)

<toc-settings structure-elements="chapter" depth="3"/>

When you've [set up the resources for your project](compose-multiplatform-resources-setup.md),
build the project to generate the special `Res` class which provides access to resources.
To manually generate the `Res` class and all the resource accessors, build or re-import the project in the IDE.

After that, you can use the generated class to access the configured multiplatform resources from your code or from external libraries.

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

* `\n` — for a new line
* `\t` — for a tab symbol
* `\uXXXX` — for a specific Unicode character

#### String templates

Currently, arguments have basic support in string resources:

```XML
<resources>
    <string name="str_template">Hello, %1$s! You have %2$d new messages.</string>
</resources>
```

There is no difference between `%...s` and `%...d ` when using string templates with arguments from composable code,
for example:

```kotlin
Text(stringResource(Res.string.str_template, "User_name", 100))
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
Text(stringArrayResource(Res.array.str_arr)[0])
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

To load a font as a `Font` type, use the `Font()` function:

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
val fontAwesome = FontFamily(Font(Res.font.font_awesome))
```

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

#### Convert byte arrays into images {label="EAP"}

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

### Generated maps for resources and string IDs {label="EAP"}

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

### Compose Multiplatform resources as Android assets {label="EAP"}

Starting with Compose Multiplatform %composeEapVersion%, all multiplatform resources are packed into Android assets.
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

### Using Java resources {label="EAP"}

While you can use Java resources with Compose Multiplatform, they don't benefit from extended features provided by
the framework: generated accessors, multimodule support, localization, and so on.
Consider transitioning fully to the multiplatform resource library to unlock that potential.

With Compose Multiplatform %composeEapVersion%, the resources API available in the `compose.ui` package is deprecated.
If you still need to work with Java resources, copy the following implementation to your project to ensure that your code
works after you upgrade to Compose Multiplatform 1.7.0:

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

Check out the official [demo project](https://github.com/JetBrains/compose-multiplatform/tree/master/components/resources/demo)
that shows how resources can be handled in a Compose Multiplatform project targeting iOS, Android, and desktop.
