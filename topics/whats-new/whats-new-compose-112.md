[//]: # (title: What's new in Compose Multiplatform %org.jetbrains.compose-eap%)

Here are the highlights for this EAP release:

 * [Automatic font fallback for web](#automatic-font-fallback)
 * [MCP server for AI agents in Compose Hot Reload](#mcp-server-for-ai-agents-in-compose-hot-reload)
 * 

You can find the full list of changes for this release on [GitHub](https://github.com/JetBrains/compose-multiplatform/releases/tag/v1.12.0-beta01).
For details about specific component versions, refer to the [Dependencies](#dependencies) section.

## Breaking changes and deprecations

## Across platforms

### Skia updated to Milestone 150

The version of Skia used by Compose Multiplatform, via Skiko, has been updated to Milestone 150.

The previous version, used in Compose Multiplatform 1.11, was Milestone 144.
You can see the changes made between these versions in the [release notes](https://skia.googlesource.com/skia/+/refs/heads/chrome/m150/RELEASE_NOTES.md).

This update also resolves duplicate symbol conflicts on iOS for apps that already bundle their own Skia library 
(for example, Chromium-based apps).

## iOS

### Improved lazy layout scrolling performance

Compose Multiplatform for iOS now supports `OutOfFrameExecutor`, improving the scrolling performance of lazy layouts. 
With this change, list item deactivations are executed outside the drawing phase, 
allowing it to complete faster and resulting in smoother scrolling.

## Web

### Automatic font fallback
<primary-label ref="Experimental"/>

Previously, characters not covered by the application's loaded fonts were displayed as replacement glyphs (□, known as "tofu").

Compose Multiplatform for Web now automatically downloads the required Noto font subsets 
on demand when unresolved characters are encountered during rendering. 
After the fonts are downloaded, Compose recomposes the affected text. 
Note that tofu may briefly appear until the required font is fetched.

## Desktop

## MCP server for AI agents in Compose Hot Reload
<primary-label ref="Experimental"/>

Compose Hot Reload now ships with an experimental [Model Context Protocol (MCP)](https://modelcontextprotocol.io/)
server that lets AI coding agents interact directly with your running Compose application.

Until now, when an AI agent edited Compose code, there was no reliable way to verify the result: the agent couldn't
confirm that the hot reload succeeded, couldn't see the rendered UI, and couldn't read runtime logs or
exceptions. With the MCP server, agents can trigger reloads, take screenshots, inspect the
semantic tree, simulate clicks and input, and read application logs without requiring your manual intervention.

To start the MCP server alongside your application, run the `hotMcpServerJvm` Gradle task.
For the full list of MCP tools available to the AI agent and how to connect it, see
[MCP server for AI agents](compose-hot-reload.md#mcp-server-for-ai-agents).

## Gradle

## Dependencies