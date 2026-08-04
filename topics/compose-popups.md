[//]: # (title: Popups)

<web-summary>Learn how to create, position, and customize popups in Compose Multiplatform for dropdowns, tooltips, and menus.</web-summary>

A popup in Compose Multiplatform is a floating container that renders its content on top of the current UI within the same window.

Unlike the multiplatform `Dialog()` API, a `Popup()` is non-modal. 
A dialog in Compose Multiplatform acts as a modal container that takes focus, centers its content, 
and uses a dimmed scrim to block interaction with the rest of the UI.
A popup, on the other hand, has no scrim, does not restrict its width, and allows users to continue interacting with the underlying UI.
It is not centered by default and requires additional arguments to anchor it to a component.

Use [`Dialog()`](https://developer.android.com/reference/kotlin/androidx/compose/ui/window/Dialog.composable)
when you need to interrupt the user and require a decision before they continue, 
for example, react to confirmations, alerts, or short forms. For separate OS-level dialogs on desktop, 
see [`DialogWindow()`](compose-desktop-top-level-windows-management.md#show-dialogs).
Use `Popup()` for lightweight, non-blocking overlays that stay anchored to a component inside the current window, 
such as dropdowns, tooltips, and menus.

## Position a popup

To position a popup, use either an `alignment` and `offset`, or a custom
`PopupPositionProvider` for anchored placement.

For a simple alignment:

```kotlin
var isPopupOpen by remember { mutableStateOf(false) }

Box(Modifier.padding(24.dp)) {
    Button(onClick = { isPopupOpen = !isPopupOpen }) {
        Text("Toggle popup")
    }

    if (isPopupOpen) {
        Popup(
            // Positions the popup relative to the button
            alignment = Alignment.TopStart,
            // Shifts the popup by (x, y) in pixels
            offset = IntOffset(30, 70),
            // Hides the popup when it is dismissed,
            // for example, when the user clicks outside it
            onDismissRequest = { isPopupOpen = false }
        ) {
            Box(
                Modifier
                    .background(Color.LightGray, RoundedCornerShape(4.dp))
                    .padding(12.dp)
            ) {
                Text("Popup content on top of UI")
            }
        }
    }
}
```

For anchored placement, use a `PopupPositionProvider`:

```kotlin
var isPopupOpen by remember { mutableStateOf(false) }

val belowAnchor = remember {
    object : PopupPositionProvider {
        override fun calculatePosition(
            anchorBounds: IntRect,
            windowSize: IntSize,
            layoutDirection: LayoutDirection,
            popupContentSize: IntSize
        ) = IntOffset(x = anchorBounds.left - 20, y = anchorBounds.bottom - 20)
    }
}

Column(Modifier.padding(24.dp)) {
    Box {
        Button(onClick = { isPopupOpen = !isPopupOpen }) {
            Text("Toggle menu")
        }

        if (isPopupOpen) {
            Popup(
                popupPositionProvider = belowAnchor,
                onDismissRequest = { isPopupOpen = false }
            ) {
                Box(
                    Modifier
                        .shadow(4.dp, RoundedCornerShape(4.dp))
                        .background(Color.White, RoundedCornerShape(4.dp))
                        .padding(12.dp)
                ) {
                    Text("Anchored to the button")
                }
            }
        }
    }
}
```

## Customize behavior

With `PopupProperties`, you can control how the popup handles focus and dismissal:

* `focusable` determines whether the popup receives key events, disabled by default.
* `dismissOnBackPress` dismisses the popup on Android's back button or the **Esc** key on desktop, 
  enabled by default. Requires `focusable = true`.
* `dismissOnClickOutside` dismisses the popup when the user presses outside its bounds, enabled by default.

`Popup()` and its `PopupProperties` are part of the common API. 
However, some properties are not available in the common source set. 
For example, `usePlatformInsets` is available on iOS, where it limits the popup's content to the platform insets (the safe area).

## What's next

For full API details, see the references in the Jetpack Compose documentation:
* [`Popup()`](https://developer.android.com/reference/kotlin/androidx/compose/ui/window/Popup.composable)
* [`Dialog()`](https://developer.android.com/reference/kotlin/androidx/compose/ui/window/Dialog.composable)