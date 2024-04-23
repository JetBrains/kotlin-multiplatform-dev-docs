[//]: # (title: Multiplatform resources)

Compose Multiplatform provides a special library and Gradle plugin support for accessing resources in common code
across all supported platforms. Resources are static content, such as images, fonts, and strings, which you can use from
your application.

> The library is [Experimental](supported-platforms.md#core-kotlin-multiplatform-technology-stability-levels).
> Its API may change in the future.
>
{type="warning"}

When working with resources in Compose Multiplatform, consider the current conditions:

* Almost all resources are read synchronously in the caller thread. The only exceptions are raw files
  and all of the resources on the JS platform that are read asynchronously.
* Reading big raw files, like long videos, as a stream is not supported yet.
  Use the [`getUri()`](#accessing-resources-from-external-libraries) function to pass separate files to a system API,
  for example, the [kotlinx-io](https://github.com/Kotlin/kotlinx-io) library.
* Multimodule projects are generally not supported yet, but resources in the top-level application module are available. The JetBrains team is working on adding this functionality in future releases.
  For now, store all resources in the main application module.
* The publication of Compose Multiplatform libraries with resources is not supported yet. The JetBrains team is working
  on adding this functionality in future releases.
* Currently, accessors are generated for the `commonMain` source set only. The JetBrains team is working on expanding
  this functionality in future releases.

  However, you can still store platform-specific resources in a platform `composeResources` directory and read them as a
  byte array. All resources will be included in each final app.

> Since the version %composeEapVersion%-beta01, limitations on resource placement have been lifted:
> you can store resources in any Gradle module and any source set, as well as publish projects and libraries
> with resources included.
> 
> To use the multimodule functionality, you'll need a 2.0.0-Beta5 or newer version of the [Kotlin Gradle plugin](https://plugins.gradle.org/plugin/org.jetbrains.kotlin.android)
> and Gradle 7.6 or newer.
>
{type="tip"}

## Setup

To access resources in your multiplatform projects:

1. In the `build.gradle.kts` file in the `composeApp` directory, add a dependency to the `commonMain` source set:

   ```kotlin
   kotlin {
       sourceSets {
           commonMain.dependencies {
               implementation(compose.components.resources)
           }
       }
   }
   ```

2. Create a new directory `composeResources` in the `commonMain` directory:

   ![Compose resources project structure](compose-resources-structure.png){width=250}

3. Organize the `composeResources` directory structure according to these rules:

   * Images should be in the `drawable` directory.
   * Fonts should be in the `font` directory.
   * Strings should be in the `values` directory.
   * Other files with any hierarchy should be in the `files` directory.

4. Starting with Compose Multiplatform %composeEapVersion%-beta01, you can alter the default settings for Compose Multiplatform
   resources by adding a `compose.resources{}` block to the `build.gradle.kts` file:

    ```kotlin
    compose.resources {
        publicResClass = true
        packageOfResClass = "me.sample.library.resources"
        generateResClass = always
    }
    ```

   * `publicResClass` set to `true` makes the generated `Res` class public. By default, the generated class is [internal](https://kotlinlang.org/docs/visibility-modifiers.html).
   * `packageOfResClass` allows you to assign the generated `Res` class to a particular package (for access within the code
     as well as for isolation in a final artifact.) By default, Compose Multiplatform uses the
     `{group name}.{module name}.generated.resources` package.
   * `generateResClass` set to `always` makes the project unconditionally generate the `Res` class. This may be useful
     when the resource library is only available transitively. By default, the `auto` value is used: generate the `Res`
     class only if the current project has an explicit `implementation` or `api` dependency on the resource library.

## Qualifiers

Sometimes, the same resource should be presented in different ways depending on the environment, such as locale,
screen density, or interface theme. For example, you might need to localize texts for different languages or adjust
images for the dark theme. For that, the library provides special qualifiers.

All resource types (except for raw files in the `files` directory) support qualifiers. Apply qualifiers to directory
names using a hyphen:

![Qualifiers in compose resources](compose-resources-qualifiers.png){width=250}

The library supports (in the order of priority) the following qualifiers: [language](#language-and-regional-qualifiers),
[theme](#theme-qualifier), and [density](#density-qualifier).

* Different types of qualifiers can be applied together. For example, "drawable-en-rUS-mdpi-dark" is an image for the
  English language in the United States region, suitable for 160 DPI screens in the dark theme.
* If a resource with the requested qualifier doesn't exist, the default resource without a qualifier is used instead.

### Language and regional qualifiers

The language is defined by a two-letter [ISO 639-1](https://en.wikipedia.org/wiki/List_of_ISO_639_language_codes)
language code.

You can add a two-letter [ISO 3166-1-alpha-2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2) regional code
to your language code. In this case, the regional code must have a lowercase `r` prefix.

The language and regional codes are case-sensitive.

### Theme qualifier

You can add "light" or "dark" qualifiers. Compose Multiplatform then selects the necessary resource depending on the
current system theme.

### Density qualifier

You can use the following density qualifiers:

* "ldpi" − 120 DPI, 0.75x density
* "mdpi" − 160 DPI, 1x density
* "hdpi" − 240 DPI, 1.5x density
* "xhdpi" − 320 DPI, 2x density
* "xxhdpi" − 480 DPI, 3x density
* "xxxhdpi" − 640dpi, 4x density

The resource is selected depending on the screen density defined in the system.

## Resource usage

After importing a project, a special `Res` class is generated which provides access to resources.
To manually generate the `Res` class, run the `generateComposeResClass` Gradle task.

### Images

You can access drawable resources as simple images, rasterized images, or XML vectors:

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

Store all string resources in the `strings.xml` file in the `composeResources/values` directory.
A static accessor is generated for each item in the `strings.xml` file.

> Starting with Compose Multiplatform %composeEapVersion%-beta01,
> accessors for strings are correctly generated for all XML files in the `values` directory.
>
{type="warning"}

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

@Composable
fun stringArrayResource(resource: StringResource): List<String> {...}
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

> Starting with Compose Multiplatform %composeEapVersion%-beta01, string arrays are a separate resource type.
>
{type="note"}

You can use special symbols in string resources:

* `\n` — for a new line
* `\t` — for a tab symbol
* `\uXXXX` — for a specific Unicode character

#### String templates

Currently, arguments have basic support in string resources:

```XML
<!-- strings.xml -->
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

> Starting with Compose Multiplatform %composeEapVersion%-beta01, string arrays are a separate string resource type.
>
{type="note"}

You can group related strings into an array and automatically access them as a `List<String>` object:

```XML
<resources>
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
Text(stringArrayResource(Res.string.str_arr))
```

</tab>
<tab title= "From non-composable code">

```kotlin
suspend fun getStringArray(resource: StringArrayResource): List<String>
```

For example:

```kotlin
coroutineScope.launch {
    val appName = getStringArray(Res.string.str_arr)
}
```

</tab>
</tabs>

#### Plurals

> Available starting with Compose Multiplatform %composeEapVersion%-beta01.
>
{type="warning"}

When your UI displays quantities of something, you might want to support grammatical agreement for different numbers
of the same thing (1 _book_, many _books_ and so on) without creating programmatically unrelated strings.

The concept and base implementation in Compose Multiplatform is the same as for [quantity strings](https://developer.android.com/guide/topics/resources/string-resource#Plurals)
on Android (follow the link to read more about nuances and best practices of using plurals in your project).

* The supported variants are `zero`, `one`, `two`, `few`, `many`, and `other`. Note that not all variants are even 
  considered for every language: for example, `zero` is ignored for English, because it is the same as any other plural
  except 1. Rely on a language specialist to know what distinctions the language actually insists upon.
* It's often possible to avoid quantity strings by using quantity-neutral formulations such as "Books: 1".
  If this doesn't worsen the user experience, 

To define a plural, add a `<plurals>` element to any .xml file in your `composeResources/values` directory.
A `plurals` collection is a simple resource referenced using the name attribute (not the name of the XML file).
As such, you can combine `plurals` resources with other simple resources in the one XML file, under one `<resources>` element:

```xml
<resources>
    <string name="app_name">Compose Resources App</string>
    <string name="hello">😊 Hello world!</string>
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
Text(pluralStringResource(Res.string.new_message, 1))
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
    val appName = getPluralString(Res.string.new_message, 1)
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
    bytes = Res.readFileBytes("files/myDir/someFile.bin")
}
Text(bytes.decodeToString())
```

</tab>
<tab title= "From non-composable code">

```kotlin
coroutineScope.launch {
    val bytes = Res.readFileBytes("files/myDir/someFile.bin")
}
```

</tab>
</tabs>

### Remote files

Only files that are part of the application are considered resources.

You can also load remote files from the internet using their URL. To load remote files, use special libraries:

* [Compose ImageLoader](https://github.com/qdsfdhvh/compose-imageloader)
* [Kamel](https://github.com/Kamel-Media/Kamel)
* [Ktor client](https://ktor.io/)

### Accessing resources from external libraries

> Available starting with Compose Multiplatform %composeEapVersion%-beta01.
>
{type="warning"}

If you would like to make Compose Multiplatform resources available to other libraries used in your project, you
can call the `Res.getUri()` function with the general path of the resource:

```kotlin
val uri = Res.getUri("files/my_video.mp4")
```

The `uri` variable will contain the precise platform-specific path to the file that the path points to.
External libraries can use that path to access the file in a manner that suits them.

## Publication

Starting with Compose Multiplatform %composeEapVersion%-beta01, all necessary resources are included in the publication
package, regardless of module and source set hierarchy.

To enable this functionality, your project needs to use Kotlin 2.0.0-Beta5 or newer and Gradle 7.6 or newer.

## What's next?

Check out the official [demo project](https://github.com/JetBrains/compose-multiplatform/tree/master/components/resources/demo)
that shows how resources can be handled in a Compose Multiplatform project targeting iOS, Android, and desktop.
