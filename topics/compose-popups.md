[//]: # (title: Popups)

A `Popup()` is a floating container that renders its content on top of the current UI.

Unlike the `Dialog()` API — a modal container that takes focus, centers its
content, and uses a dimmed scrim to block interaction with the rest of the UI — a `Popup()` is non-modal. 
It has no scrim, does not restrict its width, and allows users to continue interacting with the underlying UI.
A popup is not centered by default and requires explicit positioning to anchor it to a component.

Use `Dialog()` when you need to interrupt the user and require a decision before they continue, 
such as confirmations, alerts, or short forms. 
Use `Popup()` for lightweight, non-blocking overlays that stay anchored to a component, 
such as dropdowns, tooltips, and menus.

## Positioning a popup

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
            offset = IntOffset(0, 120),
            // Hides the popup when the user presses outside of it
            onDismissRequest = { isPopupOpen = false },
            // Makes the popup focusable so it can receive key events
            properties = PopupProperties(focusable = true)
        ) {
            Box(
                Modifier
                    .background(Color.LightGray, RoundedCornerShape(4.dp))
                    .padding(12.dp)
            ) {
                Text("Popup content")
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
        ) = IntOffset(x = anchorBounds.left, y = anchorBounds.bottom)
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
                onDismissRequest = { isPopupOpen = false },
                properties = PopupProperties(focusable = true)
            ) {
                Box(
                    Modifier
                        .shadow(4.dp, RoundedCornerShape(4.dp))
                        .background(Color.White, RoundedCornerShape(4.dp))
                        .padding(12.dp)
                ) {
                    Text("Anchored below the button")
                }
            }
        }
    }
}
```

## Customizing behavior

With `PopupProperties`, you can control how the popup handles focus and dismissal:

* `focusable` determines whether the popup receives key events, disabled by default.
  It must be enabled for `dismissOnBackPress` to work.
* `dismissOnBackPress` dismisses the popup on Android's back button or the **Esc** key on desktop, 
  enabled by default. Requires `focusable = true`.
* `dismissOnClickOutside` dismisses the popup when the user presses outside its bounds, enabled by default.

`Popup()` and its `PopupProperties` are part of the common API. However,
some properties are only available in platform-specific source sets. For example, on iOS,
`PopupProperties` provides `usePlatformInsets`, which limits the popup's content to the platform insets (the safe area).

## What's next

For full API details, see the references in the Jetpack Compose documentation:
* [`Popup()`](https://developer.android.com/reference/kotlin/androidx/compose/ui/window/Popup.composable)
* [`Dialog()`](https://developer.android.com/reference/kotlin/androidx/compose/ui/window/Dialog.composable)