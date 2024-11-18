[//]: # (title: Context menus)

Compose Multiplatform for desktop provides out-of-the-box support for text context menus and allows you to
conveniently tailor your context menus by adding more items, setting up themes, and customizing text.

## Default context menu

Compose Multiplatform for desktop offers built-in context menus for `TextField` and selectable `Text`.

The default context menu for a text field includes the following actions, depending on the cursor's position and the 
selection range: Copy, Cut, Paste, and Select All.
This standard context menu is available by default in the material `TextField` (`androidx.compose.material.TextField`
or `androidx.compose.material3.TextField`) and the foundation `BasicTextField` (`androidx.compose.foundation.text.BasicTextField`).

```kotlin
import androidx.compose.material.Text
import androidx.compose.material.TextField
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.ui.window.singleWindowApplication

fun main() = singleWindowApplication(title = "Context menu") {
    val text = remember { mutableStateOf("Hello!") }
    TextField(
        value = text.value,
        onValueChange = { text.value = it },
        label = { Text(text = "Input") }
    )
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="TextField( value = text.value, onValueChange ="}

<img src="compose-desktop-context-menu-textfield.png" alt="Default context menu for TextField" width="500"/>

The default context menu for a simple text element includes only the Copy action.
To enable context menu for a `Text` component, make the text selectable by wrapping it in a `SelectionContainer`:

```kotlin
import androidx.compose.foundation.text.selection.SelectionContainer
import androidx.compose.material.Text
import androidx.compose.ui.window.singleWindowApplication

fun main() = singleWindowApplication(title = "Context menu") {
    SelectionContainer {
        Text("Hello World!")
    }
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="SelectionContainer { Text( "}

<img src="compose-desktop-context-menu-text.png" alt="Default context menu for Text" width="500"/>

## Context menu with custom items

To add custom context menu actions for the `TextField` and `Text` components, specify new items via `ContextMenuItem` and
add them to the hierarchy of context menu items via `ContextMenuDataProvider`. For example, the following code sample 
demonstrates the addition of two new custom actions to the default context menus of a text field and a simple 
selectable text element:

```kotlin
import androidx.compose.foundation.ContextMenuDataProvider
import androidx.compose.foundation.ContextMenuItem
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.text.selection.SelectionContainer
import androidx.compose.material.Text
import androidx.compose.material.TextField
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import androidx.compose.ui.window.singleWindowApplication

fun main() = singleWindowApplication(title = "Context menu") {
    val text = remember { mutableStateOf("Hello!") }
    Column {
        ContextMenuDataProvider(
            items = {
                listOf(
                    ContextMenuItem("User-defined action") {
                        // Some custom action
                    },
                    ContextMenuItem("Another user-defined action") {
                        // Another custom action
                    }
                )
            }
        ) {
            TextField(
                value = text.value,
                onValueChange = { text.value = it },
                label = { Text(text = "Input") }
            )

            Spacer(Modifier.height(16.dp))

            SelectionContainer {
                Text("Hello World!")
            }
        }
    }
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="ContextMenuDataProvider( items = { listOf( ContextMenuItem( "}

<img src="compose-desktop-context-menu-custom-actions.png" alt="Context menu with custom actions" width="500"/>

If you want to override the text menu for all text fields and selectable texts in your application, refer to the section about [`TextContextMenu`](#override-default-context-menu).

## Context menu in a custom area

You can create a context menu for any arbitrary area of your application. Use `ContextMenuArea` to define a container where
the right mouse click will trigger the appearance of a context menu:

```kotlin
import androidx.compose.foundation.ContextMenuArea
import androidx.compose.foundation.ContextMenuItem
import androidx.compose.foundation.background
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.width
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.unit.dp
import androidx.compose.ui.window.singleWindowApplication

fun main() = singleWindowApplication(title = "Context menu") {
    ContextMenuArea(items = {
        listOf(
            ContextMenuItem("User-defined action") {
                // Some custom action
            },
            ContextMenuItem("Another user-defined action") {
                // Another custom action
            }
        )
    }) {
        // Blue box where context menu will be available
        Box(modifier = Modifier.background(Color.Blue).height(100.dp).width(100.dp))
    }
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="ContextMenuArea(items = { listOf( ContextMenuItem( "}

<img src="compose-desktop-context-menu-custom-area.png" alt="Context menu: ContextMenuArea" width="500"/>

## Set up theming

You can customize context menu colors to create a responsive UI that matches the system settings and avoid harsh contrast 
changes when switching between applications. For default light and dark themes, there are two built-in implementations:
`LightDefaultContextMenuRepresentation` and `DarkDefaultContextMenuRepresentation`.
They are not applied to context menu colors automatically, so you need to set a suitable theme via `LocalContextMenuRepresentation`:

```kotlin
import androidx.compose.foundation.DarkDefaultContextMenuRepresentation
import androidx.compose.foundation.LightDefaultContextMenuRepresentation
import androidx.compose.foundation.LocalContextMenuRepresentation
import androidx.compose.foundation.isSystemInDarkTheme
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.material.MaterialTheme
import androidx.compose.material.Surface
import androidx.compose.material.TextField
import androidx.compose.material.darkColors
import androidx.compose.material.lightColors
import androidx.compose.runtime.CompositionLocalProvider
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Modifier
import androidx.compose.ui.window.singleWindowApplication

fun main() = singleWindowApplication(title = "Dark theme") {
    MaterialTheme(
        colors = if (isSystemInDarkTheme()) darkColors() else lightColors()
    ) {
        val contextMenuRepresentation = if (isSystemInDarkTheme()) {
            DarkDefaultContextMenuRepresentation
        } else {
            LightDefaultContextMenuRepresentation
        }
        CompositionLocalProvider(LocalContextMenuRepresentation provides contextMenuRepresentation) {
            Surface(Modifier.fillMaxSize()) {
                Box {
                    var value by remember { mutableStateOf("") }
                    TextField(value, { value = it })
                }
            }
        }
    }
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="MaterialTheme( colors = if (isSystemInDarkTheme()) darkColors() else "}

<img src="compose-desktop-context-menu-dark-mode.png" alt="Context menu: Dark theme" width="500"/>

## Override default context menu

To override the default context menu for all text fields and selectable text elements, override the `TextContextMenu` interface.
In the following code sample, we reuse the original `TextContextMenu`, but add one additional item to the bottom of the list.
The new item adjusts to the text selection:

```kotlin
import androidx.compose.foundation.ContextMenuDataProvider
import androidx.compose.foundation.ContextMenuItem
import androidx.compose.foundation.ContextMenuState
import androidx.compose.foundation.ExperimentalFoundationApi
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.text.LocalTextContextMenu
import androidx.compose.foundation.text.TextContextMenu
import androidx.compose.foundation.text.selection.SelectionContainer
import androidx.compose.material.Text
import androidx.compose.material.TextField
import androidx.compose.runtime.Composable
import androidx.compose.runtime.CompositionLocalProvider
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.platform.LocalUriHandler
import androidx.compose.ui.text.AnnotatedString
import androidx.compose.ui.window.singleWindowApplication
import java.net.URLEncoder
import java.nio.charset.Charset

fun main() = singleWindowApplication(title = "Context menu") {
    CustomTextMenuProvider {
        Column {
            SelectionContainer {
                Text("Hello, Compose!")
            }
            var text by remember { mutableStateOf("") }
            TextField(text, { text = it })
        }
    }
}

@OptIn(ExperimentalFoundationApi::class)
@Composable
fun CustomTextMenuProvider(content: @Composable () -> Unit) {
    val textMenu = LocalTextContextMenu.current
    val uriHandler = LocalUriHandler.current
    CompositionLocalProvider(
        LocalTextContextMenu provides object : TextContextMenu {
            @Composable
            override fun Area(
                textManager: TextContextMenu.TextManager,
                state: ContextMenuState,
                content: @Composable () -> Unit
            ) {
                // Reuse original TextContextMenu and add a new item
                ContextMenuDataProvider({
                    val shortText = textManager.selectedText.crop()
                    if (shortText.isNotEmpty()) {
                        val encoded = URLEncoder.encode(shortText, Charset.defaultCharset())
                        listOf(ContextMenuItem("Search $shortText") {
                            uriHandler.openUri("https://google.com/search?q=$encoded")
                        })
                    } else {
                        emptyList()
                    }
                }) {
                    textMenu.Area(textManager, state, content = content)
                }
            }
        },
        content = content
    )
}

private fun AnnotatedString.crop() = if (length <= 5) toString() else "${take(5)}..."
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="CompositionLocalProvider( LocalTextContextMenu provides object : TextContextMenu { "}

