[//]: # (title: Images and resources)

Compose Multiplatform provides a special library for accessing resources in a common way for all platforms. Resources
are static data that you can use from your application. It could be images, fonts, and strings.

> The library is currently [Experimental](supported-platforms.md#core-kotlin-multiplatform-technology-stability-levels).
> Its API may be changed in the future.
> 
> Currently, the library supports accessing resources as raw data and as images. The JetBrains team is planning to extend
> this functionality.
>
{type="note"}

## Configuration

To access resources in your multiplatform projects:

* For common code, store your resource files in the `resources` directory of the `commonMain` source set.
* For platform-specific source sets, use the `resources` directory of the corresponding source set.

![Compose resources project structure](compose-resources-structure.png){width=250}

Deployment of resources differs on each platform:

* On iOS, resources get into the resulting application bundle as simple files.
* On Android, resources are stored in the `.apk` files.
* On the desktop, resources are stored in the `.jar` files.

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
It stores resource files in the `compose-resources` directory of the resulting application bundle.

> Currently, the plugin doesn't support the deployment of resources on iOS in multimodule projects. This limitation will
> be lifted in future releases.
>
{type="note"}

To add the dependency on the resource library, use the following configuration in your `build.gradle.kts`:

```kotlin
val commonMain by getting {
    dependencies {
        // Your dependencies
        @OptIn(org.jetbrains.compose.ExperimentalComposeLibrary::class)
        implementation(compose.components.resources)
    }
}
```

## Accessing images

To access resources as images, use the `painterResource()` function:

```kotlin
@Composable
fun painterResource(res: String): Painter 
```

It takes a resource path and returns a `Painter` value. The function works synchronously on all targets except for the
web. For the web target, it returns an empty painter for the first recomposition that will be replaced with the loaded
image on subsequent recompositions.

* `painterResource()` loads either a `BitmapPainter` for raster image formats, like `.png`, `.jpg`, `.bmp`, `.webp`,
  or a `VectorPainter` for XML Vector Drawable formats, like `.xml`.
* XML Vector Drawables have the same format as [Android](https://developer.android.com/reference/android/graphics/drawable/VectorDrawable),
  except that they don't support external references to Android resources.

Note that the configuration differs from the standard Android resource management.

## Accessing arbitrary resources as raw data

To access an arbitrary resource as raw data, use the `resource()` function:

```kotlin
fun resource(path: String): Resource
```

It takes a resource path and returns an instance of an abstract `Resource` type. The type has only a suspended
public function that converts your resource into a byte array:

```kotlin
suspend fun readBytes(): ByteArray
```

By design, resources are accessed asynchronously to not block the main UI thread while a resource is loaded.
To access resources this way, you can call the [`readBytes()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.io/java.io.-file/read-bytes.html)
method in a coroutine or in the `LaunchedEffect` block. Here's how you can load a resource asynchronously and convert it into a string:

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

If you want read your resource synchronously, use the `runBlocking()` function.
You can implement synchronous access to resources like this:

```kotlin
val byteArray = runBlocking {
    resource("welcome.txt").readBytes()
}
```

> The `runBlocking()` function is not available for the web target.
>
{type="note"}

## Accessing fonts and strings resources

Currently, the library doesn't support fonts and string resources. You also can't group your image resources by
resolution. The JetBrains team is planning to implement this functionality in future releases.

Meanwhile, you can use the following third-party Compose Multiplatform libraries for accessing resources that
(partially) address these issues:

* [Moko resources](https://github.com/icerockdev/moko-resources)
* [Skeptick/libres](https://github.com/Skeptick/libres)

Both libraries support localized string resources. The Moko resources library also supports fonts and colors and allows
you to group images by resolution.

> The JetBrains resource library cannot be used simultaneously with third-party libraries on iOS. All of them use the
> Gradle plugin approach for deployment resources on iOS that work differently for the libraries and conflict with each
> other.
>
{type="note"}

## Loading images from the internet

To load an image from the internet, you can use the following third-party Compose Multiplatform libraries:

* [Compose ImageLoader](https://github.com/qdsfdhvh/compose-imageloader)
* [Kamel](https://github.com/Kamel-Media/Kamel)

They handle image caching, so you don't need to download the same image multiple times, and networking for downloading
your images from the internet.
