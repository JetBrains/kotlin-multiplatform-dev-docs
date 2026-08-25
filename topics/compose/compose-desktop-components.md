[//]: # (title: Desktop-only API)

You can use Compose Multiplatform to create macOS, Linux, and Windows desktop applications. This page gives a short
overview of the desktop-specific components and events. Each section includes a link to a detailed tutorial.

## Components

* [Images and app icons](compose-desktop-images.md)
* [Windows and dialogs](compose-desktop-top-level-windows-management.md)
* [Context menus](compose-desktop-context-menus.md)
* [Tray and notifications](compose-desktop-tray.md)
* [Menu bar](compose-desktop-menu-bar.md)
* [Scrollbars](compose-desktop-scrollbars.md)
* [Tooltips](compose-desktop-tooltips.md)

## Events

* [Mouse events](compose-desktop-mouse-events.md)
* [Keyboard events](compose-desktop-keyboard.md)
* [Tabbing navigation](#tabbing-navigation-between-components)

### Tabbing navigation between components

You can set up navigation between components with the <shortcut>Tab</shortcut> keyboard shortcut for the next component
and <shortcut>⇧ + Tab</shortcut> for the previous one.

By default, the tabbed navigation allows you to move between focusable components in the order of their appearance.
Focusable components include `TextField`, `OutlinedTextField`, and `BasicTextField` composables, as well as components that
use `Modifier.clickable`, such as `Button`, `IconButton`, and `MenuItem`.

For example, here's a window where users can navigate between five text fields using standard shortcuts:

```kotlin
import androidx.compose.ui.window.application
import androidx.compose.ui.window.Window
import androidx.compose.ui.window.WindowState
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.layout.Spacer
import androidx.compose.material.OutlinedTextField
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.DpSize
import androidx.compose.ui.unit.dp

fun main() = application {
    Window(
        state = WindowState(size = DpSize(350.dp, 500.dp)),
        onCloseRequest = ::exitApplication
    ) {
        Box(
            modifier = Modifier.fillMaxSize(),
            contentAlignment = Alignment.Center
        ) {
            Column(
                modifier = Modifier.padding(50.dp)
            ) {
                for (x in 1..5) {
                    val text = remember { mutableStateOf("") }
                    OutlinedTextField(
                        value = text.value,
                        singleLine = true,
                        onValueChange = { text.value = it }
                    )
                    Spacer(modifier = Modifier.height(20.dp))
                }
            }
        }
    }
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="Column() { for (x in 1..5) { OutlinedTextField("}

You can also make a non-focusable component focusable, customize the order of tabbing navigation, and put components into
focus.

For more information, see the [Tabbing navigation and keyboard focus](https://github.com/JetBrains/compose-multiplatform/tree/master/tutorials/Tab_Navigation)
tutorial.

## What's next

* Learn how to [create unit tests for your Compose Multiplatform desktop project](compose-desktop-ui-testing.md).
* Learn how to [create native distributions, installers, and packages for desktop platforms](compose-native-distribution.md).
* Set up [interoperability with Swing and migrate your Swing applications to Compose Multiplatform](compose-desktop-swing-interoperability.md).
* Learn about [accessibility support on different platforms](compose-desktop-accessibility.md).
