[//]: # (title: Using multiplatform resources in your app)
<web-summary>Learn how to access images, strings, fonts, and raw files in Compose Multiplatform, and how to pass
resources to other libraries.</web-summary>

<show-structure depth="2"/>

When you've [set up the resources for your project](compose-multiplatform-resources-setup.md),
build the project to generate the special `Res` class which provides access to resources.
To regenerate the `Res` class and all the resource accessors, build the project again or re-import the project in the IDE.

After that, you can use the generated class to access the configured multiplatform resources from your code or from external libraries.

Read on for details about the following topics:

* [Importing the generated `Res` class and accessors](#importing-the-generated-class).
* [Customizing the accessor class generation](#customizing-accessor-class-generation): how to make it public, rename it, assign it to a package, or control when it is generated.
* Working with specific resource types: 
  * [Drawable resources](#images), such as simple images, rasterized images, or XML vectors.
  * [Vector Android XML icons](#icons) from the Material Symbols library. 
  * [Strings](#strings), including simple strings, templates, arrays, and plurals.
  * [Storing and loading custom fonts](#fonts).
  * [Raw files](#raw-files) and converting byte arrays into images. 
* [Accessing resources mapped with string IDs](#generated-maps-for-resources-and-string-ids).
* [Using multiplatform resources as Android assets](#compose-multiplatform-resources-as-android-assets).
* Handling web-specific resources:
  * [Preloading of resources](compose-web-resources.md#preloading-of-resources-for-web-targets) using browser features and the preload API.
  * [Automatic font fallback](compose-web-resources.md#automatic-font-fallback) for missing characters.
  * [Caching web resources](compose-web-resources.md#caching-web-resources).
* Working with external resources: 
  [from external libraries](#accessing-multiplatform-resources-from-external-libraries),
  [remote files](#remote-files), and [Java resources](#using-java-resources).

## Importing the generated class

To use the prepared resources, import the generated class, for example:

```kotlin
import project.shared.generated.resources.Res
import project.shared.generated.resources.example_image
```

Here:
* `project` is the name of your project
* `shared` is the module where you placed the resource directories
* `Res` is the default name for the generated class
* `example_image` is the name of an image file in the `composeResources/drawable` directory (`example_image.png`, for example).

Besides `Res` and the resource accessors, this package also contains the [generated maps](#generated-maps-for-resources-and-string-ids)
for accessing resources by string IDs, such as `allDrawableResources`. Import them in the same way.

## Customizing accessor class generation

You can customize the generated `Res` class to suit your needs using Gradle settings.

In the `compose.resources {}` block of the `build.gradle.kts` file, you can specify several settings that affect the way
the `Res` class is generated for your project:

```kotlin
compose.resources {
    publicResClass = true
    packageOfResClass = "me.sample.library.resources"
    nameOfResClass = "MyRes"
    generateResClass = always
}
```

* `publicResClass` – if `true`, makes the generated `Res` class public. By default, the generated class is [internal](https://kotlinlang.org/docs/visibility-modifiers.html).
* `packageOfResClass` – assigns the generated `Res` class to a particular package (to access within the code,
  as well as for isolation in a final artifact).
  By default, Compose Multiplatform assigns the `{group name}.{module name}.generated.resources` package to the class.
* `nameOfResClass` – changes the name of the generated class. By default, the class is named `Res`.
* `generateResClass` – controls when the `Res` class is generated:
  * `auto` – the default value; generates the `Res` class only if the current project has an explicit `implementation` or `api` dependency on the resource library.
  * `always` – unconditionally generates the `Res` class. This may be useful when the resource library is only available transitively.
  * `never` – disables the generation of the `Res` class.

## Resource usage

### Images

You can access drawable resources as simple images, rasterized images or XML vectors.
SVG images are supported on all platforms **except** Android.

> If an SVG image is placed in `commonMain`, the project builds for Android, but the app throws an
> `IllegalStateException` with the `Android platform doesn't support SVG format` message at runtime.
> Place SVG images in the `composeResources/drawable` directory of a source set that is not compiled for Android,
> for example, a custom `nonAndroidMain` source set as in the [demo project](https://github.com/JetBrains/compose-multiplatform/tree/master/components/resources/demo).
>
{style="warning"}

* To access drawable resources as `Painter` images, use the `painterResource()` function:

  ```kotlin
  @Composable
  fun painterResource(resource: DrawableResource): Painter {...}
  ```
  
  For example:

  ```kotlin
  Image(
      painter = painterResource(Res.drawable.my_image),
      contentDescription = null
  )
  ```

  * `painterResource()` works synchronously on all targets except for web. For the web target, it returns an empty 
    `Painter` for the first composition that is replaced with the loaded image in subsequent recompositions.

    Starting with Compose Multiplatform 1.13.0, you can call the experimental `awaitPendingResourceStateLoads()` suspend function
    from the `org.jetbrains.compose.resources` package to wait until all pending asynchronous resource loads are complete,
    for example, in screenshot tests. Failed or canceled loads are also considered complete.

  * `painterResource()` loads either a `BitmapPainter` for rasterized image formats, such as `.png`, `.jpg`, `.bmp`, `.webp`,
    or a `VectorPainter` for the Android XML vector drawable format.

    XML vector drawables have the same format as [Android](https://developer.android.com/reference/android/graphics/drawable/VectorDrawable),
    except that they don't support external references to Android resources.

* To access drawable resources as an `ImageBitmap` rasterized image, use the `imageResource()` function:

  ```kotlin
  @Composable
  fun imageResource(resource: DrawableResource): ImageBitmap {...}
  ```

  For example:

  ```kotlin
  Image(
      bitmap = imageResource(Res.drawable.my_image),
      contentDescription = null
  )
  ```
  
* To access drawable resources as an `ImageVector` XML vector, use the `vectorResource()` function:

  ```kotlin
  @Composable
  fun vectorResource(resource: DrawableResource): ImageVector {...}
  ```

  For example:

  ```kotlin
  Image(
      imageVector = vectorResource(Res.drawable.ic_sample_icon),
      contentDescription = null
  )
  ```

#### Drawing images on a canvas

To draw a resource image yourself instead of showing it with the `Image()` composable, load it as an `ImageBitmap` or
an `ImageVector` and use it in a `DrawScope`:

* To draw an `ImageBitmap`, use `DrawScope.drawImage()`.
* To draw an `ImageVector`, create a `Painter` with `rememberVectorPainter()` and use `Painter.draw()`.

Specify offsets and sizes in a `DrawScope` in pixels:

```kotlin
import androidx.compose.foundation.Canvas
import androidx.compose.ui.geometry.Offset
import androidx.compose.ui.geometry.Size
import androidx.compose.ui.graphics.drawscope.translate
import androidx.compose.ui.graphics.vector.rememberVectorPainter

val myImageRaster = imageResource(Res.drawable.my_image_raster)
val myImageVectorPainter = rememberVectorPainter(vectorResource(Res.drawable.my_image_vector))

Canvas(modifier = Modifier.fillMaxSize()) {
    // Inside the lambda, `this` is an instance of `DrawScope`,
    // which provides `drawImage()` and `translate()`
    drawImage(image = myImageRaster, topLeft = Offset(20f, 20f))
    translate(left = 20f, top = myImageRaster.height + 40f) {
        // The `Painter.draw()` function uses both the 
        // `DrawScope` and `Painter` as receivers
        with(myImageVectorPainter) {
            draw(Size(200f, 200f))
        }
    }
}
```

### Icons

You can use the vector Android XML icons from the Material Symbols library:

1. Open the [Google Fonts Icons](https://fonts.google.com/icons) gallery, choose an icon, go to the Android tab, and click **Download**.

2. Add the downloaded XML icon file to the `drawable` directory of your multiplatform resources.

3. Open the XML icon file and set `android:fillColor` to `#000000`. Compose Multiplatform supports only hexadecimal 
   color values; references to Android resources, such as `@android:color/white`, make the app crash with 
   an `java.lang.IllegalArgumentException: Invalid color value` error.

   You can also remove the `android:tint` attribute. Compose Multiplatform ignores it, so this step is optional.
   To adjust the icon color, use the `Image.colorFilter` parameter as shown below.

   <compare>
        <code-block lang="xml"><![CDATA[
            <vector xmlns:android="..."
                android:width="24dp"
                android:height="24dp"
                android:viewportWidth="960"
                android:viewportHeight="960"
                android:tint="?attr/colorControlNormal">
                <path
                    android:fillColor="@android:color/white"
                    android:pathData="..."/>
            </vector>
        ]]></code-block>
        <code-block lang="xml"><![CDATA[
            <vector xmlns:android="..."
                android:width="24dp"
                android:height="24dp"
                android:viewportWidth="960"
                android:viewportHeight="960">
   
                <path
                    android:fillColor="#000000"
                    android:pathData="..."/>
            </vector>
        ]]></code-block>
   </compare>
   
4. Build the project to generate the resource accessors, 
   or let the [Kotlin Multiplatform IDE plugin](https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform) handle it automatically.

Here's an example of how you can access an icon and adjust the colors using the `colorFilter` parameter
in your Compose Multiplatform code:

```kotlin
Image(
    painter = painterResource(Res.drawable.ic_sample_icon),
    contentDescription = "Sample icon",
    modifier = Modifier.size(24.dp),
    colorFilter = ColorFilter.tint(Color.Blue)
)
```

### Strings

Store all string resources in XML files in `composeResources/values` directories.
A static accessor is generated for each item in each file.

To learn how to localize strings for different locales, refer to the 
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

The `getString()`, `getStringArray()`, and `getPluralString()` functions also have overloads that accept a `ResourceEnvironment`
as the first parameter. To get an environment, use one of the following functions:

* `getSystemResourceEnvironment()` returns the environment based on the current system settings:

  ```kotlin
  coroutineScope.launch {
      val appName = getString(getSystemResourceEnvironment(), Res.string.app_name)
  }
  ```
  
  `getString()`, `getStringArray()`, and `getPluralString()` use this function by default.

* `rememberResourceEnvironment()` returns the environment of the current composition,
  including the settings [overridden in the app](compose-resource-environment.md).
  It is a composable function, so call it in composable code and pass the result to non-composable code,
  for example, to a coroutine:

  ```kotlin
  val environment = rememberResourceEnvironment()

  LaunchedEffect(environment) {
      val appName = getString(environment, Res.string.app_name)
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

> In the IDE with the [Kotlin Multiplatform plugin](https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform), 
> you can use an Emmet-like shorthand syntax to add string resources, string arrays, and plurals directly in XML files.
> Type an abbreviation in `strings.xml` and press **Tab** to expand it into string tags:
> * `test` → `<string name="test"></string>`
> * `test{Example}` or `s.test{Example}` → `<string name="test">Example</string>`
>
{style="tip"}

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

```XML
<resources>
    <string name="price">Price: %1$s</string>
</resources>
```

```kotlin
Text(stringResource(Res.string.price, 100.1f))
// "Price: 100.1"
```

> In the IDE with the [Kotlin Multiplatform plugin](https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform),
> instead of manually typing `%1$s` or `%2$d` for placeholders, you can use inline numeric shortcuts. 
> For example, when you type `1` or `1s` within the string value, it expands into `%1$s`.
> Similarly, when you type `2d`, it expands into `%2$d`.
> 
{style="tip"}

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
    val items = getStringArray(Res.array.str_arr)
}
```

</tab>
</tabs>

> In the IDE with the [Kotlin Multiplatform plugin](https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform),
> you can use Emmet-like syntax to quickly define string arrays.
> Use `string-array`, `sa`, or the `>` operator to generate an empty array template.
> For a named array with a predefined number of items and starting text, type `test>2{Hello}` and press **Tab**:
> ```xml
> <string-array name="test">
>    <item>Hello</item>
>    <item>Hello</item>
> </string-array>
> ```
>
{style="tip"}

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
* It's often possible to avoid quantity strings by using quantity-neutral formulations such as "Books: 1",
  if this doesn't worsen the user experience.

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
    val message = getPluralString(Res.plurals.new_message, 1, 1)
}
```

</tab>
</tabs>

The `quantity` parameter only selects the grammatical form; to insert the number into the string,
pass it as a format argument: in `pluralStringResource(Res.plurals.new_message, 1, 1)`,
the first `1` is the quantity, and the second `1` replaces the `%1$d` placeholder.

> In the IDE with the [Kotlin Multiplatform plugin](https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform),
> you can use Emmet-like syntax to generate plural resources.
> For example, use `plurals`, `p`, or `:` to generate a default empty string template.
> If you work in `values-en/strings.xml`, the IDE automatically detects locale, the required quantities,
> and that English only requires `one` and `other`.
> Type `p.test` or `plurals.test `and press **Tab** to expand the abbreviation into the `plurals` block:
> ```xml
> <plurals name="test">
>     <item quantity="one"></item>
>     <item quantity="other"></item>
> </plurals>
> ```
>
{style="tip"}

### Fonts

Store custom fonts in the `composeResources/font` directory.
Compose Multiplatform supports TTF, OTF, TTC, and variable font formats on all platforms.

WOFF and WOFF2 are available on web, on iOS, and on desktop when the app runs on macOS. 
On desktop, support depends on the operating system that runs the app, not on the target you build for:
a WOFF or WOFF2 font that works on macOS may not load on Windows or Linux.
When a font format is not supported, no error is reported, and the default font is used instead.

To load a font as a `Font` type, use the `Font()` composable function:

```kotlin
@Composable
fun Font(
    resource: FontResource,
    weight: FontWeight = FontWeight.Normal,
    style: FontStyle = FontStyle.Normal,
    variationSettings: FontVariation.Settings = FontVariation.Settings(weight, style)
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
            titleLarge = titleLarge.copy(fontFamily = interFont, fontWeight = FontWeight.Bold),
            bodyLarge = bodyLarge.copy(fontFamily = interFont, fontWeight = FontWeight.Normal),
            // Apply the font family to other text styles in the same way
        )
    }
}
```

> Since `Font()` is a composable function, you must call it while inside a composition.
> Create the `FontFamily`, `TextStyle`, and `Typography` objects that use `Font()` inside a `@Composable` function.
>
{style="note"}

Special characters like emojis and Arabic script are supported automatically in web
targets through [automatic font fallback](compose-web-resources.md#automatic-font-fallback),
which downloads the required Noto fonts on demand.

If you need full control over which font is used, bundle a specific font and register it
manually with the [preload API](compose-web-resources.md#preload-resources-using-the-compose-multiplatform-preload-api).

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

If the file you are reading is a bitmap (JPEG, PNG, BMP, WEBP) or an XML vector image, you can use the
`decodeToImageBitmap()` and `decodeToImageVector()` functions to convert them into image objects suitable 
for the `Image()` composable.

Access the raw files as shown in the [Raw files](#raw-files) section, then pass the result to a composable.
Make sure the bytes are loaded before decoding them: decoding an empty byte array throws an exception.

```kotlin
var bytes by remember { mutableStateOf(ByteArray(0)) }
LaunchedEffect(Unit) {
    bytes = Res.readBytes("files/example.png")
}
if (bytes.isNotEmpty()) {
    Image(bytes.decodeToImageBitmap(), null)
}
```

In the same way, decode an XML vector image:

```kotlin
// bytes = Res.readBytes("files/example.xml")
Image(bytes.decodeToImageVector(LocalDensity.current), null)
```

On every platform except Android, you can also turn an SVG file into a `Painter` object:

```kotlin
// bytes = Res.readBytes("files/example.svg")
Image(bytes.decodeToSvgPainter(LocalDensity.current), null)
```

The `decodeToSvgPainter()` function is not available in `commonMain`, because it doesn't exist on Android.
Call it from a source set that is not compiled for Android, or use the `expect`/`actual` mechanism.

### Generated maps for resources and string IDs

For ease of access, Compose Multiplatform also maps resources to string IDs:

```kotlin
val Res.allDrawableResources: Map<String, DrawableResource>
val Res.allStringResources: Map<String, StringResource>
val Res.allStringArrayResources: Map<String, StringArrayResource>
val Res.allPluralStringResources: Map<String, PluralStringResource>
val Res.allFontResources: Map<String, FontResource>
```

An example of passing a mapped resource to a composable:

```kotlin
import project.shared.generated.resources.allDrawableResources
// ...

Res.allDrawableResources["compose_multiplatform"]?.let { resource ->
    Image(painterResource(resource), null)
}
```

* For drawable and font resources, the key is the filename without the extension, with hyphens replaced by underscores:
  for example, `compose-multiplatform.xml` can be accessed by the `compose_multiplatform` key.
* For strings, string arrays, and plurals, the key is the value of the `name` attribute.

Compose Multiplatform generates maps as extension properties that are located in the same package as the `Res` class.
The maps should be imported explicitly.

The maps have the same visibility as the `Res` class, so they are internal unless you set [`publicResClass`](#customizing-accessor-class-generation)
to `true`. There is no map for raw files.

### Compose Multiplatform resources as Android assets

Starting with Compose Multiplatform 1.7.0, all multiplatform resources are packed into Android assets.
This enables Android Studio to generate previews for Compose Multiplatform composables in Android source sets.

> Android Studio previews are available only for composables in an Android source set.
> They also require a version of the Android Gradle plugin that is [compatible](compose-compatibility-and-versioning.md)
> with your Compose Multiplatform version.
>
{style="warning"}

Using Multiplatform resources as Android assets also makes possible direct access from WebViews and media player components
on Android, since resources can be reached by a simple path, for example `Res.getUri("files/index.html")`.

An example of an Android composable displaying a resource HTML page with a link to a resource image:

```kotlin
// androidMain/kotlin/com/example/webview/ResourceWebView.kt
@Composable
@Preview
fun ResourceWebView() {
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
URIs of the resource files to these other APIs.
To get a platform-specific URI, call the `Res.getUri()` function with the project path to the resource:

```kotlin
val uri = Res.getUri("files/myDir/someFile.bin")
```

The format of the URI depends on the platform:

| Platform | URI example                                                                                                                                                                                                             |
|----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Android  | `file:///android_asset/composeResources/<package of the Res class>/files/myDir/someFile.bin`                                                                                                                            |
| Desktop  | `jar:file:/<path to the application JAR>!/composeResources/<package of the Res class>/files/myDir/someFile.bin` in a packaged application, or `file:/<path to the resources directory>/composeResources/<package of the Res class>/files/myDir/someFile.bin` when the resources are not packed into a JAR, for example, in tests |
| iOS      | `file:///<path to the application bundle>/compose-resources/composeResources/<package of the Res class>/files/myDir/someFile.bin`                                                                                       |
| Web      | `<page origin><page path>./composeResources/<package of the Res class>/files/myDir/someFile.bin`, or the URL produced by your [custom path mapping](compose-multiplatform-resources-setup.md#custom-web-resource-paths) |

On web, `Res.getUri()` adds the path of the current page as is. If the page URL ends with a file name, for example,
`https://example.com/index.html`, the result with the default path mapping is not a valid resource URL:
`https://example.com/index.html./composeResources/...`. To get a valid URL in this case, use a
[custom path mapping](compose-multiplatform-resources-setup.md#custom-web-resource-paths) that starts with `/` or `http`.

On desktop, a `jar:` URI points to a file inside the application JAR, so it can't be opened as a file,
for example, with `java.io.File`. If a library needs a file path, read the resource with `Res.readBytes()` and write it
to a file first.

If the resource doesn't exist, `Res.getUri()` throws a `MissingResourceException` on Android and desktop.
On iOS and web, the path is not checked, and the function returns a URI even for a missing resource.

For Android-specific uses, multiplatform resources are also [packed as Android assets](#compose-multiplatform-resources-as-android-assets).

### Remote files

In the context of the resource library, only files that are part of the application are considered resources.

You can load remote files from the internet using their URL using specialized libraries:

* [Compose ImageLoader](https://github.com/qdsfdhvh/compose-imageloader)
* [Kamel](https://github.com/Kamel-Media/Kamel)
* [Ktor client](https://ktor.io/)

If you download or read the bytes of an image yourself, convert them into an image with the
[decoding functions](#convert-byte-arrays-into-images) of the resources library.
For an example of loading images from the network in a desktop application, see
the tutorial for [using images in Compose Multiplatform for Desktop](compose-desktop-images.md#loading-images-from-the-file-system-or-the-network).

### Using Java resources

While you can use Java resources with Compose Multiplatform, they don't benefit from extended features provided by
the framework: generated accessors, multimodule support, localization, and so on.
Consider transitioning fully to the multiplatform resource library to unlock that potential.

With Compose Multiplatform 1.7.0, the resources API available in the `compose.ui` package is deprecated.
If you still need to work with Java resources, copy the following implementation to your project to ensure that your code
works after you upgrade to Compose Multiplatform 1.7.0 or above. 
The implementation uses JVM-only APIs, so place it in a desktop (JVM) source set:

```kotlin
import androidx.compose.runtime.Composable
import androidx.compose.runtime.remember
import androidx.compose.ui.graphics.painter.BitmapPainter
import androidx.compose.ui.graphics.painter.Painter
import androidx.compose.ui.graphics.vector.rememberVectorPainter
import androidx.compose.ui.platform.LocalDensity
import org.jetbrains.compose.resources.decodeToImageBitmap
import org.jetbrains.compose.resources.decodeToImageVector
import org.jetbrains.compose.resources.decodeToSvgPainter

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
private fun readResourceBytes(resourcePath: String): ByteArray =
    ResourceLoader.javaClass.classLoader.getResourceAsStream(resourcePath)
        ?.use { it.readAllBytes() }
        ?: error("Resource not found: $resourcePath")
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="internal fun painterResource(resourcePath: String): Painter"}

> The `painterResource(String)` function has the same name as `org.jetbrains.compose.resources.painterResource()`.
> Check the imports in your files to make sure that the intended function is called.
>
{style="note"}

## What's next?

* Check out the official [demo project](https://github.com/JetBrains/compose-multiplatform/tree/master/components/resources/demo)
that shows how resources can be handled in a Compose Multiplatform project targeting iOS, Android, and desktop.
* Learn how to manage the application's [resource environment](compose-resource-environment.md) like in-app theme and language.
