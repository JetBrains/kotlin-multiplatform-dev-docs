[//]: # (title: Popups)

A `Popup()` is a floating container that renders its content on top of the current UI.
Unlike a modal dialog, a popup is non-modal, has no scrim, and is not centered by default, 
which makes it suitable for dropdowns, tooltips, and custom overlays anchored to a component.

`Popup()` and its `PopupProperties` are part of the common API. However,
some properties are only available in platform-specific source sets. For example, on iOS,
`PopupProperties` provides `usePlatformInsets`, which limits the popup's content to the platform insets (the safe area).

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

## What's next

For more details, see the
[`Popup()`](https://developer.android.com/reference/kotlin/androidx/compose/ui/window/Popup.composable) API
reference in the Jetpack Compose documentation.