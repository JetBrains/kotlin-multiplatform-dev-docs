[//]: # (title: What's new in Compose Multiplatform %org.jetbrains.compose-eap%)

Here are the highlights for this EAP release:

* [Content-based sizing of Compose Multiplatform views in SwiftUI and UIKit](#content-based-sizing-for-compose-multiplatform-views-in-swiftui-and-uikit)
* [Saving and autofilling passwords on iOS](#saving-and-autofilling-passwords)
* [WebGL interop](#webgl-interop)
* [Faster desktop startup with AppCDS and AOT options](#faster-desktop-startup-with-appcds-or-aot)

You can find the full list of changes for this release on [GitHub](https://github.com/JetBrains/compose-multiplatform/releases/tag/v1.13.0-beta01).
For details about specific component versions, refer to the [Dependencies](#dependencies) section.

## Across platforms

Several features that were Experimental in previous Compose Multiplatform releases are now Stable.
You no longer need to opt in to use them:

* Creating a `ComposeUIView()` on iOS and configuring it with `ComposeUIViewportConfiguration`.
* [Configuring `ComposeView()` for web with `ComposeViewportConfiguration`](compose-css-styles.md).
* [Preloading resources for web with `preload` API](compose-web-resources.md#preload-resources-using-the-compose-multiplatform-preload-api).

## iOS

### Content-based sizing for Compose Multiplatform views in SwiftUI and UIKit

Compose Multiplatform views embedded in SwiftUI or UIKit can now report a preferred size based on the available parent space.
Native containers can use this size to fit Compose content, for example, 
to adjusting a view’s height while keeping its width fixed.

On iOS 16 and later, Compose Multiplatform views use `PreferredSizeReportingStrategy.SizeThatFits` by default,
so no additional sizing configuration is needed.

Create a Compose view on the Kotlin side:

```kotlin
val view = ComposeUIView {
    Text("Hello from Compose")
}
```

Then, in the SwiftUI `UIViewRepresentable` wrapper, pass the sizing proposal to `sizeThatFits()`.
Use `UIView.noIntrinsicMetric` for dimensions that SwiftUI leaves unspecified:

```swift
@available(iOS 16.0, *)
func sizeThatFits(
    _ proposal: ProposedViewSize,
    uiView: UIView,
    context: Context
) -> CGSize? {
    uiView.sizeThatFits(
        CGSize(
            width: proposal.width ?? UIView.noIntrinsicMetric,
            height: proposal.height ?? UIView.noIntrinsicMetric
        )
    )
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="func sizeThatFits( _ proposal: ProposedViewSize, uiView: UIView, "}

In UIKit, after Auto Layout sets the Compose Multiplatform view’s width,
use `sizeThatFits()` to get its preferred height and update its height constraint:

```swift
let preferredSize = uiView.sizeThatFits(
    CGSize(
        width: uiView.bounds.width,
        height: UIView.noIntrinsicMetric
    )
)
heightConstraint.constant = preferredSize.height
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="let preferredSize = uiView.sizeThatFits("}

### Saving and autofilling passwords
<primary-label ref="Experimental"/>

On iOS, Compose Multiplatform can now show native prompts to save login credentials and autofill Compose text fields.
Keep both the username and password fields on screen so iOS can recognize the form as a login form.

To configure the fields, use `PlatformImeOptions` in your iOS source set.
Set `textContentType(UITextContentTypeUsername)` for the username and `isSecureTextEntry(true)` for the password.
For example, inside a composable annotated with `@OptIn(ExperimentalComposeUiApi::class)`:

```kotlin
var username by remember { mutableStateOf("") }
var password by remember { mutableStateOf("") }

Column {
    TextField(
        value = username,
        onValueChange = { username = it },
        keyboardOptions = KeyboardOptions(
            keyboardType = KeyboardType.Email,
            platformImeOptions = PlatformImeOptions {
                textContentType(UITextContentTypeUsername)
            }
        )
    )

    TextField(
        value = password,
        onValueChange = { password = it },
        visualTransformation = PasswordVisualTransformation(),
        keyboardOptions = KeyboardOptions(
            keyboardType = KeyboardType.Password,
            platformImeOptions = PlatformImeOptions {
                isSecureTextEntry(true)
            }
        )
    )
}
```

## Web

### WebGL interop
<primary-label ref="Experimental"/>

Compose Multiplatform now provides an experimental API for rendering [WebGL](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API) textures.
`WebGLRenderTarget` exposes a Compose `Painter` that displays WebGL graphics alongside other Compose UI elements.

Use `rememberWebGLRenderTarget()` to create a target, draw into it with `render {}`, and display its `painter` in an `Image`.
For example, this fragment renders a blue texture inside a composable with `@OptIn(ExperimentalComposeUiApi::class)`:

```kotlin
val target = rememberWebGLRenderTarget(IntSize(64, 64))

if (target != null) {
    LaunchedEffect(target) {
        target.render {
            with(target.webGLContext) {
                disable(WebGLRenderingContext.SCISSOR_TEST)
                clearColor(0.15f, 0.55f, 1f, 1f)
                clear(WebGLRenderingContext.COLOR_BUFFER_BIT)
            }
        }
    }

    Image(
        painter = target.painter,
        contentDescription = "A blue WebGL texture",
        modifier = Modifier.size(200.dp),
    )
}
```

For more advanced use cases, the `target` also provides access to the underlying WebGL rendering context.

### System window insets on web
<primary-label ref="Experimental"/>

Compose Multiplatform for web now supports system window insets through the standard `WindowInsets` APIs.
Your UI can draw edge-to-edge while respecting safe areas, including display cutouts and the on-screen keyboard.

To enable this feature, set `enableBrowserWindowInsets` to `true` in your `ComposeViewport` configuration.
Use the standard inset modifiers to adjust the layout:

```kotlin
ComposeViewport(configure = { enableBrowserWindowInsets = true }) {
    Box(Modifier.fillMaxSize().safeDrawingPadding()) {
        Text("Content within the safe drawing area")
    }
}
```

You can also access the insets directly through APIs such as `WindowInsets.safeDrawing` and `WindowInsets.ime`.

For edge-to-edge rendering, add `viewport-fit=cover` to the page's viewport `meta` tag:

```html
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover" />
```

> Without `viewport-fit=cover`, the browser applies safe-area padding and reports zero safe-area insets.
> Compose Multiplatform will log a warning if browser window insets are enabled without this setting.

### Configurable font fallback on web

In the previous release, we introduced [automatic font fallback](compose-web-resources.md#automatic-font-fallback)
for characters not covered by your application's fonts.
This release lets you choose the source for fallback fonts or disable automatic downloads.

By default, Compose Multiplatform downloads fallback fonts from [Google Fonts](https://fonts.gstatic.com/s/).
To use a different source, set `fontFallbackUrl` in the window configuration:

```kotlin
createComposeWindow(
    configure = { fontFallbackUrl = "https://custom.fonts.com" }
) {
    BasicText("გამარჯობა")
}
```

To disable automatic font fallback, set `fontFallbackUrl` to `null`:

```kotlin
createComposeWindow(
    configure = { fontFallbackUrl = null }
) {
    BasicText("გამარჯობა")
}
```

## Desktop

### Faster desktop startup with AppCDS or AOT

Compose Multiplatform now supports [Application Class-Data Sharing (AppCDS)](https://openjdk.org/jeps/310) and 
[Ahead-of-Time (AOT) caching](https://openjdk.org/jeps/514)
for desktop applications. Both options use an archive to reduce startup time.

In benchmarks on a small demo application, prebuilt AOT caches reduced cold startup time:

* On macOS (M1 Ultra), from 2017 ms to 424 ms — 4.8× faster.
* On Windows (Dell Precision 5690), from 1356 ms to 650 ms — 2.1× faster.

These measurements cover the time from launching the process to displaying the window.
Results depend on the application and environment. 
For AOT startup optimization, we recommend the latest [JetBrains Runtime (JBR) 25](https://github.com/JetBrains/JetBrainsRuntime) release.

To enable these startup options, configure the `aot {}` block in your `build.gradle.kts` file:

```kotlin
compose.desktop {
    application {
        aot {
            mode = AotMode.AotPrebuild
        }
    }
}
```

Choose a mode based on your JDK version and packaging requirements:

| Mode             | Minimum JDK | Behavior                                                                                       |
|------------------|-------------|------------------------------------------------------------------------------------------------|
| `None`           | —           | Default, no startup optimizations.                                                             |
| `AppCdsAuto`     | 19          | Creates an AppCDS archive when the user's first session ends, speeding up subsequent launches. |
| `AppCdsPrebuild` | 21          | Creates an AppCDS archive before packaging, speeding up startup from the first launch.         |
| `AotPrebuild`    | 25          | Creates an AOT cache before packaging, also improving linking and JIT warmup.                  |

Both prebuild modes run your application during packaging to create the archive.
During this cache-generation run, the plugin sets the `compose.aot.training-run` system property to `true`.

Note that the prebuild modes increase the distributable size because the application includes the archives.

### Smoother window resizing on macOS and Windows

Compose Multiplatform now supports smoother interactive window resizing when using Metal on macOS or DirectX on Windows.
Rendering stays synchronized with window resizing, reducing stutter and preventing white bars along the window edges.

To enable this behavior, set the system property for your renderer to `true` before showing any windows.
Set it with `System.setProperty()` or a JVM `-D` flag:

* macOS with Metal: `skiko.rendering.macos.metalSynchronousLiveResize`
* Windows with DirectX: `skiko.rendering.windows.direct3DSynchronousLiveResize`

The feature requires Compose content to fill the entire window.

## Dependencies

| Library            | Maven coordinates                                                      | Based on Jetpack version                                                                                                             |
|--------------------|------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------|
| Runtime            | `org.jetbrains.compose.runtime:runtime*:1.13.0-beta01`                 | [Runtime 1.13.0-beta01](https://developer.android.com/jetpack/androidx/releases/compose-runtime#1.13.0-beta01)                       |
| UI                 | `org.jetbrains.compose.ui:ui*:1.13.0-beta01`                           | [UI 1.13.0-beta01](https://developer.android.com/jetpack/androidx/releases/compose-ui#1.13.0-beta01)                                 |
| Foundation         | `org.jetbrains.compose.foundation:foundation*:1.13.0-beta01`           | [Foundation 1.13.0-beta01](https://developer.android.com/jetpack/androidx/releases/compose-foundation#1.13.0-beta01)                 |
| Material           | `org.jetbrains.compose.material:material*:1.13.0-beta01`               | [Material 1.13.0-beta01](https://developer.android.com/jetpack/androidx/releases/compose-material#1.13.0-beta01)                     |
| Material3          | `org.jetbrains.compose.material3:material3*:1.13.0-beta01`             | [Material3 1.5.0-beta01](https://developer.android.com/jetpack/androidx/releases/compose-material3#1.5.0-beta01)                     |
| Material3 Adaptive | `org.jetbrains.compose.material3.adaptive:adaptive*:1.4.0-alpha01`     | [Material3 Adaptive 1.4.0-alpha03](https://developer.android.com/jetpack/androidx/releases/compose-material3-adaptive#1.4.0-alpha03) |
| Lifecycle          | `org.jetbrains.androidx.lifecycle:lifecycle-*:2.11.0`                  | [Lifecycle 2.11.0](https://developer.android.com/jetpack/androidx/releases/lifecycle#2.11.0)                                         |
| Navigation         | `org.jetbrains.androidx.navigation:navigation-*:2.10.0`                | [Navigation 2.10.0](https://developer.android.com/jetpack/androidx/releases/navigation#2.10.0)                                       |
| Navigation3        | `org.jetbrains.androidx.navigation3:navigation3-*:1.2.0`               | [Navigation3 1.2.0](https://developer.android.com/jetpack/androidx/releases/navigation3#1.2.0)                                       |
| Navigation Event   | `org.jetbrains.androidx.navigationevent:navigationevent-compose:1.1.0` | [Navigation Event 1.1.1](https://developer.android.com/jetpack/androidx/releases/navigationevent#1.1.1)                              |
| Savedstate         | `org.jetbrains.androidx.savedstate:savedstate*:1.4.0`                  | [Savedstate 1.4.0](https://developer.android.com/jetpack/androidx/releases/savedstate#1.4.0)                                         |
| WindowManager Core | `org.jetbrains.androidx.window:window-core:1.5.1`                      | [WindowManager 1.5.1](https://developer.android.com/jetpack/androidx/releases/window#1.5.1)                                          |