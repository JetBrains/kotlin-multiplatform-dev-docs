[//]: # (title: Setting up the viewport)
<show-structure for="none"/>

Compose Multiplatform for web uses the `ComposeViewport` function to render your UI onto an HTML canvas.
It does not inject global CSS styles, 
giving you full control over how your application integrates with the HTML structure.

To correctly fit the content within the browser window, apply explicit CSS to the host container.
If no CSS is specified, the canvas may not resize correctly or fill the intended space.

Here is a standard `styles.css` example to make the content fill the entire screen:

```css
html, body {
    width: 100%;
    height: 100%;
    margin: 0;
    padding: 0;
    overflow: hidden;
}
```

Then, initialize the entry point in the `main` function of your web source set:

```kotlin
@OptIn(ExperimentalComposeUiApi::class)
fun main() {
    ComposeViewport(viewportContainerId = "composeApp") {
        App()
    }
}
```

> The previously used `CanvasBasedWindow` is now deprecated. It automatically inserted CSS styles directly into the page's HTML elements
> to force the canvas to fill the browser window. 
> While simpler for standalone apps, this approach made it difficult to embed Compose into existing web layouts. 
> `ComposeViewport` is a more flexible approach that relies on standard CSS-based layout management.

## What's next?

* Learn how to [handle web-specific resources](compose-web-resources.md).
* Read more about [Kotlin/Wasm and Compose Multiplatform](https://kotlinlang.org/docs/wasm-get-started.html).