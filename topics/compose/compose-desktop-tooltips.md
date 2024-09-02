[//]: # (title: Tooltips)

You can add a tooltip to any component using the `TooltipArea` composable. `TooltipArea` is similar to the `Box`
component that can show a tooltip:

```kotlin
import androidx.compose.foundation.ExperimentalFoundationApi
import androidx.compose.foundation.TooltipArea
import androidx.compose.foundation.TooltipPlacement
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material.Button
import androidx.compose.material.Surface
import androidx.compose.material.Text
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.shadow
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.unit.DpOffset
import androidx.compose.ui.unit.dp
import androidx.compose.ui.window.Window
import androidx.compose.ui.window.application
import androidx.compose.ui.window.rememberWindowState

@OptIn(ExperimentalFoundationApi::class)
fun main() = application {
    Window(
        onCloseRequest = ::exitApplication,
        title = "Tooltip Example",
        state = rememberWindowState(width = 300.dp, height = 300.dp)
    ) {
        val buttons = listOf("Button A", "Button B", "Button C", "Button D", "Button E", "Button F")
        Column(Modifier.fillMaxSize(), Arrangement.spacedBy(5.dp)) {
            buttons.forEachIndexed { index, name ->
                // Wrap the button in BoxWithTooltip
                TooltipArea(
                    tooltip = {
                        // Composable tooltip content:
                        Surface(
                            modifier = Modifier.shadow(4.dp),
                            color = Color(255, 255, 210),
                            shape = RoundedCornerShape(4.dp)
                        ) {
                            Text(
                                text = "Tooltip for ${name}",
                                modifier = Modifier.padding(10.dp)
                            )
                        }
                    },
                    modifier = Modifier.padding(start = 40.dp),
                    delayMillis = 600, // In milliseconds
                    tooltipPlacement = TooltipPlacement.CursorPoint(
                        alignment = Alignment.BottomEnd,
                        offset = if (index % 2 == 0) DpOffset(-16.dp, 0.dp) else DpOffset.Zero // Tooltip offset
                    )
                ) {
                    Button(onClick = {}) { Text(text = name) }
                }
            }
        }
    }
}
```
{initial-collapse-state="collapsed" collapsed-title="TooltipArea(tooltip = { Surface( "}

<img src="compose-desktop-tooltips.png" alt="Tooltips" width="288" animated="true"/>

The main parameters of the `TooltipArea` composable are:

* `tooltip`, composable content for a tooltip.
* `tooltipPlacement`, defines the tooltip placement. You can specify an anchor (the mouse cursor or the component),
  an offset, and an alignment.
* `delay`, the time in milliseconds after which the tooltip is shown. The default value is 500 ms.

## What's next?

You can also explore the tutorials about [other desktop components](https://github.com/JetBrains/compose-multiplatform/tree/master/tutorials#desktop).