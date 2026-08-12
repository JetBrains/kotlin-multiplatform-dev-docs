[//]: # (title: Tray and notifications)
[//]: # (description: Learn how to add an application icon to the system tray and send system notifications in Compose Multiplatform for desktop.)

In Compose Multiplatform for desktop, you can add an application icon to the system tray and send system
notifications through it.

<include from="compose-desktop-scrollbars.md" element-id="desktop-snippets-intro"/>

## System tray

Use the `Tray` composable to add an application icon to the system tray. `Tray` is available in the scope of the
`application()` function, so it can be called next to the application windows or on its own.

The `Tray` composable has the following parameters:

* `icon` – the `Painter` that draws the tray icon.
* `menu` – the contents of the tray menu. The menu opens on a right-click on Windows and on a left-click on macOS.
  If you don't add any items, the menu doesn't appear.
* `state` – the `TrayState` used to send notifications.
* `tooltip` – the hint shown when the user hovers over the icon.
* `onAction` – the action triggered by clicking the icon: a double click on Windows and a right-click on macOS.

The following example creates an application icon in the tray with three menu items: 
* **Increment value** that changes the state shown in the window
* **Send notification**
* **Exit** that closes the application

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
import androidx.compose.ui.geometry.Offset
import androidx.compose.ui.geometry.Size
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.graphics.drawscope.DrawScope
import androidx.compose.ui.graphics.painter.Painter
import androidx.compose.ui.window.Tray
import androidx.compose.ui.window.Window
import androidx.compose.ui.window.application
import androidx.compose.ui.window.rememberNotification
import androidx.compose.ui.window.rememberTrayState

fun main() = application {
    var count by remember { mutableStateOf(0) }
    var isOpen by remember { mutableStateOf(true) }

    if (isOpen) {
        val trayState = rememberTrayState()
        val notification = rememberNotification("Notification", "Message from MyApp!")

        Tray(
            state = trayState,
            icon = TrayIcon,
            menu = {
                Item(
                    "Increment value",
                    onClick = {
                        count++
                    }
                )
                Item(
                    "Send notification",
                    onClick = {
                        trayState.sendNotification(notification)
                    }
                )
                Item(
                    "Exit",
                    onClick = {
                        isOpen = false
                    }
                )
            }
        )

        Window(
            onCloseRequest = {
                isOpen = false
            },
            icon = MyAppIcon
        ) {
            // Window content:
            Box(
                modifier = Modifier.fillMaxSize(),
                contentAlignment = Alignment.Center
            ) {
                Text(text = "Value: $count")
            }
        }
    }
}

object MyAppIcon : Painter() {
    override val intrinsicSize = Size(256f, 256f)

    override fun DrawScope.onDraw() {
        drawOval(Color.Green, Offset(size.width / 4, 0f), Size(size.width / 2f, size.height))
        drawOval(Color.Blue, Offset(0f, size.height / 4), Size(size.width, size.height / 2f))
        drawOval(Color.Red, Offset(size.width / 4, size.height / 4), Size(size.width / 2f, size.height / 2f))
    }
}

object TrayIcon : Painter() {
    override val intrinsicSize = Size(256f, 256f)

    override fun DrawScope.onDraw() {
        drawOval(Color(0xFFFFA500))
    }
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="Tray(state = trayState, icon = TrayIcon, menu = { Item( "}

<img src="compose-desktop-tray.animated.gif" alt="Tray" width="600"/>

Not every desktop environment has a system tray. If the platform doesn't support one, `Tray` outputs an error to the
standard error stream instead of throwing an exception. Check the `isTraySupported` property before you show
tray-related options in your application.

### Tray without a window

An application doesn't need a window to have a tray icon. If only the `Tray` function is called, the application lives 
entirely in the system tray:

```kotlin
import androidx.compose.ui.geometry.Size
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.graphics.drawscope.DrawScope
import androidx.compose.ui.graphics.painter.Painter
import androidx.compose.ui.window.Tray
import androidx.compose.ui.window.application

fun main() = application {
    Tray(
        icon = TrayIcon,
        menu = {
            Item(
                "Exit",
                onClick = ::exitApplication
            )
        }
    )
}

object TrayIcon : Painter() {
    override val intrinsicSize = Size(256f, 256f)

    override fun DrawScope.onDraw() {
        drawOval(Color(0xFFFFA500))
    }
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="Tray(icon = TrayIcon, menu = { Item( "}

Because there is no window to close, `exitApplication()` should be called from a menu item.

## Notifications

To send a system notification, create it with `rememberNotification()` and pass it to `TrayState.sendNotification()`,
as the [system tray example](#system-tray) does. Notifications are delivered through the `TrayState` passed to a `Tray` composable. 
If the state isn't attached to a tray, the notification is lost.

A notification consists of a title, a message, and a type. The type defines the icon and the sound of the notification:

* `Notification.Type.None` – a simple notification, the default option
* `Notification.Type.Info`
* `Notification.Type.Warning`
* `Notification.Type.Error`

The specific assets used for the icon and the sound depend on the platform.

> To test the notifications on macOS, the app must be packaged. Otherwise, the notifications will not be shown.
>
{style="warning"}

## What's next

* Learn how to add a [menu bar](compose-desktop-menu-bar.md) to a window.
* Explore the tutorials about [other desktop components](compose-desktop-components.md).
