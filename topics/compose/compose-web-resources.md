[//]: # (title: Handling web resources)

Here you can find information about preloading resources using browser features and the `preload` API,
as well as caching web resources.
  
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
import androidx.compose.ui.window.ComposeViewport
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
    ComposeViewport(viewportContainerId = "composeApplication") {
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
