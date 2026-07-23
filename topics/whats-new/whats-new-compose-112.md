[//]: # (title: What's new in Compose Multiplatform %org.jetbrains.compose-eap%)

Here are the highlights for this EAP release:

 * [Automatic font fallback for web](#automatic-font-fallback)
 * [MCP server for AI agents in Compose Hot Reload](#mcp-server-for-ai-agents-in-compose-hot-reload)
 * [New window and dialog API for desktop](#new-window-and-dialog-api)

You can find the full list of changes for this release on [GitHub](https://github.com/JetBrains/compose-multiplatform/releases/tag/v1.12.0-beta01).
For details about specific component versions, refer to the [Dependencies](#dependencies) section.

## Across platforms

### Skia updated to Milestone 150

The version of Skia used by Compose Multiplatform, via Skiko, has been updated to Milestone 150.

The previous version, used in Compose Multiplatform 1.11, was Milestone 144.
You can see the changes made between these versions in the [release notes](https://skia.googlesource.com/skia/+/refs/heads/chrome/m150/RELEASE_NOTES.md).

This update also resolves duplicate symbol conflicts on iOS for apps that already bundle their own Skia library 
(for example, Chromium-based apps).

## iOS

### Improved lazy layout scrolling performance

Compose Multiplatform for iOS now offers improved scrolling performance for lazy layouts. 
List item deactivations are executed outside the drawing phase, allowing the drawing phase to complete faster 
and resulting in smoother scrolling.

## Web

### Automatic font fallback
<primary-label ref="Experimental"/>

Previously, characters not covered by the application's loaded fonts were displayed as replacement glyphs (□, known as "tofu").

Compose Multiplatform for Web now automatically downloads the required Noto font subsets 
on demand when unresolved characters are encountered during rendering. 
After the fonts are downloaded, Compose recomposes the affected text. 
Note that tofu may briefly appear until the required font is fetched.

## Desktop

### MCP server for AI agents in Compose Hot Reload
<primary-label ref="Experimental"/>

Compose Hot Reload now ships with an experimental [Model Context Protocol (MCP)](https://modelcontextprotocol.io/)
server that lets AI coding agents interact directly with your running Compose application.

Until now, when an AI agent edited Compose code, there was no reliable way to verify the result: the agent couldn't
confirm that the hot reload succeeded, couldn't see the rendered UI, and couldn't read runtime logs or
exceptions. With the MCP server, agents can trigger reloads, take screenshots, inspect the
semantic tree, simulate clicks and input, and read application logs without requiring your manual intervention.

For the full list of MCP tools available to the AI agent and how to connect it, see
[MCP server for AI agents](compose-hot-reload.md#mcp-server-for-ai-agents).

### New window and dialog API
<primary-label ref="Experimental"/>

We've introduced a new experimental API for `WindowState` and `DialogState` on desktop, 
addressing several limitations of the existing API.
It lives alongside the current API in the `androidx.compose.ui.window.v2` subpackage, 
so you can adopt it gradually.

The new API gives you more control over how windows and dialogs are placed and sized: 
you can select the screen they appear on, 
provide custom positioning and sizing logic (including logic based on the content's intrinsic size), 
set minimum and maximum window sizes, and position dialogs relative to their parent window. 
It also makes the asynchronous nature of window state changes explicit, 
cleanly separating requested state from actual state.

For example, to open a window centered on the screen at a fixed size:

```kotlin
import androidx.compose.ui.window.v2.*
...

@OptIn(ExperimentalComposeUiApi::class)
fun main() = application {
    val windowState = rememberWindowState(
        initialBoundsProvider = WindowBoundsProvider(
            positionProvider = 
                WindowPositionProvider.AlignedToScreen(Alignment.Center),
            sizeProvider = 
                WindowSizeProvider.Fixed(DpSize(400.dp, 200.dp))
        )
    )

    Window(
        onCloseRequest = ::exitApplication,
        state = windowState,
    ) {
        Text("Hello, World!", fontSize = 48.sp)
    }
}
```

The redesigned API also unlocks scenarios that weren't possible before, such as sizing a window to its 
content's preferred size while still letting the content expand (via modifiers like `fillMaxSize()`) when the window is larger.
See the [dedicated documentation](compose-desktop-top-level-windows-management.md#new-window-and-dialog-api) for details.

## Dependencies

| Library            | Maven coordinates                                                      | Based on Jetpack version                                                                                                           |
|--------------------|------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------|
| Runtime            | `org.jetbrains.compose.runtime:runtime*:1.12.0-beta02`                 | [Runtime 1.12.0-beta02](https://developer.android.com/jetpack/androidx/releases/compose-runtime#1.12.0-beta02)                     | 
| UI                 | `org.jetbrains.compose.ui:ui*:1.12.0-beta02`                           | [UI 1.12.0-beta02](https://developer.android.com/jetpack/androidx/releases/compose-ui#1.12.0-beta02)                               | 
| Foundation         | `org.jetbrains.compose.foundation:foundation*:1.12.0-beta02`           | [Foundation 1.12.0-beta02](https://developer.android.com/jetpack/androidx/releases/compose-foundation#1.12.0-beta02)               |
| Material           | `org.jetbrains.compose.material:material*:1.12.0-beta02`               | [Material 1.12.0-beta02](https://developer.android.com/jetpack/androidx/releases/compose-material#1.12.0-beta02)                   | 
| Material3          | `org.jetbrains.compose.material3:material3*:1.12.0-alpha03`            | [Material3 1.5.0-alpha22](https://developer.android.com/jetpack/androidx/releases/compose-material3#1.5.0-alpha22)                 |
| Material3 Adaptive | `org.jetbrains.compose.material3.adaptive:adaptive*:1.3.0-beta02`      | [Material3 Adaptive 1.3.0-beta02](https://developer.android.com/jetpack/androidx/releases/compose-material3-adaptive#1.3.0-beta02) |
| Lifecycle          | `org.jetbrains.androidx.lifecycle:lifecycle-*:2.11.0`                  | [Lifecycle 2.11.0](https://developer.android.com/jetpack/androidx/releases/lifecycle#2.11.0)                                       | 
| Navigation         | `org.jetbrains.androidx.navigation:navigation-*:2.10.0-alpha02`        | [Navigation 2.10.0-alpha05](https://developer.android.com/jetpack/androidx/releases/navigation#2.10.0-alpha05)                     |
| Navigation3        | `org.jetbrains.androidx.navigation3:navigation3-*:1.2.0-alpha02`       | [Navigation3 1.2.0-alpha04](https://developer.android.com/jetpack/androidx/releases/navigation3#1.2.0-alpha04)                     |
| Navigation Event   | `org.jetbrains.androidx.navigationevent:navigationevent-compose:1.1.0` | [Navigation Event 1.1.1](https://developer.android.com/jetpack/androidx/releases/navigationevent#1.1.1)                            |
| Savedstate         | `org.jetbrains.androidx.savedstate:savedstate*:1.4.0`                  | [Savedstate 1.4.0](https://developer.android.com/jetpack/androidx/releases/savedstate#1.4.0)                                       |
| WindowManager Core | `org.jetbrains.androidx.window:window-core:1.5.1`                      | [WindowManager 1.5.1](https://developer.android.com/jetpack/androidx/releases/window#1.5.1)                                        |