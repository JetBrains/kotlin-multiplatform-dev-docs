# RTL languages

Compose Multiplatform provides support for Right-to-Left (RTL) languages such as Arabic, Hebrew, and Persian. 
The framework handles most RTL requirements automatically and adjusts layouts, alignment, and text input 
behavior when the system's locale is set to an RTL language.

## Layout mirroring

When the system locale is set to an RTL language, Compose Multiplatform automatically mirrors most UI components.
Adjustments include changes to paddings, alignments, and component positions:

* **Paddings, Margins, and Alignments**  
   Default paddings and alignments are flipped. For example, `Modifier.padding(start: Dp, top: Dp, end: Dp, bottom: Dp)` will mirror to 
 `Modifier.padding(end: Dp, top: Dp, start: Dp, bottom: Dp)` automatically in RTL mode.

* **Component Alignment**  
   For UI elements like text, navigation items, and icons, the default `Start` alignment becomes `End` in RTL mode.

* **Horizontally Scrollable Lists**  
   Horizontal lists reverse their item alignment and scroll direction.

* **Button Positioning**  
   Common UI patterns, such as the position of **Cancel** and **Confirm** buttons, adjust to RTL expectations.

## Forcing layout direction

Some UI elements, such as logos or icons, may need to keep their original orientation regardless of layout direction.
The layout direction can be set explicitly for an entire app or individual components, overriding the system's default locale behavior.

To exclude an element from automatic mirroring, use `LayoutDirection.Rtl` or `LayoutDirection.Ltr` to enforce a specific orientation:

```kotlin
CompositionLocalProvider(LocalLayoutDirection provides LayoutDirection.Ltr) {
    Column(modifier = Modifier.fillMaxWidth()) {
        // Components in this block will be laid out in LTR
        Text("LTR Latin")
        TextField("Hello world\nHello world")
    }
}
```

The `CompositionLocalProvider()` ensures that the specified layout direction applies to all child components in the composition.

## Text input in RTL layouts

Compose Multiplatform's support for text input includes special characters, mixed bidirectional (BiDi) text, and handling various edge cases.

* **Cursor behavior**

Text cursor behavior is fully RTL-aware. Movement, insertion, and deletion respect the directionality of the characters.

* **Bidirectional content handling**

Compose Multiplatform uses the Unicode Bidirectional Algorithm to render mixed RTL and LTR content correctly. 
For example, when combining Arabic and English, punctuation and numbers align properly:

```kotlin
Text(text = "مرحبا Hello!")
```

The above example will display Arabic and English in the expected visual order, following the Unicode rules for BiDi text.

* **Emojis** behave consistently in both RTL and LTR contexts.

* **Numbers** (Arabic or Western numerals) align properly in the context of surrounding RTL or LTR text.

## Accessibility in RTL layouts

Compose Multiplatform supports accessibility features for RTL layouts, including screen readers and proper text direction handling for gestures.

* **Text direction and alignment**  
  Text adapts to the correct logical direction for screen readers.

* **Navigational order**  
  Focus-based navigation adjusts automatically to reflect the mirrored element order in RTL layouts.