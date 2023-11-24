[//]: # (title: Images and resources)

Compose Multiplatform provides a special library for accessing resources in a common way for all platforms. Resources
are static data, such as images, fonts, strings, that you can use from your application.

> The library is [Experimental](supported-platforms.md#core-kotlin-multiplatform-technology-stability-levels).
> Its API may change in the future.
> 
> Currently, the library supports accessing resources as raw data and images. The JetBrains team plans to extend
> this functionality.
>
{type="note"}

## Configuration

To access resources in your multiplatform projects:

* For common code, store your resource files in the `resources` directory of the `commonMain` source set.
* For platform-specific code, store your resource files in the `resources` directory of the corresponding source set.

![Compose resources project structure](compose-resources-structure.png){width=250}

## Configuration

The deployment of resources differs per platform:

* On Android, resources are stored in the `.apk` files.
* On iOS, resources are included in the resulting application bundle as simple files.
* On desktop, resources are stored in the `.jar` files.

### Android

To make your resources accessible from the resource library, use the following configuration in your `build.gradle.kts`
file:

```kotlin
android {
    // …
    sourceSets["main"].resources.srcDirs("src/commonMain/resources")
}
```

Note that the configuration differs from the standard Android resource configuration.

### iOS

For iOS, the Compose Multiplatform Gradle plugin handles resource deployment.
The plugin stores resource files in the `compose-resources` directory of the resulting application bundle.

> Currently, the plugin doesn't support deployment of resources on iOS in multimodule projects. This limitation will
> be removed in future releases.
>
{type="note"}

To add a dependency on the resource library, use the following configuration in your `build.gradle.kts` file:

```kotlin
val commonMain by getting {
    dependencies {
        // Your dependencies
        @OptIn(org.jetbrains.compose.ExperimentalComposeLibrary::class)
        implementation(compose.components.resources)
    }
}
```

## Access images

To access resources as images, use the `painterResource()` function:

```kotlin
@Composable
fun painterResource(res: String): Painter 
```

The `painterResource()` function takes a resource path and returns a `Painter` value. The function works synchronously
on all targets, except for web. For the web target, it returns an empty `Painter` for the first recomposition that is
replaced with the loaded image in subsequent recompositions.

* `painterResource()` loads either a `BitmapPainter` for rasterized image formats, like `.png`, `.jpg`, `.bmp`, `.webp`,
  or a `VectorPainter` for XML Vector Drawable formats, like `.xml`.
* XML Vector Drawables have the same format as [Android](https://developer.android.com/reference/android/graphics/drawable/VectorDrawable),
  except that they don't support external references to Android resources.

Here's an example of how you can access images in your compose code:

```kotlin
Image(
    painterResource("compose-multiplatform.xml"),
    null // description
)
```

## Access arbitrary resources as raw data

To access an arbitrary resource as raw data, use the `resource()` function:

```kotlin
fun resource(path: String): Resource
```

The `resource()` function takes a resource path and returns an instance of an abstract `Resource` type. The type has only a suspended
public function that converts your resource into a byte array:

```kotlin
suspend fun readBytes(): ByteArray
```

By design, resources are accessed asynchronously to not block the main UI thread while a resource is loaded.
To access resources this way, you can call the [`readBytes()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.io/java.io.-file/read-bytes.html)
method in a coroutine or in the `LaunchedEffect` block.

Here's an example of how you can load a resource asynchronously and convert it into a string:

```kotlin
@OptIn(ExperimentalResourceApi::class)
@Composable
fun App() {
    var text: String? by remember { mutableStateOf(null) }

    LaunchedEffect(Unit) {
        text = String(resource("welcome.txt").readBytes())
    }


    text?.let {
        Text(it)
    }
}
```

If you want to read your resource synchronously, use the `runBlocking()` function.
You can implement asynchronous access to resources like this:

```kotlin
val byteArray = runBlocking {
    resource("welcome.txt").readBytes()
}
```

> The `runBlocking()` function is not available for the web target.
>
{type="note"}

## Access fonts and strings resources

Currently, the library doesn't support fonts and strings resources. You also can't group your image resources by
resolution. The JetBrains team is planning to implement this functionality in future releases.

Meanwhile, you can use the following third-party Compose Multiplatform libraries for accessing resources that
(partially) addresses these issues:

* [Moko resources](https://github.com/icerockdev/moko-resources)
* [Skeptick/libres](https://github.com/Skeptick/libres)

Both libraries support localized string resources. The Moko resources library also supports fonts and colors, and allows
you to group images by resolution.

> The Compose Multiplatform resource library can't be used simultaneously with third-party libraries on iOS. Each
> library uses its own Gradle plugin to deploy resources on iOS. Plugins work differently, so using more than one
> causes them to conflict with each other.
>
{type="note"}

## Load images from the internet

To load an image from the internet, you can use the following third-party Compose Multiplatform libraries:

* [Compose ImageLoader](https://github.com/qdsfdhvh/compose-imageloader)
* [Kamel](https://github.com/Kamel-Media/Kamel)

They handle image caching, so you don't need to download the same image multiple times, and networking for downloading
your images from the internet.
