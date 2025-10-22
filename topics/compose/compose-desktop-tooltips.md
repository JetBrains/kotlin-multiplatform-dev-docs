[//]: # (title: Tooltips)

You can add a tooltip to any component using the `TooltipArea` composable. `TooltipArea` is similar to the `Box`
component and can show a tooltip.

The `TooltipArea` composable has the following main parameters:

* `tooltip`, composable content of the tooltip.
* `tooltipPlacement`, defines the tooltip position. You can specify an anchor (the mouse cursor or the component),
  an offset, and an alignment.
* `delayMillis`, time in milliseconds after which the tooltip is shown. The default value is 500 ms.

The following example shows how to create a simple window that contains a list of buttons, 
each wrapped in a `TooltipArea`. When you hover over a button, a tooltip with the button's name will appear.
Add this code to the `main.kt` file in `composeApp/src/jvmMain/kotlin`:

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
import androidx.compose.ui.window.WindowState
import androidx.compose.ui.window.singleWindowApplication

@OptIn(ExperimentalFoundationApi::class)
fun main() = singleWindowApplication(
    WindowState(width = 300.dp, height = 350.dp),
    title = "Tooltip Example"
) {
    val buttons = listOf("Button A", "Button B", "Button C", "Button D", "Button E", "Button F")
    Column(Modifier.fillMaxSize(), Arrangement.spacedBy(5.dp)) {
        buttons.forEachIndexed { index, name ->
            // Wrap the button in TooltipArea
            TooltipArea(
                tooltip = {
                    // Composable tooltip content:
                    Surface(
                        modifier = Modifier.shadow(4.dp),
                        color = Color(255, 255, 210),
                        shape = RoundedCornerShape(4.dp)
                    ) {
                        Text(
                            text = "Tooltip for $name",
                            modifier = Modifier.padding(10.dp)
                        )
                    }
                },
                modifier = Modifier.padding(start = 40.dp),
                delayMillis = 600, // In milliseconds
                tooltipPlacement = TooltipPlacement.CursorPoint(
                    alignment = Alignment.BottomEnd,
                    offset = if (index % 2 == 0) DpOffset(
                        (-16).dp,
                        0.dp
                    ) else DpOffset.Zero // Tooltip offset
                )
            ) {
                Button(onClick = {}) { Text(text = name) }
            }
        }
    }
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="TooltipArea(tooltip = { Surface( "}

<img src="compose-desktop-tooltips.animated.gif" alt="Tooltips" width="288" preview-src="compose-desktop-tooltips.png"/>

## What's next?

Explore the tutorials about [other desktop components](https://github.com/JetBrains/compose-multiplatform/tree/master/tutorials#desktop).