<img src="compose-desktop-context-menu-custom-text.png" alt="Context menu: LocalTextContextMenu" width="500"/>

## Localized menu items

By default, the context menu will appear in the preferred language of your system settings:

<img src="compose-desktop-context-menu-localization.png" alt="Context menu: Localization" width="500"/>

If you want to use a specific language, specify it as a default language explicitly before running your application: 

```Console
java.util.Locale.setDefault(java.util.Locale("en"))
```

## Swing interoperability

If you are embedding Compose code into an existing Swing application and need the context menu to match the appearance and 
behavior of other parts of the application, you can use the `JPopupTextMenu` class. In this class, `LocalTextContextMenu` 
uses Swing's `JPopupMenu` for context menus in Compose components.

```kotlin
import androidx.compose.foundation.ExperimentalFoundationApi
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.text.JPopupTextMenu
import androidx.compose.foundation.text.LocalTextContextMenu
import androidx.compose.foundation.text.selection.SelectionContainer
import androidx.compose.material.Text
import androidx.compose.material.TextField
import androidx.compose.runtime.Composable
import androidx.compose.runtime.CompositionLocalProvider
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.awt.ComposePanel
import androidx.compose.ui.platform.LocalLocalization
import java.awt.Color
import java.awt.Component
import java.awt.Dimension
import java.awt.Graphics
import java.awt.event.KeyEvent
import java.awt.event.KeyEvent.CTRL_DOWN_MASK
import java.awt.event.KeyEvent.META_DOWN_MASK
import javax.swing.Icon
import javax.swing.JFrame
import javax.swing.JMenuItem
import javax.swing.JPopupMenu
import javax.swing.KeyStroke.getKeyStroke
import javax.swing.SwingUtilities
import org.jetbrains.skiko.hostOs

fun main() = SwingUtilities.invokeLater {
    val panel = ComposePanel()
    panel.setContent {
        JPopupTextMenuProvider(panel) {
            Column {
                SelectionContainer {
                    Text("Hello, World!")
                }

                var text by remember { mutableStateOf("") }

                TextField(text, { text = it })
            }
        }
    }

    val window = JFrame()
    window.contentPane.add(panel)
    window.size = Dimension(800, 600)
    window.isVisible = true
    window.title = "Swing interop"
}

@OptIn(ExperimentalFoundationApi::class)
@Composable
fun JPopupTextMenuProvider(owner: Component, content: @Composable () -> Unit) {
    val localization = LocalLocalization.current
    CompositionLocalProvider(
        LocalTextContextMenu provides JPopupTextMenu(owner) { textManager, items ->
            JPopupMenu().apply {
                textManager.cut?.also {
                    add(
                        swingItem(localization.cut, Color.RED, KeyEvent.VK_X, it)
                    )
                }
                textManager.copy?.also {
                    add(
                        swingItem(localization.copy, Color.GREEN, KeyEvent.VK_C, it)
                    )
                }
                textManager.paste?.also {
                    add(
                        swingItem(localization.paste, Color.BLUE, KeyEvent.VK_V, it)
                    )
                }
                textManager.selectAll?.also {
                    add(JPopupMenu.Separator())
                    add(
                        swingItem(localization.selectAll, Color.BLACK, KeyEvent.VK_A, it)
                    )
                }

                // Add items that can be defined via ContextMenuDataProvider in other parts of the application 
                for (item in items) {
                    add(
                        JMenuItem(item.label).apply {
                            addActionListener { item.onClick() }
                        }
                    )
                }
            }
        },
        content = content
    )
}

private fun swingItem(
    label: String,
    color: Color,
    key: Int,
    onClick: () -> Unit
) = JMenuItem(label).apply {
    icon = circleIcon(color)
    accelerator = getKeyStroke(key, if (hostOs.isMacOS) META_DOWN_MASK else CTRL_DOWN_MASK)
    addActionListener { onClick() }
}

private fun circleIcon(color: Color) = object : Icon {
    override fun paintIcon(c: Component?, g: Graphics, x: Int, y: Int) {
        g.create().apply {
            this.color = color
            translate(8, 2)
            fillOval(0, 0, 16, 16)
        }
    }

    override fun getIconWidth() = 16

    override fun getIconHeight() = 16
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="fun JPopupTextMenuProvider(owner: Component, content: @Composable () -> Unit) "}

<img src="compose-desktop-context-menu-swing.png" alt="Context menu: Swing interoperability" width="500"/>

## What's next

Explore the tutorials about [other desktop components](https://github.com/JetBrains/compose-multiplatform/tree/master/tutorials#desktop).