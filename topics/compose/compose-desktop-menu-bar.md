[//]: # (title: Menu bar)
[//]: # (description: Learn how to create a menu bar for a specific window in Compose Multiplatform for desktop.)

You can create a menu bar for a specific window using the `MenuBar` composable. `MenuBar` is available in the scope 
of the `Window` composable, so each window can have its own menu bar.

<include from="compose-desktop-scrollbars.md" element-id="desktop-snippets-intro"/>

You can use the following components in `MenuBar`:

* `Menu` – a menu or a submenu
* `Item` – a clickable menu item
* `CheckboxItem` – an item with a checkbox
* `RadioButtonItem` – an item with a radio button
* `Separator` – a horizontal line that groups items

Items and menus accept a `mnemonic` parameter, a character that opens the menu or triggers the item when pressed
together with <shortcut>Alt</shortcut>. If the character occurs in the text, its first occurrence is underlined.
Items also accept a `shortcut` parameter – a `KeyShortcut` that triggers the action without navigating the menu.

> Setting `ctrl = true` in `KeyShortcut` always maps to <shortcut>Ctrl</shortcut>, including on macOS. 
> To use <shortcut>⌘</shortcut> for the standard macOS shortcuts, set `meta = true` instead.
>
{style="tip"}

Menu content is composable, so you can use conditions and loops inside `MenuBar` to decide which items exist. When the 
state they read changes, the menu updates. In the following example, the **Settings** submenu only exists while the 
**Advanced settings** checkbox is selected:

```kotlin
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.material.Text
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.geometry.Size
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.graphics.drawscope.DrawScope
import androidx.compose.ui.graphics.painter.Painter
import androidx.compose.ui.input.key.Key
import androidx.compose.ui.input.key.KeyShortcut
import androidx.compose.ui.window.MenuBar
import androidx.compose.ui.window.Window
import androidx.compose.ui.window.application

fun main() = application {
    var action by remember { mutableStateOf("Last action: None") }
    var isOpen by remember { mutableStateOf(true) }

    if (isOpen) {
        var isSubmenuShowing by remember { mutableStateOf(false) }

        Window(onCloseRequest = { isOpen = false }) {
            MenuBar {
                Menu("File", mnemonic = 'F') {
                    Item("Copy", onClick = { action = "Last action: Copy" }, shortcut = KeyShortcut(Key.C, ctrl = true))
                    Item(
                        "Paste",
                        onClick = { action = "Last action: Paste" },
                        shortcut = KeyShortcut(Key.V, ctrl = true)
                    )
                }
                Menu("Actions", mnemonic = 'A') {
                    CheckboxItem(
                        "Advanced settings",
                        checked = isSubmenuShowing,
                        onCheckedChange = {
                            isSubmenuShowing = !isSubmenuShowing
                        }
                    )
                    if (isSubmenuShowing) {
                        Menu("Settings") {
                            Item("Setting 1", onClick = { action = "Last action: Setting 1" })
                            Item("Setting 2", onClick = { action = "Last action: Setting 2" })
                        }
                    }
                    Separator()
                    Item("About", icon = AboutIcon, onClick = { action = "Last action: About" })
                    Item("Exit", onClick = { isOpen = false }, shortcut = KeyShortcut(Key.Escape), mnemonic = 'E')
                }
            }

            Box(
                modifier = Modifier.fillMaxSize(),
                contentAlignment = Alignment.Center
            ) {
                Text(text = action)
            }
        }
    }
}

object AboutIcon : Painter() {
    override val intrinsicSize = Size(256f, 256f)

    override fun DrawScope.onDraw() {
        drawOval(Color(0xFFFFA500))
    }
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="Window(MenuBar { Menu( "}

<img src="compose-desktop-menu-bar.animated.gif" alt="Menu bar" width="600"/>

On Windows and Linux, the menu bar is part of the window. On macOS, it's displayed in the system menu bar at the top of
the screen when the window is active.

## What's next

* Learn how to add an application icon and a menu to the [system tray](compose-desktop-tray.md).
* Explore the tutorials about [other desktop components](compose-desktop-components.md).
