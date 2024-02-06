[//]: # (title: Images and resources)

> This page describes the new revamped version of the resource library available since Compose Multiplatform 1.6.0.
> 
{type="note"}

Compose Multiplatform provides a special library and a Gradle plugin support for accessing resources in common code
across all supported platforms. Resources are static data, such as images, fonts, and strings, that you can use from
your application.

> The library is [Experimental](supported-platforms.md#core-kotlin-multiplatform-technology-stability-levels).
> Its API may change in the future.
>
{type="warning"}

When working with resources in Compose Multiplatform, consider the current conditions:

* All resource types (except raw files) are read synchronously in the caller thread. JS is the only platform
  where resources are downloaded asynchronously.
* It's expected that resources are relatively small files, like icons and texts, that you load and use as part of the UI.
* Reading big raw files, like long videos, as a stream is not supported yet. Use separate files on the user device and
  read them with the file system API, for example, the [kotlinx-io](https://github.com/Kotlin/kotlinx-io) library.
* Multimodule projects are not supported yet. Store all resources in the main application module.
* The publication of Compose Multiplatform libraries with resources is not supported yet.

## Configuration

To access resources in your multiplatform projects:

1. In the `build.gradle.kts` file in the `composeApp` directory, add a dependency to the `commonMain` source set:

   ```kotlin
   kotlin {
       sourceSets {
           commonMain.dependencies {
               implementation("compose.components.resources")
           }
       }
   }
   ```

2. Create a new directory `composeResources` in the `commonMain` directory:

   ![Compose resources project structure](compose-resources-structure.png){width=250}

3. Organize the `composeResources` directory structure accordingly:

   * Images should be in the `drawable` directory.
   * Fonts should be in the `font` directory.
   * Strings (`strings.xml`) should be in the `values` directory.
   * Other files with any hierarchy should be in `files` directory.

## Qualifiers

Sometimes, the same resource should be presented in different ways depending on the environment, such as locale,
screen density, or an interface theme. For example, you might need to localize texts for different languages or adjust
images for the dark theme. For that, the library provides special qualifiers.

All resource types (except for raw files in the `files` directory) support qualifiers. Apply qualifiers to directory
names using a dash:

![Qualifiers in compose resources](compose-resources-qualifiers.png){width=250}

The library supports (in the order of priority) language, theme, and density qualifiers.

### Language and regional qualifiers

The language is defined by a two-letter [ISO 639-1](https://en.wikipedia.org/wiki/List_of_ISO_639_language_codes)
language code.

You can add a two-letter [ISO 3166-1-alpha-2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) regional code
to your language code. In this case, the regional code must have a lowercase `r` prefix.

The language and regional codes are not case-sensitive.

### Theme qualifier

You can use "light" or "dark" qualifiers. The resource is selected depending on the current system theme.

### Density qualifier

You can use the following density qualifiers:

* "ldpi" − 120 DPI, 0.75x density
* "mdpi" − 160 DPI, 1x density
* "hdpi" − 240 DPI, 1.5x density
* "xhdpi" − 320 DPI, 2x density
* "xxhdpi" − 480 DPI, 3x density
* “xxxhdpi” − 640dpi, 4x density

The resource is selected depending on a screen density defined in the system.

Different types of qualifiers can be applied together. For example, "drawable-en-rUS-mdpi-dark" is an image for the
English language in the United States region, suitable for 160 DPI screens in the dark theme.

If a resource with the requested qualifier doesn't exist, a default resource without a qualifier is used instead.

## Resource usage

After importing a project, a special `Res` class that provides access to resources is generated.
To force run the `Res` class generation, use the `generateComposeResClass` Gradle task.

### Images

You can access drawable resources as simple images, rasterized images, and XML vectors:

* To access drawable resources as `Painter` images, use the `painterResource()` function:

  ```kotlin
  @Composable
  fun painterResource(resource: DrawableResource): Painter {...}
  ```
    
  The `painterResource()` function takes a resource path and returns a `Painter` value. The function works synchronously
  on all targets, except for web. For the web target, it returns an empty `Painter` for the first recomposition that is
  replaced with the loaded image in subsequent recompositions.
    
  * `painterResource()` loads either a `BitmapPainter` for rasterized image formats, such as `.png`, `.jpg`, `.bmp`, `.webp`,
    or a `VectorPainter` for XML vector drawable formats like `.xml`.
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

Store all string resources in the `strings.xml` file in the `composeResources/values` directory, for example:

```XML
<resources>
    <string name="app_name">My awesome app</string>
    <string name="title">Some title</string>
</resources>
```

A static accessor is generated for each item in the `strings.xml` file.

To get string resources as a `String`:

<tabs>
<tab title= "From composable code">

```kotlin
@Composable
fun stringResource(resource: StringResource): String {...}

@Composable
fun stringResource(resource: StringResource, vararg formatArgs: Any): String {...}

@Composable
fun stringArrayResource(resource: StringResource): List<String> {...}
```

For example:

```kotlin
Text(stringResource(Res.string.app_name))
```

</tab>
<tab title= "From a non-composable code">

```kotlin
suspend fun getString(resource: StringResource): String

suspend fun getString(resource: StringResource, vararg formatArgs: Any): String

suspend fun getStringArray(resource: StringResource): List<String>
```

For example:

```kotlin
coroutineScope.launch {
    val appName = getString(Res.string.app_name)
}
```

</tab>
</tabs>

Strings with arguments currently have basic support:

```XML
<!-- strings.xml -->
<resources>
	<string name="str_template">Hello, %1$s! You have %2$d new messages.</string>
</resources>
```

There is no difference between `%...s` and `%...d `, for example:

```kotlin
Text(stringResource(Res.string.str_template, "User_name", 100))
```

### Fonts

Store custom fonts in the `composeResources/font` directory as `*.ttf` files.

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

To load any file as a byte array, use the `Res.readBytes()` function:

```kotlin
suspend fun readBytes(path: String): ByteArray
```

You can place such files in the `composeResources/files` directory and create any hierarchy inside it.

For example, to access raw files:

<tabs>
<tab title= "From composable code">

```kotlin
var bytes by remember {
    mutableStateOf(ByteArray(0))
}
LaunchedEffect(Unit) {
    bytes = Res.readFileBytes("files/myDir/someFile.bin")
}
Text(bytes.decodeToString())
```

</tab>
<tab title= "From a non-composable code">

```kotlin
coroutineScope.launch {
    val bytes = Res.readFileBytes("files/myDir/someFile.bin")
}
```

</tab>
</tabs>

### Remote files

Only files that are a part of the application are considered as resources.

You can also load remote files from the internet using their URL. To load them, use special libraries:

* [Compose ImageLoader](https://github.com/qdsfdhvh/compose-imageloader)
* [Kamel](https://github.com/Kamel-Media/Kamel)
* [Ktor client](https://ktor.io/)

## What's next?

Check out the official [demo project](https://github.com/JetBrains/compose-multiplatform/tree/master/components/resources/demo)
that shows how resources can be handled in a Compose Multiplatform project targeting iOS, Android, and desktop.
