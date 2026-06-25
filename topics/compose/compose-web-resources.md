[//]: # (title: Handling web resources)

Here you can find information about preloading resources using browser features and the `preload` API,
caching web resources, and automatic font fallback.
  
## Preloading of resources for web targets

The web resources like fonts and images are loaded asynchronously using the [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API).
During the initial load or with slower network connections, resource fetching can cause visual glitches, 
such as [FOUT](https://fonts.google.com/knowledge/glossary/fout) or displaying placeholders instead of images.

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
   ./gradlew :shared:wasmJsBrowserDistribution
```

2. Find the required resource in the generated `dist` directory and save the path.
3. Open the `wasmJsMain/resources/index.html` file and add a `<link>` tag inside the `<head>` element.
4. Set the `href` attribute to the resource path:

```html
<link rel="preload" href="./composeResources/username.shared.generated.resources/font/FiraMono-Regular.ttf" as="fetch" type="font/ttf" crossorigin/>
```

### Preload resources using the Compose Multiplatform preload API
<primary-label ref="Experimental"/>

Even if you preloaded the resources in the browser, they are cached as raw bytes that still need to be converted into a
format suitable for rendering, such as `FontResource` and `DrawableResource`. When the application requests the resource
for the first time, the conversion is done asynchronously, which may again result in flickering. To further optimize the experience,
Compose Multiplatform resources have their own internal cache for higher-level representations of the resources, which can also be preloaded.

Compose Multiplatform 1.8.0 introduced an experimental API for preloading font and image resources on
web targets: `preloadFont()`, `preloadImageBitmap()`, and `preloadImageVector()`.

Additionally, you can use fallback fonts different from the default bundled option if you require special characters like emojis.
When unresolved characters are encountered during rendering, a fallback font containing the missing characters is [downloaded automatically](#automatic-font-fallback).
To specify a fallback font manually, use the `FontFamily.Resolver.preload()` method.
Web targets support TTF, OTF, TTC, variable, and WOFF/WOFF2 font formats.

The following example demonstrates how to use preloading of a vector image:

```kotlin
@OptIn(ExperimentalComposeUiApi::class, ExperimentalResourceApi::class)
@Composable
fun App() {
    val icon by preloadImageVector(Res.drawable.heavy_vector_icon)

    if (icon != null) {
        MainScreen()
    } else {
        Box(modifier = Modifier.fillMaxSize()) {
            CircularProgressIndicator(modifier = Modifier.align(Alignment.Center))
        }
    }
}

@Composable
fun MainScreen() {
    // The icon is loaded from the cache
    Image(painter = painterResource(Res.drawable.heavy_vector_icon), contentDescription = null)
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="val icon by preloadImageVector(Res.drawable.heavy_vector_icon)"}

## Automatic font fallback
<primary-label ref="Experimental"/>

By default, characters not covered by the application's loaded fonts are displayed as replacement glyphs 
(□, known as "[tofu](https://fonts.google.com/knowledge/glossary/tofu)").

[//]: # (TODO update version for stable release)

Starting with version 1.12.0-beta01, Compose Multiplatform monitors unresolved
characters during rendering and downloads the required Noto font subsets on demand.
The name Noto is short for "no tofu", as these fonts are designed to eliminate tofu glyphs.

Once the fonts are available, the affected text is recomposed. 
Note that tofu may briefly appear during the download.

For CJK (Chinese, Japanese, and Korean) characters, the correct font variant is selected automatically based on the browser's language setting.

## Caching web resources
<primary-label ref="Experimental"/>

Compose Multiplatform uses the [Web Cache API](https://developer.mozilla.org/en-US/docs/Web/API/Cache) to cache successful responses
and avoid redundant HTTP revalidations typically performed by the browser's default caching mechanism.

The cache is cleared globally on every app launch and page refresh.
Resetting the cache at this stage ensures resource consistency,
as reusing the cache across multiple sessions could lead to outdated or incompatible resources
that may result in application crashes or logical inconsistencies.

To prevent redundant concurrent fetches for the same resource, the implementation uses resource-specific locks.
Each request is guarded by a per-resource mutex,
allowing parallel requests for different resources while serializing duplicate requests for the same path.
This design minimizes unnecessary network traffic and eliminates race conditions during cache population.

## What's next?

* Read more about [setting up resources](compose-multiplatform-resources-setup.md) and [using them in your app](compose-multiplatform-resources-usage.md).
* Learn how to manage the application's [resource environment](compose-resource-environment.md) like in-app theme and language.