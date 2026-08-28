[//]: # (title: Tabbing navigation and keyboard focus)
<web-summary>Learn how to move keyboard focus between components with the Tab shortcut in Compose Multiplatform for desktop.</web-summary>

In Compose Multiplatform for desktop, you can set up navigation between components with the <shortcut>Tab</shortcut>
keyboard shortcut for the next component and <shortcut>Shift+Tab</shortcut> for the previous one.

<include from="compose-desktop-scrollbars.md" element-id="desktop-snippets-intro"/>

## Default tabbing

By default, tabbing navigation enables the user to move between focusable components in the order of their appearance.
This functionality is enabled by default and doesn't require any additional code.

Focusable components include:

* `TextField()`
* `OutlinedTextField()`
* `BasicTextField()`
* `Button()`
* `IconButton()`
* `MenuItem()`

For example, here's a window where the user can navigate between five text fields using standard shortcuts:

```kotlin
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.text.input.TextFieldLineLimits
import androidx.compose.foundation.text.input.rememberTextFieldState
import androidx.compose.material3.TextField
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.DpSize
import androidx.compose.ui.unit.dp
import androidx.compose.ui.window.Window
import androidx.compose.ui.window.WindowState
import androidx.compose.ui.window.application

fun main() = application {
    Window(
        onCloseRequest = ::exitApplication,
        state = WindowState(size = DpSize(350.dp, 500.dp))
    ) {
        Box(
            modifier = Modifier.fillMaxSize(),
            contentAlignment = Alignment.Center
        ) {
            Column(
                modifier = Modifier.padding(50.dp),
                verticalArrangement = Arrangement.spacedBy(20.dp)
            ) {
                repeat(5) {
                    TextField(
                        state = rememberTextFieldState(),
                        lineLimits = TextFieldLineLimits.SingleLine
                    )
                }
            }
        }
    }
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="Column() { repeat(5) { TextField(state = rememberTextFieldState()"}

<img src="compose-desktop-tabbing-default.animated.gif" alt="Default tabbing order" width="450" preview-src="compose-desktop-tabbing-default.png"/>

## Custom focusable components

To include a component that isn't focusable by default in the tabbing order, apply the `focusable()` modifier.

To change the appearance of the component when it receives focus, pass a `MutableInteractionSource` to the `focusable()`
modifier, read the focus state from it with `collectIsFocusedAsState()`, and use that state to change the style of the
component: a different background, a border, or any other highlight. To make the component react to keyboard presses,
handle key events with the `onKeyEvent()` modifier.

The following example turns a `Box()` composable into a button-like component. The box is highlighted when focused, and 
pressing <shortcut>Enter</shortcut> or <shortcut>Space</shortcut> triggers the related action:

```kotlin
import androidx.compose.foundation.background
import androidx.compose.foundation.focusable
import androidx.compose.foundation.interaction.MutableInteractionSource
import androidx.compose.foundation.interaction.collectIsFocusedAsState
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.layout.size
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.ExperimentalComposeUiApi
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.clip
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.graphics.lerp
import androidx.compose.ui.input.key.Key
import androidx.compose.ui.input.key.KeyEventType
import androidx.compose.ui.input.key.key
import androidx.compose.ui.input.key.onKeyEvent
import androidx.compose.ui.input.key.type
import androidx.compose.ui.input.pointer.PointerEventType
import androidx.compose.ui.input.pointer.onPointerEvent
import androidx.compose.ui.unit.DpSize
import androidx.compose.ui.unit.dp
import androidx.compose.ui.window.Window
import androidx.compose.ui.window.WindowState
import androidx.compose.ui.window.application

fun main() = application {
    Window(
        onCloseRequest = ::exitApplication,
        state = WindowState(size = DpSize(350.dp, 450.dp))
    ) {
        MaterialTheme(
            colorScheme = MaterialTheme.colorScheme.copy(
                primary = Color(10, 132, 232),
                secondary = Color(150, 232, 150)
            )
        ) {
            var clicks by remember { mutableStateOf(0) }
            Box(
                modifier = Modifier.fillMaxSize(),
                contentAlignment = Alignment.Center
            ) {
                Column(
                    modifier = Modifier.padding(40.dp),
                    verticalArrangement = Arrangement.spacedBy(20.dp)
                ) {
                    Text(text = "Clicks: $clicks")
                    repeat(5) { index ->
                        FocusableBox("Button ${index + 1}", onClick = { clicks++ })
                    }
                }
            }
        }
    }
}

@OptIn(ExperimentalComposeUiApi::class)
@Composable
fun FocusableBox(
    text: String = "",
    onClick: () -> Unit = {},
    size: DpSize = DpSize(200.dp, 35.dp)
) {
    var isKeyPressed by remember { mutableStateOf(false) }
    val interactionSource = remember { MutableInteractionSource() }
    val isFocused by interactionSource.collectIsFocusedAsState()
    val backgroundColor = when {
        isFocused && isKeyPressed -> lerp(MaterialTheme.colorScheme.secondary, Color(64, 64, 64), 0.3f)
        isFocused -> MaterialTheme.colorScheme.secondary
        else -> MaterialTheme.colorScheme.primary
    }
    Box(
        modifier = Modifier
            .clip(RoundedCornerShape(4.dp))
            .background(backgroundColor)
            .size(size)
            .onPointerEvent(PointerEventType.Press) { onClick() }
            .onKeyEvent {
                if (it.key == Key.Enter || it.key == Key.Spacebar) {
                    when (it.type) {
                        KeyEventType.KeyDown -> isKeyPressed = true
                        KeyEventType.KeyUp -> {
                            isKeyPressed = false
                            onClick()
                        }
                    }
                }
                false
            }
            .focusable(interactionSource = interactionSource),
        contentAlignment = Alignment.Center
    ) {
        Text(text = text, color = Color.White)
    }
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="Box(modifier = Modifier.focusable(interactionSource = interactionSource)"}

<img src="compose-desktop-tabbing-custom-focusable.animated.gif" alt="A custom focusable component" width="450" preview-src="compose-desktop-tabbing-custom-focusable.png"/>

## Custom tabbing order

To move focus in an order other than the order of appearance, combine two modifiers:

* `focusRequester()` attaches a `FocusRequester` handle to a focusable component. If the component [is not focusable by
  default](#custom-focusable-components), the `focusable()` modifier should be applied _after_ `focusRequester()`.
* `focusProperties()` sets the `next` and `previous` elements in the tabbing order: components with a `FocusRequester` 
  handle that are focused by pressing <shortcut>Tab</shortcut> or <shortcut>Shift+Tab</shortcut>.

The following example creates a `FocusRequester` for each of the five text fields and reverses the default tabbing order:

```kotlin
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.text.input.TextFieldLineLimits
import androidx.compose.foundation.text.input.rememberTextFieldState
import androidx.compose.material3.TextField
import androidx.compose.runtime.remember
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.focus.FocusRequester
import androidx.compose.ui.focus.focusProperties
import androidx.compose.ui.focus.focusRequester
import androidx.compose.ui.unit.DpSize
import androidx.compose.ui.unit.dp
import androidx.compose.ui.window.Window
import androidx.compose.ui.window.WindowState
import androidx.compose.ui.window.application

fun main() = application {
    Window(
        onCloseRequest = ::exitApplication,
        state = WindowState(size = DpSize(350.dp, 500.dp))
    ) {
        val focusRequesters = remember { List(5) { FocusRequester() } }
        Box(
            modifier = Modifier.fillMaxSize(),
            contentAlignment = Alignment.Center
        ) {
            Column(
                modifier = Modifier.padding(50.dp),
                verticalArrangement = Arrangement.spacedBy(20.dp)
            ) {
                focusRequesters.forEachIndexed { index, focusRequester ->
                    TextField(
                        state = rememberTextFieldState(),
                        lineLimits = TextFieldLineLimits.SingleLine,
                        modifier = Modifier
                            .focusRequester(focusRequester)
                            .focusProperties {
                                // Reverses the default order:
                                next = focusRequesters[(index - 1 + focusRequesters.size) % focusRequesters.size]
                                previous = focusRequesters[(index + 1) % focusRequesters.size]
                            }
                    )
                }
            }
        }
    }
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="Modifier.focusRequester(focusRequester).focusProperties { next ="}

<img src="compose-desktop-tabbing-custom-order.animated.gif" alt="Custom tabbing order" width="450" preview-src="compose-desktop-tabbing-custom-order.png"/>

## Moving focus from code

To put a component into focus without user interaction, attach a `FocusRequester` handle to a focusable component using 
the `focusRequester()` modifier and call `FocusRequester.requestFocus()`. If the component [is not focusable by
default](#custom-focusable-components), the `focusable()` modifier should be applied _after_ `focusRequester()`.

In the following example, a button moves the focus to a text field and back to itself:

```kotlin
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.text.input.TextFieldLineLimits
import androidx.compose.foundation.text.input.rememberTextFieldState
import androidx.compose.material3.Button
import androidx.compose.material3.Text
import androidx.compose.material3.TextField
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.focus.FocusRequester
import androidx.compose.ui.focus.focusRequester
import androidx.compose.ui.unit.DpSize
import androidx.compose.ui.unit.dp
import androidx.compose.ui.window.Window
import androidx.compose.ui.window.WindowState
import androidx.compose.ui.window.application

fun main() = application {
    Window(
        onCloseRequest = ::exitApplication,
        state = WindowState(size = DpSize(350.dp, 450.dp))
    ) {
        val buttonFocusRequester = remember { FocusRequester() }
        val textFieldFocusRequester = remember { FocusRequester() }
        var isTextFieldFocused by remember { mutableStateOf(false) }
        Box(
            modifier = Modifier.fillMaxSize(),
            contentAlignment = Alignment.Center
        ) {
            Column(
                modifier = Modifier.padding(50.dp),
                verticalArrangement = Arrangement.spacedBy(20.dp)
            ) {
                Button(
                    onClick = {
                        isTextFieldFocused = !isTextFieldFocused
                        if (isTextFieldFocused) {
                            textFieldFocusRequester.requestFocus()
                        } else {
                            buttonFocusRequester.requestFocus()
                        }
                    },
                    modifier = Modifier
                        .fillMaxWidth()
                        .focusRequester(buttonFocusRequester)
                ) {
                    Text(text = "Focus switcher")
                }
                TextField(
                    state = rememberTextFieldState(),
                    lineLimits = TextFieldLineLimits.SingleLine,
                    modifier = Modifier.focusRequester(textFieldFocusRequester)
                )
            }
        }
    }
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="Button(onClick = { textFieldFocusRequester.requestFocus()"}

<img src="compose-desktop-tabbing-move-focus-from-code.animated.gif" alt="Moving focus from code" width="450" preview-src="compose-desktop-tabbing-move-focus-from-code.png"/>

### Focusing a component when it appears

Forms and dialogs commonly focus their first input right away, so the user can start typing without reaching for the
mouse. In this use-case, the focus should be requested from a `LaunchedEffect(Unit)` block, which runs once after the 
component enters the composition.

In the following example, the first text field is focused as soon as the window opens:

```kotlin
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.text.input.TextFieldLineLimits
import androidx.compose.foundation.text.input.rememberTextFieldState
import androidx.compose.material3.TextField
import androidx.compose.runtime.LaunchedEffect
import androidx.compose.runtime.remember
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.focus.FocusRequester
import androidx.compose.ui.focus.focusRequester
import androidx.compose.ui.unit.DpSize
import androidx.compose.ui.unit.dp
import androidx.compose.ui.window.Window
import androidx.compose.ui.window.WindowState
import androidx.compose.ui.window.application

fun main() = application {
    Window(
        onCloseRequest = ::exitApplication,
        state = WindowState(size = DpSize(350.dp, 300.dp))
    ) {
        val focusRequester = remember { FocusRequester() }
        Box(
            modifier = Modifier.fillMaxSize(),
            contentAlignment = Alignment.Center
        ) {
            Column(
                modifier = Modifier.padding(50.dp),
                verticalArrangement = Arrangement.spacedBy(20.dp)
            ) {
                TextField(
                    state = rememberTextFieldState(),
                    lineLimits = TextFieldLineLimits.SingleLine,
                    modifier = Modifier.focusRequester(focusRequester)
                )
                TextField(
                    state = rememberTextFieldState(),
                    lineLimits = TextFieldLineLimits.SingleLine
                )
            }
        }
        LaunchedEffect(Unit) {
            focusRequester.requestFocus()
        }
    }
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="LaunchedEffect(Unit) { focusRequester.requestFocus()"}

<img src="compose-desktop-tabbing-focus-on-appearance.animated.gif" alt="Focus text field on appearance" width="450" preview-src="compose-desktop-tabbing-focus-on-appearance.png"/>

## Moving focus from multiline text fields

In multiline text fields, pressing <shortcut>Tab</shortcut> inserts a tab character instead of moving the focus to the 
next component:

```kotlin
Column {
    repeat(5) {
        TextField(
            state = rememberTextFieldState("Hello, World!"),
            // MultiLine is the default value of lineLimits
            lineLimits = TextFieldLineLimits.MultiLine(),
            modifier = Modifier.padding(8.dp)
        )
    }
}
```

> This issue affects any text field that accepts more than one line, which is the default behavior.
> 
{style="note"}

This is a known issue, [CMP-5822](https://youtrack.jetbrains.com/issue/CMP-5822). As a workaround, intercept the 
<shortcut>Tab</shortcut> key with the `onPreviewKeyEvent` modifier and move the focus using the `FocusManager` from 
`LocalFocusManager`:

```kotlin
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.text.input.TextFieldLineLimits
import androidx.compose.foundation.text.input.rememberTextFieldState
import androidx.compose.material3.TextField
import androidx.compose.runtime.Composable
import androidx.compose.ui.Modifier
import androidx.compose.ui.focus.FocusDirection
import androidx.compose.ui.input.key.Key
import androidx.compose.ui.input.key.KeyEventType
import androidx.compose.ui.input.key.isShiftPressed
import androidx.compose.ui.input.key.key
import androidx.compose.ui.input.key.onPreviewKeyEvent
import androidx.compose.ui.input.key.type
import androidx.compose.ui.platform.LocalFocusManager
import androidx.compose.ui.unit.dp
import androidx.compose.ui.window.singleWindowApplication

fun main() = singleWindowApplication(title = "Multiline text fields") {
    Column {
        repeat(5) {
            TextField(
                state = rememberTextFieldState("Hello, World!"),
                lineLimits = TextFieldLineLimits.MultiLine(),
                modifier = Modifier.padding(8.dp).moveFocusOnTab()
            )
        }
    }
}

@Composable
fun Modifier.moveFocusOnTab(): Modifier {
    val focusManager = LocalFocusManager.current
    return onPreviewKeyEvent {
        if (it.type == KeyEventType.KeyDown && it.key == Key.Tab) {
            focusManager.moveFocus(
                if (it.isShiftPressed) FocusDirection.Previous else FocusDirection.Next
            )
            true
        } else {
            false
        }
    }
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="fun Modifier.moveFocusOnTab() { focusManager.moveFocus("}

## What's next

* Learn more about handling [keyboard events](compose-desktop-keyboard.md).
* Learn about [accessibility support on different platforms](compose-desktop-accessibility.md).
* Explore the tutorials about [other desktop components](compose-desktop-components.md).
