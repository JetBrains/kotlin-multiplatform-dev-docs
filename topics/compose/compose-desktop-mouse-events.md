[//]: # (title: Mouse event listeners)

In your desktop project, you can listen to various mouse events, like clicking, moving, scrolling, or entering and exiting the input region.

## Click listeners

Click listeners are available in both Compose Multiplatform for Android and Compose Multiplatform for desktop, so your code will work on both platforms.
For example, here is how to set up simple click listeners with the `onClick`, `onDoubleClick`, and `onLongClick` modifiers:

```kotlin
import androidx.compose.foundation.ExperimentalFoundationApi
import androidx.compose.foundation.background
import androidx.compose.foundation.combinedClickable
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxHeight
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.material.Text
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableIntStateOf
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.unit.sp
import androidx.compose.ui.window.singleWindowApplication

fun main() = singleWindowApplication(title = "Mouse clicks") {
    var count by remember { mutableIntStateOf(0) }
    Box(contentAlignment = Alignment.Center, modifier = Modifier.fillMaxWidth()) {
        var text by remember { mutableStateOf("Click magenta box!") }
        Column {
            @OptIn(ExperimentalFoundationApi::class)
            Box(modifier = Modifier
                .background(Color.Magenta)
                .fillMaxWidth(0.7f)
                .fillMaxHeight(0.7f)
                .combinedClickable(
                    onClick = {
                        text = "Click! ${count++}"
                    },
                    onDoubleClick = {
                        text = "Double click! ${count++}"
                    },
                    onLongClick = {
                        text = "Long click! ${count++}"
                    }
                )
            )
            Text(text = text, fontSize = 40.sp)
        }
    }
}
```
{initial-collapse-state="collapsed" collapsed-title="Box(modifier = Modifier.combinedClickable(onClick = { text ="}

<img src="compose-mouse-click-listeners.png" alt="Mouse click listeners" width="600" animated="true"/>

The `combinedClickable` modifier supports only the primary button (left mouse button) and touch events. If you need to handle buttons differently, 
see the [`Modifier.onClick`](#experimental-onclick-handlers) section.

## Move listeners

> The `onPointerEvent` modifier is [Experimental](supported-platforms.md#compose-multiplatform-ui-framework-stability-levels). Opt-in is required (see details below),
> and you should use it only for evaluation purposes. 
> For a [Stable](supported-platforms.md#compose-multiplatform-ui-framework-stability-levels) API, see [`Modifier.pointerInput`](#listening-for-raw-events-in-common-code-via-pointerinput).
>
{type="warning"}

To create a pointer move listener that changes the background color of the window according to the mouse pointer position, 
add the following code:

```kotlin
import androidx.compose.foundation.background
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.wrapContentSize
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.ExperimentalComposeUiApi
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.input.pointer.PointerEventType
import androidx.compose.ui.input.pointer.onPointerEvent
import androidx.compose.ui.window.singleWindowApplication

@OptIn(ExperimentalComposeUiApi::class)
fun main() = singleWindowApplication(title = "Mouse move listeners") {
    var color by remember { mutableStateOf(Color(0, 0, 0)) }
    Box(modifier = Modifier
        .wrapContentSize(Alignment.Center)
        .fillMaxSize()
        .background(color = color)
        .onPointerEvent(PointerEventType.Move) {
            val position = it.changes.first().position
            color = Color(position.x.toInt() % 256, position.y.toInt() % 256, 0)
        }
    )
}
```
{initial-collapse-state="collapsed" collapsed-title="Box(modifier = Modifier.onPointerEvent(PointerEventType.Move) { color ="}

<img src="compose-mouse-move-listeners.png" alt="Mouse move listeners" width="600" animated="true"/>

## Enter listeners

> The `onPointerEvent` modifier is [Experimental](supported-platforms.md#compose-multiplatform-ui-framework-stability-levels). Opt-in is required (see details below),
> and you should use it only for evaluation purposes.
> For a [Stable](supported-platforms.md#compose-multiplatform-ui-framework-stability-levels) API, see [`Modifier.pointerInput`](#listening-for-raw-events-in-common-code-via-pointerinput).
>
{type="warning"}

Compose Multiplatform for desktop supports handlers of the pointer entering and exiting the input region. For example, the following code will change the font style of a line on hover:

```kotlin
import androidx.compose.foundation.background
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.material.Text
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.ExperimentalComposeUiApi
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.input.pointer.PointerEventType
import androidx.compose.ui.input.pointer.onPointerEvent
import androidx.compose.ui.text.font.FontStyle
import androidx.compose.ui.text.style.TextAlign
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp
import androidx.compose.ui.window.singleWindowApplication

@OptIn(ExperimentalComposeUiApi::class)
fun main() = singleWindowApplication(title = "Mouse enter listeners") {
    Column(
        Modifier.background(Color.White),
        verticalArrangement = Arrangement.spacedBy(10.dp),
    ) {
        repeat(10) { index ->
            var active by remember { mutableStateOf(false) }
            Text(modifier = Modifier
                .fillMaxWidth()
                .background(color = if (active) Color.Green else Color.White)
                .onPointerEvent(PointerEventType.Enter) { active = true }
                .onPointerEvent(PointerEventType.Exit) { active = false },
                fontSize = 30.sp,
                fontStyle = if (active) FontStyle.Italic else FontStyle.Normal,
                text = "Item $index",
                textAlign = TextAlign.Center
            )
        }
    }
}
```
{initial-collapse-state="collapsed" collapsed-title="Text(modifier = Modifier.onPointerEvent(PointerEventType.Enter) { active ="}

<img src="compose-mouse-enter-listeners.png" alt="Mouse enter listeners" width="600" animated="true"/>

## Scroll listeners

> The `onPointerEvent` modifier is [Experimental](supported-platforms.md#compose-multiplatform-ui-framework-stability-levels). Opt-in is required (see details below),
> and you should use it only for evaluation purposes.
> For a [Stable](supported-platforms.md#compose-multiplatform-ui-framework-stability-levels) API, see [`Modifier.pointerInput`](#listening-for-raw-events-in-common-code-via-pointerinput).
>
{type="warning"}

The following code sample demonstrates how you can increase or decrease the displayed number depending on the mouse scroll direction:

```kotlin
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.material.Text
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableFloatStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.ExperimentalComposeUiApi
import androidx.compose.ui.Modifier
import androidx.compose.ui.input.pointer.PointerEventType
import androidx.compose.ui.input.pointer.onPointerEvent
import androidx.compose.ui.unit.sp
import androidx.compose.ui.window.singleWindowApplication

@OptIn(ExperimentalComposeUiApi::class)
fun main() = singleWindowApplication(title = "Mouse scroll listeners") {
    var number by remember { mutableFloatStateOf(0f) }
    Box(
        Modifier
            .fillMaxSize()
            .onPointerEvent(PointerEventType.Scroll) {
                number += it.changes.first().scrollDelta.y
            },
        contentAlignment = Alignment.Center
    ) {
        Text("Scroll to change the number: $number", fontSize = 30.sp)
    }
}
```
{initial-collapse-state="collapsed" collapsed-title="Box(Modifier.onPointerEvent(PointerEventType.Scroll) { number +="}

<img src="compose-mouse-scroll-listeners.png" alt="Mouse scroll listeners" width="600" animated="true"/>

## Experimental onClick handlers

> The `onClick` modifier is [Experimental](supported-platforms.md#compose-multiplatform-ui-framework-stability-levels) and supported in desktop projects only. Opt-in is required (see details below),
> and you should use it only for evaluation purposes.
>
{type="warning"}

`Modifier.onClick` provides independent callbacks for clicks, double clicks, and long clicks. It handles clicks originating from pointer events only and doesn't handle accessibility `click` events out of the box.

You can configure each `onClick` to target specific pointer events using `matcher: PointerMatcher` and `keyboardModifiers: PointerKeyboardModifiers.() -> Boolean`:
* `matcher` allows you to choose which mouse button should trigger a click event. 
* `keyboardModifiers` allows you to filter pointer events that have specified key pressed.

You can also create a chain of multiple `onClick` modifiers to handle different clicks with different conditions of the matcher and keyboard modifiers. 
Unlike `clickable`, `onClick` doesn't have default `Modifier.indication` and `Modifier.semantics`, and it doesn't trigger a click event when you press `Enter`. If necessary, add these modifiers separately. 
You should declare the most generic (with the least number of conditions) `onClick` handlers before others to ensure correct propagation of events.

```kotlin
import androidx.compose.animation.AnimatedContent
import androidx.compose.foundation.ExperimentalFoundationApi
import androidx.compose.foundation.LocalIndication
import androidx.compose.foundation.PointerMatcher
import androidx.compose.foundation.background
import androidx.compose.foundation.indication
import androidx.compose.foundation.interaction.MutableInteractionSource
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.size
import androidx.compose.foundation.onClick
import androidx.compose.material.Text
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.input.pointer.PointerButton
import androidx.compose.ui.input.pointer.isAltPressed
import androidx.compose.ui.input.pointer.isShiftPressed
import androidx.compose.ui.text.style.TextAlign
import androidx.compose.ui.unit.dp
import androidx.compose.ui.window.singleWindowApplication

@OptIn(ExperimentalFoundationApi::class)
fun main() = singleWindowApplication(title = "Mouse clicks") {
    Column {
        var topBoxText by remember { mutableStateOf("Click me\nusing LMB or LMB + Shift") }
        var topBoxCount by remember { mutableIntStateOf(0) }
        // No indication on interaction
        Box(modifier = Modifier
            .size(300.dp, 200.dp)
            .background(Color.LightGray)
            // The most generic click handler (without extra conditions) should be the first one
            .onClick {
                // Receives all left mouse button clicks except for when Shift is pressed
                println("Click with primary button")
                topBoxText = "LMB ${topBoxCount++}"
            }.onClick(
                keyboardModifiers = { isShiftPressed } // Accepts clicks only when Shift is pressed
            ) {
                // Receives all left mouse button clicks when Shift is pressed
                println("Click with primary button and shift pressed")
                topBoxCount++
                topBoxText = "LMB + Shift ${topBoxCount++}"
            }
        ) {
            AnimatedContent(
                targetState = topBoxText,
                modifier = Modifier.align(Alignment.Center)
            ) {
                Text(text = it, textAlign = TextAlign.Center)
            }
        }

        var bottomBoxText by remember { mutableStateOf("Click me\nusing LMB or\nRMB + Alt") }
        var bottomBoxCount by remember { mutableStateOf(0) }
        val interactionSource = remember { MutableInteractionSource() }
        // With indication on interaction
        Box(modifier = Modifier
            .size(300.dp, 200.dp)
            .background(Color.Yellow)
            .onClick(
                enabled = true,
                interactionSource = interactionSource,
                matcher = PointerMatcher.mouse(PointerButton.Secondary), // Right mouse button
                keyboardModifiers = { isAltPressed }, // Accepts clicks only when Alt is pressed
                onLongClick = { // Optional
                    bottomBoxText = "RMB Long Click + Alt ${bottomBoxCount++}"
                    println("Long Click with secondary button and Alt pressed")
                },
                onDoubleClick = { // Optional
                    bottomBoxText = "RMB Double Click + Alt ${bottomBoxCount++}"
                    println("Double Click with secondary button and Alt pressed")
                },
                onClick = {
                    bottomBoxText = "RMB Click + Alt ${bottomBoxCount++}"
                    println("Click with secondary button and Alt pressed")
                }
            )
            .onClick(interactionSource = interactionSource) { // Uses default parameters
                bottomBoxText = "LMB Click ${bottomBoxCount++}"
                println("Click with primary button (mouse left button)")
            }
            .indication(interactionSource, LocalIndication.current)
        ) {
            AnimatedContent(
                targetState = bottomBoxText,
                modifier = Modifier.align(Alignment.Center)
            ) {
                Text(text = it, textAlign = TextAlign.Center)
            }
        }
    }
}
```
{initial-collapse-state="collapsed" collapsed-title="Box(modifier = Modifier.onClick {"}

<img src="compose-onclick-modifier.png" alt="Modifier.onClick" width="600" animated="true"/>

## Experimental onDrag modifier

> The `onDrag` modifier is [Experimental](supported-platforms.md#compose-multiplatform-ui-framework-stability-levels) and supported in desktop projects only. Opt-in is required (see details below),
> and you should use it only for evaluation purposes.
>
{type="warning"}

With `Modifier.onDrag`, you can specify the pointer that should trigger the drag via `matcher: PointerMatcher`. 
Similar to `onClick`, you can chain together many `onDrag` modifiers.

You can also check the state of keyboard modifiers via `LocalWindowInfo.current.keyboardModifier` for cases when keys can alter the behavior of the drag. 
For example, when you move an item with a simple drag, and copy/paste an item with a drag and `Ctrl` pressed.

The following code sample demonstrates how to handle drag events triggered by left and right mouse buttons, and with the keyboard involved:

```kotlin
import androidx.compose.foundation.ExperimentalFoundationApi
import androidx.compose.foundation.PointerMatcher
import androidx.compose.foundation.background
import androidx.compose.foundation.gestures.onDrag
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.offset
import androidx.compose.foundation.layout.size
import androidx.compose.material.Text
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.geometry.Offset
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.input.pointer.PointerButton
import androidx.compose.ui.input.pointer.isCtrlPressed
import androidx.compose.ui.platform.LocalWindowInfo
import androidx.compose.ui.unit.IntOffset
import androidx.compose.ui.unit.dp
import androidx.compose.ui.window.singleWindowApplication

@OptIn(ExperimentalFoundationApi::class)
fun main() = singleWindowApplication(title = "Drag") {
    val windowInfo = LocalWindowInfo.current

    Column {
        var topBoxOffset by remember { mutableStateOf(Offset(0f, 0f)) }

        Box(modifier = Modifier
            .offset {
                IntOffset(topBoxOffset.x.toInt(), topBoxOffset.y.toInt())
            }
            .size(200.dp)
            .background(Color.Green)
            .onDrag { // By default: enabled = true, matcher = PointerMatcher.Primary (left mouse button)
                topBoxOffset += it
            }
        ) {
            Text(text = "Drag with LMB", modifier = Modifier.align(Alignment.Center))
        }

        var bottomBoxOffset by remember { mutableStateOf(Offset(0f, 0f)) }

        Box(modifier = Modifier
            .offset {
                IntOffset(bottomBoxOffset.x.toInt(), bottomBoxOffset.y.toInt())
            }
            .size(200.dp)
            .background(Color.LightGray)
            .onDrag(
                matcher = PointerMatcher.mouse(PointerButton.Secondary), // Right mouse button
                onDragStart = {
                    println("Gray Box: drag start")
                },
                onDragEnd = {
                    println("Gray Box: drag end")
                }
            ) {
                val keyboardModifiers = windowInfo.keyboardModifiers
                bottomBoxOffset += if (keyboardModifiers.isCtrlPressed) it * 2f else it
            }
        ) {
            Text(
                text = "Drag with RMB,\ntry with CTRL",
                modifier = Modifier.align(Alignment.Center)
            )
        }
    }
}
```
{initial-collapse-state="collapsed" collapsed-title="Box(modifier = Modifier.onDrag {"}

<img src="compose-ondrag-modifier.png" alt="Modifier.onDrag" width="600" animated="true"/>

There is also a non-modifier way to handle drags using `suspend fun PointerInputScope.detectDragGestures`:

```kotlin
import androidx.compose.foundation.ExperimentalFoundationApi
import androidx.compose.foundation.PointerMatcher
import androidx.compose.foundation.background
import androidx.compose.foundation.gestures.detectDragGestures
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.offset
import androidx.compose.foundation.layout.size
import androidx.compose.material.Text
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.geometry.Offset
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.input.pointer.pointerInput
import androidx.compose.ui.unit.IntOffset
import androidx.compose.ui.unit.dp
import androidx.compose.ui.window.singleWindowApplication

@OptIn(ExperimentalFoundationApi::class)
fun main() = singleWindowApplication(title = "Drag") {
    var topBoxOffset by remember { mutableStateOf(Offset(0f, 0f)) }

    Box(modifier = Modifier
        .offset {
            IntOffset(topBoxOffset.x.toInt(), topBoxOffset.y.toInt())
        }
        .size(200.dp)
        .background(Color.Green)
        .pointerInput(Unit) {
            detectDragGestures(
                matcher = PointerMatcher.Primary
            ) {
                topBoxOffset += it
            }
        }
    ) {
        Text(text = "Drag with LMB", modifier = Modifier.align(Alignment.Center))
    }
}
```
{initial-collapse-state="collapsed" collapsed-title="Modifier.pointerInput(Unit) { detectDragGestures(matcher = PointerMatcher.Primary)"}

## Accessing raw AWT events with Swing interoperability

> The `onPointerEvent` modifier is [Experimental](supported-platforms.md#compose-multiplatform-ui-framework-stability-levels). Opt-in is required (see details below),
> and you should use it only for evaluation purposes.
> For a [Stable](supported-platforms.md#compose-multiplatform-ui-framework-stability-levels) API, see [`Modifier.pointerInput`](#listening-for-raw-events-in-common-code-via-pointerinput).
>
{type="warning"}

Compose Multiplatform for desktop uses Swing under the hood and allows to access raw AWT events:

```kotlin
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.material.Text
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.ExperimentalComposeUiApi
import androidx.compose.ui.Modifier
import androidx.compose.ui.awt.awtEventOrNull
import androidx.compose.ui.input.pointer.PointerEventType
import androidx.compose.ui.input.pointer.onPointerEvent
import androidx.compose.ui.window.singleWindowApplication

@OptIn(ExperimentalComposeUiApi::class)
fun main() = singleWindowApplication(title = "Raw AWT events") {
    var text by remember { mutableStateOf("") }

    Box(
        Modifier
            .fillMaxSize()
            .onPointerEvent(PointerEventType.Press) {
                text = it.awtEventOrNull?.locationOnScreen?.toString().orEmpty()
            },
        contentAlignment = Alignment.Center
    ) {
        Text(text)
    }
}
```
{initial-collapse-state="collapsed" collapsed-title="Box(Modifier.onPointerEvent(PointerEventType.Press) { text ="}

<img src="compose-raw-awt-events.png" alt="Swing interoperability" width="600" animated="true"/>

## Listening for raw events in common code via pointerInput

In the snippets above we use the `Modifier.onPointerEvent` function, which is a helper function that subscribes to a type of pointer events. It is a new and short variant of the `Modifier.pointerInput` function. 
It is currently experimental and desktop-only, so you can't use it in common code.

If you need to subscribe to events in common code, or if you need a stable API, you can use the `Modifier.pointerInput` function:

```kotlin
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.material.Text
import androidx.compose.runtime.mutableStateListOf
import androidx.compose.runtime.remember
import androidx.compose.ui.Modifier
import androidx.compose.ui.input.pointer.PointerEventType
import androidx.compose.ui.input.pointer.pointerInput
import androidx.compose.ui.window.singleWindowApplication

fun main() = singleWindowApplication(title = "Raw events via Modifier.pointerInput") {
    val list = remember { mutableStateListOf<String>() }

    Column(
        Modifier
            .fillMaxSize()
            .pointerInput(Unit) {
                awaitPointerEventScope {
                    while (true) {
                        val event = awaitPointerEvent()
                        val position = event.changes.first().position
                        // On each relayout, Compose will send a synthetic Move event,
                        // so we skip it to avoid event spam
                        if (event.type != PointerEventType.Move) {
                            list.add(0, "${event.type} $position")
                        }
                    }
                }
            },
    ) {
        for (item in list.take(20)) {
            Text(item)
        }
    }
}
```
{initial-collapse-state="collapsed" collapsed-title="Column(Modifier.pointerInput(Unit) { awaitPointerEventScope {"}

<img src="compose-raw-events.png" alt="Raw events via Modifier.pointerInput" width="600" animated="true"/>

## What's next

Explore the tutorials about [other desktop components](https://github.com/JetBrains/compose-multiplatform/tree/master/tutorials#desktop).