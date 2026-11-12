[//]: # (title: WebGL interoperability)
<primary-label ref="EAP"/>

[//]: # (TODO After release change label to Experimental) 

<tldr>
    You can include WebGL graphics in the Compose UI of your web app. Draw into a texture using the WebGL context that Compose provides,
    then display it alongside other UI elements.
</tldr>

[WebGL](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API) is a browser API for rendering
hardware-accelerated 2D and 3D graphics. In a Compose Multiplatform app, you can use it for:

* [Custom graphics](#render-a-webgl-texture): draw shapes, animations, or shader effects.
* [JavaScript renderers](#integrate-a-javascript-webgl-renderer): integrate existing JavaScript WebGL renderers to display 
3D scenes or animated characters.

Other use cases include displaying video frames with Compose controls and overlays, 
or experimenting with HTML content as a texture through the [HTML-in-Canvas](https://developer.chrome.com/docs/web-platform/html-in-canvas)
API where the browser supports it.

## Render a WebGL texture

A `WebGLRenderTarget` lets you draw into a texture with WebGL and display the result in Compose through a `Painter`.
To render a WebGL texture:

1. Create a target with `rememberWebGLRenderTarget()`. Specify the texture size using an `IntSize` in pixels 
   or a `DpSize` in density-independent pixels.
2. Draw using the target's `webGLContext` inside a `target.render()` block.
3. Pass `target.painter` to an `Image()` composable to display the result.

The following example updates a texture on each animation frame to produce a pulsing color
and displays it inside an `Image()` composable. Add it to the `wasmJsMain` source set and call `ColorPulse()` from your UI:

```kotlin
import androidx.compose.foundation.Image
import androidx.compose.foundation.layout.size
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.LaunchedEffect
import androidx.compose.runtime.withFrameNanos
import androidx.compose.ui.ExperimentalComposeUiApi
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.clip
import androidx.compose.ui.platform.webgl.rememberWebGLRenderTarget
import androidx.compose.ui.unit.IntSize
import androidx.compose.ui.unit.dp
import kotlin.math.sin
import org.khronos.webgl.WebGLRenderingContext

@OptIn(ExperimentalComposeUiApi::class)
@Composable
fun ColorPulse() {
    val target = rememberWebGLRenderTarget(IntSize(64, 64))

    if (target == null) {
        Text("WebGL rendering is unavailable")
        return
    }

    LaunchedEffect(target) {
        var phase = 0f
        var previousFrameTimeNanos = 0L

        while (true) {
            withFrameNanos { frameTimeNanos ->
                target.render {
                    val deltaNanos = if (previousFrameTimeNanos == 0L) {
                        0L
                    } else {
                        frameTimeNanos - previousFrameTimeNanos
                    }
                    previousFrameTimeNanos = frameTimeNanos
                    phase += deltaNanos / 1_000_000_000f * 1.5f
                    val level = sin(phase) * 0.5f + 0.5f

                    with(target.webGLContext) {
                        // Fills the texture with the pulse color
                        disable(WebGLRenderingContext.SCISSOR_TEST)
                        clearColor(0.15f * level, 0.55f * level, level, level)
                        clear(WebGLRenderingContext.COLOR_BUFFER_BIT)
                    }
                }
            }
        }
    }

    Image(
        painter = target.painter,
        contentDescription = "Pulsing blue WebGL texture",
        modifier = Modifier.size(200.dp).clip(RoundedCornerShape(16.dp))
    )
}
```

## Integrate a JavaScript WebGL renderer

You can use an existing JavaScript WebGL renderer, such as [three.js](https://threejs.org/), to draw content for your Compose UI.
The renderer shares the WebGL context with Compose and draws into the target's framebuffer.
To use the renderer with a `WebGLRenderTarget`:

1. Define Kotlin bindings for the library using
   [Kotlin/Wasm–JavaScript interoperability](https://kotlinlang.org/docs/wasm-js-interop.html).
2. Initialize the renderer with the target's `htmlCanvas` and `webGLContext`.
   Let Compose manage the canvas size and context lifetime.
3. Draw into `target.framebuffer` inside a `target.render()` block.
   If the renderer caches graphics state, reset that cache before drawing.
   For example, three.js provides `renderer.resetState()` for this purpose.
4. Use `DisposableEffect()` to release the renderer's resources when its composable leaves the composition.
   If cleanup modifies the shared WebGL context, call `target.markGLStateStale()` afterward.

The following example draws a frame using an initialized three.js renderer.
Here, `target` is a `WebGLRenderTarget`; `renderer`, `scene`, and `camera` are your three.js objects; 
`threeTarget` is the render target for three.js, and its dimensions match `target.size`.

```kotlin
target.render {
    renderer.resetState()
    renderer.setRenderTargetFramebuffer(threeTarget, target.framebuffer)
    renderer.setRenderTarget(threeTarget)
    try {
        renderer.render(scene, camera)
    } finally {
        // Restores the default framebuffer for Compose
        renderer.setRenderTarget(null)
    }
}
```