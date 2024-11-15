[//]: # (title: Support for desktop accessibility features)

Compose Multiplatform builds on top of [Jetpack Compose](https://developer.android.com/jetpack/compose), making most accessibility features available in common
code across all platforms. The current status of accessibility support on desktop is as follows:

| Platform | Accessibility status             |
|----------|----------------------------------|
| MacOS    | Fully supported                  |
| Windows  | Supported via Java Access Bridge |
| Linux    | Not supported                    | 

## Enabling accessibility on Windows

Accessibility on Windows is provided via Java Access Bridge, which is disabled by default.
To enable it, run the following command:

```Console
%\JAVA_HOME%\bin\jabswitch.exe /enable
```

## Example: Custom button with semantic rules

Let's create a simple app with a custom button and specify explanatory text for screen reader tools.
With a screen reader enabled, you will hear the "Click to increment value" text from the button description:

```kotlin
import androidx.compose.foundation.*
import androidx.compose.foundation.layout.*
import androidx.compose.material.Text
import androidx.compose.runtime.*
import androidx.compose.ui.*
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.semantics.*
import androidx.compose.ui.unit.*
import androidx.compose.ui.window.*

fun main() = singleWindowApplication(
    title = "Custom Button", state = WindowState(size = DpSize(300.dp, 200.dp))
) {
    var count by remember { mutableStateOf(0) }

    Box(modifier = Modifier.padding(50.dp)) {
        Box(modifier = Modifier
            .background(Color.LightGray)
            .fillMaxSize()
            .clickable { count += 1 }
            // Use text from the content  
            .semantics(mergeDescendants = true) {
                // This is a button
                role = Role.Button
                // Add some help text to button
                contentDescription = "Click to increment value"
            }
        ) {
            val text = when (count) {
                0 -> "Click Me!"
                1 -> "Clicked"
                else -> "Clicked $count times"
            }
            Text(text, modifier = Modifier.align(Alignment.Center), fontSize = 24.sp)
        }
    }
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title=".clickable { count += 1 } .semantics(mergeDescendants = true)"}

To test the application's accessibility on macOS, you can use [Accessibility Inspector](https://developer.apple.com/documentation/accessibility/accessibility-inspector)
(**Xcode** | **Open Developer Tool** | **Accessibility Inspector**):

<img src="compose-desktop-accessibility-macos.png" alt="Accessibility inspector on mcOS" width="700"/>

On Windows, you can use the **Show Speech History** feature in [JAWS](https://www.freedomscientific.com/Products/Blindness/JAWS) 
or the **Speech Viewer** in [NVDA](https://www.nvaccess.org/):

<img src="compose-desktop-accessibility.png" alt="Accessibility on Windows" width="600"/>


For more examples, refer to the [Accessibility in Jetpack Compose](https://developer.android.com/develop/ui/compose/accessibility) guide.

## What's next?

Explore the tutorials about [other desktop components](https://github.com/JetBrains/compose-multiplatform/tree/master/tutorials#desktop).