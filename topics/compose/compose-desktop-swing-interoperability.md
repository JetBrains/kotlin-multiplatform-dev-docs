[//]: # (title: Swing interoperability)

Here, you'll learn about using Swing components in the Compose Multiplatform application and vice versa, 
the limitations and advantages of this interoperability, and when you should or shouldn't use this approach.

The interoperability between Compose Multiplatform and Swing aims to help you:
* Simplify and smooth the migration process of Swing applications to Compose Multiplatform.
* Enhance Compose Multiplatform applications using Swing components when no Compose analogues are available.

In many cases, it's more effective to implement a missing component directly in Compose Multiplatform (and contribute it to the community) 
rather than using a Swing component within a Compose Multiplatform application.

## Swing interop use cases and limitations

### Compose Multiplatform component in a Swing app

The first use case involves adding a Compose Multiplatform component to a Swing application. 
You can achieve it using the `ComposePanel` Swing component to render the Compose Multiplatform part of the application. 
From Swing's perspective, `ComposePanel` is another Swing component, and handles it accordingly.

Note that all Compose Multiplatform components, including popups, tooltips, and context menus, are rendered within Swing's `ComposePanel` and positioned and resized inside it. 
Therefore, consider replacing these components with Swing-based implementations, or try two new experimental features:

[Off-screen rendering](#experimental-off-screen-rendering) 
: Allows rendering of compose panels directly on Swing components.

[Separate platform views for popups, dialogs, and dropdowns](#experimental-separate-views-for-popups)
: Popups are no longer limited by the initial composable canvas or the app window.


Here are several scenarios for using `ComposePanel`:
* Embed animated objects or a whole panel of animated objects into your application (for example, selection of emoticons or a toolbar with animated reactions to events).
* Implement an interactive rendering area such as graphics or infographics in your application, 
which is easier and more convenient to accomplish using Compose Multiplatform.
* Integrate a complex rendering area (potentially even animated) into your application, which is simpler with Compose Multiplatform.
* Replace complex parts of the user interface in your Swing-based application, as Compose Multiplatform provides a convenient 
component layout system and a wide range of built-in components and options for quickly creating custom components.

### Swing component in a Compose Multiplatform app

Another use case is when you need to use a component that exists in Swing but has no analog in Compose Multiplatform. 
If creating its new implementation from scratch is too time-consuming, try `SwingPanel`. The `SwingPanel` function serves 
as a wrapper that manages the size, position, and rendering of a Swing component placed on top of a Compose Multiplatform component.

Note that the Swing component within `SwingPanel` will always be layered above the Compose Multiplatform component, 
so anything positioned underneath your `SwingPanel` will be clipped by the Swing component. To avoid clipping and overlapping issues,
try [experimental interop blending](#experimental-interop-blending). If there is still a risk of incorrect rendering, 
you can redesign the UI accordingly or avoid using `SwingPanel` and try implementing the missing component, contributing to technology development.

Here are scenarios for using `SwingPanel`:
* Your application does not require popups, tooltips, or context menus, or at least they are not inside your `SwingPanel`.
* `SwingPanel` remains in a fixed position. In this case, you reduce the risk of glitches and artifacts when the Swing component's position changes. 
However, this condition is not mandatory and should be tested for each particular case.
  
Compose Multiplatform and Swing can be combined in both ways, allowing for flexible UI design. You can place a `SwingPanel` inside a `ComposePanel`, 
which can also be inside another `SwingPanel`. 
However, before using such nested combinations, consider potential rendering glitches. 
Refer to [Layout with nested `SwingPanel` and `ComposePanel`](#layout-with-nested-swing-and-compose-multiplatform-components) for a code sample.

## Use Compose Multiplatform in a Swing application

`ComposePanel` allows you to create a UI with Compose Multiplatform within a Swing-based application.
Add an instance of `ComposePanel` to your Swing layout and define the composition inside `setContent`:

```kotlin
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.size
import androidx.compose.foundation.layout.width
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material.Button
import androidx.compose.material.Surface
import androidx.compose.material.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.MutableState
import androidx.compose.runtime.mutableStateOf
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.awt.ComposePanel
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.unit.dp
import java.awt.BorderLayout
import java.awt.Dimension
import javax.swing.JButton
import javax.swing.JFrame
import javax.swing.SwingUtilities
import javax.swing.WindowConstants

val northClicks = mutableStateOf(0)
val westClicks = mutableStateOf(0)
val eastClicks = mutableStateOf(0)

fun main() = SwingUtilities.invokeLater {
    val window = JFrame()

    // Creates ComposePanel
    val composePanel = ComposePanel()
    window.defaultCloseOperation = WindowConstants.EXIT_ON_CLOSE
    window.title = "SwingComposeWindow"

    window.contentPane.add(actionButton("NORTH", action = { northClicks.value++ }), BorderLayout.NORTH)
    window.contentPane.add(actionButton("WEST", action = { westClicks.value++ }), BorderLayout.WEST)
    window.contentPane.add(actionButton("EAST", action = { eastClicks.value++ }), BorderLayout.EAST)
    window.contentPane.add(
        actionButton(
            text = "SOUTH/REMOVE COMPOSE",
            action = {
                window.contentPane.remove(composePanel)
            }
        ),
        BorderLayout.SOUTH
    )

    // Adds ComposePanel to JFrame
    window.contentPane.add(composePanel, BorderLayout.CENTER)

    // Sets the content
    composePanel.setContent {
        ComposeContent()
    }

    window.setSize(800, 600)
    window.isVisible = true
}

fun actionButton(text: String, action: () -> Unit): JButton {
    val button = JButton(text)
    button.toolTipText = "Tooltip for $text button."
    button.preferredSize = Dimension(100, 100)
    button.addActionListener { action() }
    return button
}

@Composable
fun ComposeContent() {
    Box(
        modifier = Modifier.fillMaxSize(),
        contentAlignment = Alignment.Center
    ) {
        Row {
            Counter("West", westClicks)
            Spacer(modifier = Modifier.width(25.dp))
            Counter("North", northClicks)
            Spacer(modifier = Modifier.width(25.dp))
            Counter("East", eastClicks)
        }
    }
}

@Composable
fun Counter(text: String, counter: MutableState<Int>) {
    Surface(
        modifier = Modifier.size(130.dp, 130.dp),
        color = Color(180, 180, 180),
        shape = RoundedCornerShape(4.dp)
    ) {
        Column {
            Box(
                modifier = Modifier.height(30.dp).fillMaxWidth(),
                contentAlignment = Alignment.Center
            ) {
                Text(text = "${text}Clicks: ${counter.value}")
            }
            Spacer(modifier = Modifier.height(25.dp))
            Box(
                modifier = Modifier.fillMaxSize(),
                contentAlignment = Alignment.Center
            ) {
                Button(onClick = { counter.value++ }) {
                    Text(text = text, color = Color.White)
                }
            }
        }
    }
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="composePanel.setContent { ComposeContent() }"}

<img src="compose-desktop-swing-composepanel.animated.gif" alt="IntegrationWithSwing" preview-src="compose-desktop-swing-composepanel.png" width="799"/>

### Experimental off-screen rendering

An experimental mode allows rendering compose panels directly on Swing components.
This prevents transitional rendering issues when panels are shown, hidden, or resized.
It also enables proper layering when combining Swing components and compose panels: a Swing component can be shown 
above or beneath a `ComposePanel`.

> Off-screen rendering is [Experimental](supported-platforms.md#compose-multiplatform-ui-framework-stability-levels),
> and you should use it only for evaluation purposes.
>
{style="warning"}

To enable off-screen rendering, use the `compose.swing.render.on.graphics` system property.
The property must be set before executing any Compose code in your application, so it is recommended to enable it using 
the `-D` command-line JVM argument at startup: 

```Console
-Dcompose.swing.render.on.graphics=true
```

Alternatively, use `System.setProperty()` at the entry point:

```kotlin
fun main() {
    System.setProperty("compose.swing.render.on.graphics", "true")
    ...
}
```

### Experimental separate views for popups

It can be important that popup elements such as tooltips and dropdown menus are not limited by the initial composable canvas 
or the app window. For example, when the composable view does not occupy the full screen but needs to spawn an alert dialog.

> Creating separate views or windows for popups is [Experimental](supported-platforms.md#compose-multiplatform-ui-framework-stability-levels). Opt-in is required (see details below),
> and you should use it only for evaluation purposes.
>
{style="warning"}

To create separate views or windows for popups on desktop, set the `compose.layers.type` system property. Supported values:
* `WINDOW` creates `Popup` and `Dialog` components as separate undecorated windows.
* `COMPONENT` creates `Popup` or `Dialog` as a separate Swing component in the same window. Note that the setting requires enabled
off-screen rendering (see the [Experimental off-screen rendering](#experimental-off-screen-rendering) section), and off-screen rendering only works 
for `ComposePanel` components, not full window applications.

Note that popups and dialogs are still unable to draw anything outside their own bounds (for example, the shadow of the topmost container).

Here is an example of the code that uses the `COMPONENT` property:

```kotlin
import androidx.compose.foundation.background
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.size
import androidx.compose.runtime.Composable
import androidx.compose.ui.ExperimentalComposeUiApi
import androidx.compose.ui.Modifier
import androidx.compose.ui.awt.ComposePanel
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.unit.dp
import androidx.compose.ui.window.Dialog
import javax.swing.JFrame
import javax.swing.JLayeredPane
import javax.swing.SwingUtilities
import javax.swing.WindowConstants

@OptIn(ExperimentalComposeUiApi::class)
fun main() = SwingUtilities.invokeLater {
    System.setProperty("compose.swing.render.on.graphics", "true")
    System.setProperty("compose.layers.type", "COMPONENT")

    val window = JFrame()
    window.defaultCloseOperation = WindowConstants.EXIT_ON_CLOSE

    val contentPane = JLayeredPane()
    contentPane.layout = null

    val composePanel = ComposePanel()
    composePanel.setBounds(200, 200, 200, 200)
    composePanel.setContent {
        ComposeContent()
    }
    
    // Uses the full window for dialogs
    composePanel.windowContainer = contentPane
    contentPane.add(composePanel)

    window.contentPane.add(contentPane)
    window.setSize(800, 600)
    window.isVisible = true
}

@Composable
fun ComposeContent() {
    Box(Modifier.fillMaxSize().background(Color.Green)) {
        Dialog(onDismissRequest = {}) {
            Box(Modifier.size(100.dp).background(Color.Yellow))
        }
    }
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="@OptIn(ExperimentalComposeUiApi::class) fun main()"}

## Use Swing in a Compose Multiplatform application

`SwingPanel` allows you to create a UI with Swing within a Compose Multiplatform application.
Use the `factory` parameter of `SwingPanel` to create a Swing `JPanel`:

```kotlin
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.size
import androidx.compose.material.Button
import androidx.compose.material.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.awt.SwingPanel
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.unit.dp
import androidx.compose.ui.window.singleWindowApplication
import java.awt.Component
import javax.swing.BoxLayout
import javax.swing.JButton
import javax.swing.JPanel

fun main() = singleWindowApplication(title = "SwingPanel") {
    val counter = remember { mutableStateOf(0) }

    val inc: () -> Unit = { counter.value++ }
    val dec: () -> Unit = { counter.value-- }

    Box(
        modifier = Modifier.fillMaxWidth().height(60.dp).padding(top = 20.dp),
        contentAlignment = Alignment.Center
    ) {
        Text("Counter: ${counter.value}")
    }

    Box(
        modifier = Modifier.fillMaxSize(),
        contentAlignment = Alignment.Center
    ) {
        Column(
            modifier = Modifier.padding(top = 80.dp, bottom = 20.dp)
        ) {
            Button("1. Compose Button: increment", inc)
            Spacer(modifier = Modifier.height(20.dp))

            SwingPanel(
                background = Color.LightGray,
                modifier = Modifier.size(270.dp, 90.dp),
                factory = {
                    JPanel().apply {
                        layout = BoxLayout(this, BoxLayout.Y_AXIS)
                        add(actionButton("1. Swing Button: decrement", dec))
                        add(actionButton("2. Swing Button: decrement", dec))
                        add(actionButton("3. Swing Button: decrement", dec))
                    }
                }
            )

            Spacer(modifier = Modifier.height(20.dp))
            Button("2. Compose Button: increment", inc)
        }
    }
}

@Composable
fun Button(text: String = "", action: (() -> Unit)? = null) {
    Button(
        modifier = Modifier.size(270.dp, 30.dp),
        onClick = { action?.invoke() }
    ) {
        Text(text)
    }
}

fun actionButton(
    text: String,
    action: () -> Unit
): JButton {
    val button = JButton(text)
    button.alignmentX = Component.CENTER_ALIGNMENT
    button.addActionListener { action() }

    return button
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="factory = { JPanel().apply { layout = BoxLayout(this, BoxLayout.Y_AXIS)"}

<img src="compose-desktop-swingpanel.animated.gif" alt="SwingPanel" preview-src="compose-desktop-swingpanel.png" width="600"/>

### Update Swing components when Compose state changes

To keep a Swing component up to date, provide an `update: (T) -> Unit` callback, which is invoked whenever the composable 
state changes or the layout is inflated.
The following code sample demonstrates how to update a Swing component within a `SwingPanel` whenever the composable state changes:

```kotlin
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.width
import androidx.compose.material.Button
import androidx.compose.material.Text
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.ui.awt.SwingPanel
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import androidx.compose.ui.window.application
import androidx.compose.ui.window.Window
import androidx.compose.ui.window.rememberWindowState
import java.awt.BorderLayout
import javax.swing.JPanel
import javax.swing.JLabel

val swingLabel = JLabel()

fun main() = application {
    Window(
        onCloseRequest = ::exitApplication,
        state = rememberWindowState(width = 400.dp, height = 200.dp),
        title = "SwingLabel"
    ) {
        val clicks = remember { mutableStateOf(0) }
        Column(
            modifier = Modifier.fillMaxSize().padding(20.dp),
            horizontalAlignment = Alignment.CenterHorizontally
        ) {
            SwingPanel(
                modifier = Modifier.fillMaxWidth().height(40.dp),
                factory = {
                    JPanel().apply {
                        add(swingLabel, BorderLayout.CENTER)
                    }
                },
                update = {
                    swingLabel.text = "SwingLabel clicks: ${clicks.value}"
                }
            )
            Spacer(modifier = Modifier.height(40.dp))
            Row (
                modifier = Modifier.height(40.dp),
                verticalAlignment = Alignment.CenterVertically
            ) {
                Button(onClick = { clicks.value++ }) {
                    Text(text = "Increment")
                }
                Spacer(modifier = Modifier.width(20.dp))
                Button(onClick = { clicks.value-- }) {
                    Text(text = "Decrement")
                }
            }
        }
    }
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="factory = { JPanel().apply { add(swingLabel, BorderLayout.CENTER)} }, update = {"}

<img src="compose-desktop-swinglabel.animated.gif" alt="SwingLabel" preview-src="compose-desktop-swinglabel.png" width="600"/>

### Experimental interop blending

By default, the interop view implemented using the `SwingPanel` wrapper is rectangular and in the foreground, 
on top of any Compose Multiplatform components. To make popup elements easier to use, we introduced experimental support for 
interop blending.

> Interop blending is [Experimental](supported-platforms.md#compose-multiplatform-ui-framework-stability-levels),
> and you should use it only for evaluation purposes.
>
{style="warning"}

To enable this experimental feature, set the `compose.interop.blending` system property to `true`.
The property must be enabled before executing any Compose code in your application,
so set it via the `-Dcompose.interop.blending=true` command-line JVM argument or
use `System.setProperty()` at the entry point:

```kotlin
fun main() {
    System.setProperty("compose.interop.blending", "true")
    ...
}
```

With interop blending enabled, you can rely on Swing in the following use cases:

* **Clipping**. You're no longer limited by a rectangular shape: the `clip` and `shadow` modifiers work correctly with `SwingPanel`.
* **Overlapping**. It is possible to draw any Compose Multiplatform content on top of a `SwingPanel` and interact with it as usual.

For details and known limitations, see the [description on GitHub](https://github.com/JetBrains/compose-multiplatform-core/pull/915).

## Layout with nested Swing and Compose Multiplatform components

With interoperability, you can combine Swing and Compose Multiplatform in both ways: adding Swing components to a Compose Multiplatform
application and adding Compose Multiplatform components to a Swing application. If you want to nest several components and 
freely combine approaches, this scenario is also supported.

The following code sample demonstrates how to add a `SwingPanel` to a `ComposePanel`, which is already inside another `SwingPanel`,
creating a Swing-Compose Multiplatform-Swing structure:

```kotlin
import androidx.compose.foundation.*
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material.*
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.*
import androidx.compose.runtime.*
import androidx.compose.ui.awt.*
import androidx.compose.ui.*
import androidx.compose.ui.draw.*
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.graphics.vector.ImageVector
import androidx.compose.ui.window.*
import androidx.compose.ui.unit.*
import java.awt.BorderLayout
import java.awt.Dimension
import java.awt.Insets
import javax.swing.*
import javax.swing.border.EmptyBorder

val Gray = java.awt.Color(64, 64, 64)
val DarkGray = java.awt.Color(32, 32, 32)
val LightGray = java.awt.Color(210, 210, 210)

data class Item(
    val text: String,
    val icon: ImageVector,
    val color: Color,
    val state: MutableState<Boolean> = mutableStateOf(false)
)
val panelItemsList = listOf(
    Item(text = "Person", icon = Icons.Filled.Person, color = Color(10, 232, 162)),
    Item(text = "Favorite", icon = Icons.Filled.Favorite, color = Color(150, 232, 150)),
    Item(text = "Search", icon = Icons.Filled.Search, color = Color(232, 10, 162)),
    Item(text = "Settings", icon = Icons.Filled.Settings, color = Color(232, 162, 10)),
    Item(text = "Close", icon = Icons.Filled.Close, color = Color(232, 100, 100))
)
val itemSize = 50.dp

fun java.awt.Color.toCompose(): Color {
    return Color(red, green, blue)
}

fun main() = application {
    Window(
        onCloseRequest = ::exitApplication,
        state = rememberWindowState(width = 500.dp, height = 500.dp),
        title = "Layout"
    ) {
        Column(
            modifier = Modifier.fillMaxSize().background(color = Gray.toCompose()).padding(20.dp),
            horizontalAlignment = Alignment.CenterHorizontally
        ) {
            Text(text = "Compose Area", color = LightGray.toCompose())
            Spacer(modifier = Modifier.height(40.dp))
            SwingPanel(
                background = DarkGray.toCompose(),
                modifier = Modifier.fillMaxSize(),
                factory = {
                    ComposePanel().apply {
                        setContent {
                            Box {
                                SwingPanel(
                                    modifier = Modifier.fillMaxSize(),
                                    factory = { SwingComponent() }
                                )
                                Box (
                                    modifier = Modifier.align(Alignment.TopStart)
                                        .padding(start = 20.dp, top = 80.dp)
                                        .background(color = DarkGray.toCompose())
                                ) {
                                    SwingPanel(
                                        modifier = Modifier.size(itemSize * panelItemsList.size, itemSize),
                                        factory = {
                                            ComposePanel().apply {
                                                setContent {
                                                    ComposeOverlay()
                                                }
                                            }
                                        }
                                    )
                                }
                            }
                        }
                    }
                }
            )
        }
    }
}

fun SwingComponent() : JPanel {
    return JPanel().apply {
        background = DarkGray
        border = EmptyBorder(20, 20, 20, 20)
        layout = BorderLayout()
        add(
            JLabel("TextArea Swing Component").apply {
                foreground = LightGray
                verticalAlignment = SwingConstants.NORTH
                horizontalAlignment = SwingConstants.CENTER
                preferredSize = Dimension(40, 160)
            },
            BorderLayout.NORTH
        )
        add(
            JTextArea().apply {
                background = LightGray
                lineWrap = true
                wrapStyleWord = true
                margin = Insets(10, 10, 10, 10)
                text = "The five boxing wizards jump quickly. " +
                "Crazy Fredrick bought many very exquisite opal jewels. " +
                "Pack my box with five dozen liquor jugs.\n" +
                "Cozy sphinx waves quart jug of bad milk. " +
                "The jay, pig, fox, zebra and my wolves quack!"
            },
            BorderLayout.CENTER
        )
    }
}

@Composable
fun ComposeOverlay() {
    Box(
        modifier = Modifier.fillMaxSize().
            background(color = DarkGray.toCompose()),
        contentAlignment = Alignment.Center
    ) {
        Row(
            modifier = Modifier.background(
                shape = RoundedCornerShape(4.dp),
                color = Color.DarkGray.copy(alpha = 0.5f)
            )
        ) {
            for (item in panelItemsList) {
                SelectableItem(
                    text = item.text,
                    icon = item.icon,
                    color = item.color,
                    selected = item.state
                )
            }
        }
    }
}

@Composable
fun SelectableItem(
    text: String,
    icon: ImageVector,
    color: Color,
    selected: MutableState<Boolean>
) {
    Box(
        modifier = Modifier.size(itemSize)
            .clickable { selected.value = !selected.value },
        contentAlignment = Alignment.Center
    ) {
        Column(
            modifier = Modifier.alpha(if (selected.value) 1.0f else 0.5f),
            horizontalAlignment = Alignment.CenterHorizontally
        ) {
            Icon(modifier = Modifier.size(32.dp), imageVector = icon, contentDescription = null, tint = color)
            Text(text = text, color = Color.White, fontSize = 10.sp)
        }
    }
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="fun SwingComponent() : JPanel { return JPanel().apply {"}

<img src="compose-desktop-swing-layout.animated.gif" alt="Swing layout" preview-src="compose-desktop-swing-layout.png" width="600"/>

## What's next?

Explore the tutorials about [other desktop-specific components](https://github.com/JetBrains/compose-multiplatform/tree/master/tutorials#desktop).
