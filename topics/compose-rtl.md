# RTL languages

Compose Multiplatform provides support for Right-to-Left (RTL) languages such as Arabic, Hebrew, and Persian.
The framework automatically handles most RTL requirements and adjusts layouts, alignment, and text input behavior 
according to the system's locale settings when RTL languages are used.

## Layout mirroring

When the system locale is set to an RTL language, Compose Multiplatform automatically mirrors most UI components.
Adjustments include changes to paddings, alignments, and component positions:

* **Paddings, margins, and alignments**  
   Default paddings and alignments are reversed. For example, <br/>
 `Modifier.padding(start: Dp, top: Dp, end: Dp, bottom: Dp)` will automatically adjust to
 `Modifier.padding(end: Dp, top: Dp, start: Dp, bottom: Dp)`.

* **Component alignment**  
   For UI elements like text, navigation items, and icons, the default `Start` alignment becomes `End` in RTL mode.

* **Horizontally scrollable lists**  
   Horizontal lists reverse their item alignment and scroll direction.

* **Button positioning**  
   Common UI patterns, such as the position of **Cancel** and **Confirm** buttons, adjust to RTL expectations.

## Forcing layout direction

Some UI elements, such as logos or icons, may need to keep their original orientation regardless of layout direction.
You can explicitly set the layout direction for an entire app or individual components, 
overriding the system's default locale-based layout behavior.

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

## Handling text input in RTL layouts

Compose Multiplatform provides support for various scenarios of text input in RTL layouts,
including mixed-direction content, special characters, numbers, and emojis. 

When you design an application that supports an RTL layout, consider the following aspects.
While these features work out-of-the-box with Compose Multiplatform, testing them can help you identify potential localization issues.

### Cursor behavior

The cursor should behave intuitively within RTL layouts, aligning with the logical direction of characters. For example:

* When typing in Arabic, the cursor moves right-to-left, but inserting LTR content follows left-to-right behavior.
* Operations like text selection, deletion, and insertion respect the text's natural directional flow.

### BiDi text

Compose Multiplatform uses the [Unicode Bidirectional Algorithm](https://www.w3.org/International/articles/inline-bidi-markup/uba-basics)
to manage and render bidirectional (BiDi) text, aligning punctuation and numbers.

The following line displays text in the expected visual order: punctuation and numbers aligned correctly, 
Arabic script flows from right to left, and English is from left to right.

```kotlin
Text("مرحباً 123, this is a test.")
```

Compose Multiplatform also ensures proper alignment and spacing in complex BiDi cases,
including multi-line wrapping and nesting of BiDi content.

### Numbers and emojis

Numbers should be displayed consistently based on the direction of the surrounding text. 
Arabic numerals align naturally in RTL text, and Western numerals follow typical LTR behavior.

Emojis should adapt to both RTL and LTR contexts, maintaining proper alignment and spacing within the text.

The following test line includes an emoji, Arabic numeral, and bidirectional text:

```kotlin
Text("⚠️ تحذير: هذا النص يضم Symbols and عدد ٥!")
```

## Accessibility in RTL layouts

Compose Multiplatform supports accessibility features for RTL layouts, 
including proper text direction and order for screen readers and handling of gestures.

### Screen readers

Screen readers automatically adapt to RTL layouts, keeping the logical reading order for users:

* RTL text is read from right to left, and mixed-direction text follows standard BiDi rules.
* Punctuation and numbers are announced in the correct sequence.

However, in complex layouts, screen readers cannot determine the correct reading order without properly defined traversal semantics.
Learn how to set traversal indexes in the [Accessibility](whats-new-compose-180.md#accessibility-for-container-views) section.

[//]: # (todo: replace accessibility link)

### Focus-based navigation

Focus navigation in RTL layouts follows the layout's mirrored structure:

* Focus moves from right to left and top to bottom, following the natural flow of RTL content.
* Gestures like swipes or taps are automatically adjusted to the mirrored layout.

For a complex layout, you can define traversal semantics to ensure correct navigation between different traversal groups 
with the swipe-up or swipe-down accessibility gestures.
Learn how to define traversal semantics in the [Accessibility](whats-new-compose-180.md#accessibility-for-container-views) section.
