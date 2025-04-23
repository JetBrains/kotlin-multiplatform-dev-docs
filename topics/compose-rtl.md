# Working with RTL languages
<primary-label ref="EAP"/>

Compose Multiplatform provides support for Right-to-Left (RTL) languages such as Arabic, Hebrew, and Persian.
The framework automatically handles most RTL requirements and adjusts layouts, alignment, and text input behavior 
according to the system's locale settings when RTL languages are used.

## Layout mirroring

When the system locale is configured for an RTL language, Compose Multiplatform automatically mirrors most UI components.
Adjustments include changes to paddings, alignments, and component positions:

* **Paddings, margins, and alignments**  
   Default paddings and alignments are reversed. For example, <br/>
 `Modifier.padding(start = 1.dp, top = 2.dp, end = 3.dp, bottom = 4.dp)` will be ran as
 `Modifier.padding(start = 3.dp, top = 2.dp, end = 1.dp, bottom = 4.dp)`.

* **Component alignment**  
   For UI elements like text, navigation items, and icons, the default `Start` alignment becomes `End` in RTL mode.

* **Horizontally scrollable lists**  
   Horizontal lists reverse their item alignment and scroll direction.

* **Button positioning**  
   Common UI patterns, such as the position of **Cancel** and **Confirm** buttons, adjust to RTL expectations.

## Forcing layout direction

You may need to keep the original orientation of some UI elements, such as logos or icons, regardless of layout direction.
You can explicitly set the layout direction for an entire app or individual components, 
overriding the system's default locale-based layout behavior.

To exclude an element from automatic mirroring and enforce a specific orientation, 
you can use `LayoutDirection.Rtl` or `LayoutDirection.Ltr`.
To specify a layout direction within a scope, use the `CompositionLocalProvider()`, which ensures the layout direction 
applies to all child components in the composition:

```kotlin
CompositionLocalProvider(LocalLayoutDirection provides LayoutDirection.Ltr) {
    Column(modifier = Modifier.fillMaxWidth()) {
        // Components in this block will be laid out left-to-right
        Text("LTR Latin")
        TextField("Hello world\nHello world")
    }
}
```

## Handling text input in RTL layouts

Compose Multiplatform provides support for various scenarios of text input in RTL layouts,
including mixed-direction content, special characters, numbers, and emojis. 

When you design an application that supports an RTL layout, consider the following aspects.
Testing them can help you identify potential localization issues.

### Cursor behavior

The cursor should behave intuitively within RTL layouts, aligning with the logical direction of characters. For example:

* When typing in Arabic, the cursor moves right-to-left, but inserting LTR content follows left-to-right behavior.
* Operations like text selection, deletion, and insertion respect the text's natural directional flow.

### BiDi text

Compose Multiplatform uses the [Unicode Bidirectional Algorithm](https://www.w3.org/International/articles/inline-bidi-markup/uba-basics)
to manage and render bidirectional (BiDi) text, aligning punctuation and numbers.

The text should be displayed in the expected visual order: punctuation and numbers are aligned correctly, 
the Arabic script flows from right to left, and English flows from left to right.

The following test sample includes text with Latin and Arabic letters, and their bidirectional combination:

```kotlin
import androidx.compose.foundation.border
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.foundation.text.BasicTextField
import androidx.compose.material.MaterialTheme
import androidx.compose.material.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.CompositionLocalProvider
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.saveable.rememberSaveable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.platform.LocalLayoutDirection
import androidx.compose.ui.unit.LayoutDirection
import androidx.compose.ui.unit.dp
import org.jetbrains.compose.ui.tooling.preview.Preview

// Arabic text for "Hello World"
private val helloWorldArabic = "مرحبا بالعالم"

// Bidirectional text
private val bidiText = "Hello $helloWorldArabic world"

@Composable
@Preview
fun App() {
    MaterialTheme {
        LazyColumn(
            Modifier
                .fillMaxWidth()
                .padding(16.dp),
            horizontalAlignment = Alignment.CenterHorizontally
        ) {
            item {
                CompositionLocalProvider(LocalLayoutDirection provides LayoutDirection.Ltr) {
                    Column(modifier = Modifier.fillMaxWidth()) {
                        Text("Latin and BiDi in LTR")
                        TextField("Hello world")
                        TextField(bidiText)
                    }
                }
            }
            item {
                CompositionLocalProvider(LocalLayoutDirection provides LayoutDirection.Rtl) {
                    Column(modifier = Modifier.fillMaxWidth()) {
                        Text("Arabic and BiDi in RTL")
                        TextField(helloWorldArabic)
                        TextField(bidiText)
                    }
                }
            }
        }
    }
}

// Wrap function for BasicTextField() to reduce code duplication
@Composable
internal fun TextField(
    text: String = ""
) {
    val state = rememberSaveable { mutableStateOf(text) }

    BasicTextField(
        modifier = Modifier
            .border(1.dp, Color.LightGray, RoundedCornerShape(8.dp))
            .padding(8.dp),
        value = state.value,
        singleLine = false,
        onValueChange = { state.value = it },
    )
}
```
{default-state="collapsed" collapsible="true" collapsed-title="item { CompositionLocalProvider(LocalLayoutDirection provides LayoutDirection.Rtl) {"}

<img src="compose-rtl-bidi.png" alt="BiDi text" width="600"/>

Compose Multiplatform also ensures proper alignment and spacing in complex BiDi cases,
including multi-line wrapping and nesting of BiDi content.

### Numbers and emojis

Numbers should be displayed consistently based on the direction of the surrounding text. 
Eastern Arabic numerals align naturally in RTL text, and Western Arabic numerals follow typical LTR behavior.

Emojis should adapt to both RTL and LTR contexts, maintaining proper alignment and spacing within the text.

The following test sample includes emojis, Eastern and Western Arabic numerals, and bidirectional text:

```kotlin
import androidx.compose.foundation.border
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.lazy.LazyColumn
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.foundation.text.BasicTextField
import androidx.compose.material.MaterialTheme
import androidx.compose.runtime.Composable
import androidx.compose.runtime.CompositionLocalProvider
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.saveable.rememberSaveable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.platform.LocalLayoutDirection
import androidx.compose.ui.unit.LayoutDirection
import androidx.compose.ui.unit.dp
import org.jetbrains.compose.ui.tooling.preview.Preview

// Arabic text for "Hello World" with emojis
private val helloWorldArabic = "مرحبا بالعالم 🌎👋"

// Bidirectional text with numbers and emojis
private val bidiText = "67890 Hello $helloWorldArabic 🎉"

@Composable
@Preview
fun App() {
    MaterialTheme {
        LazyColumn(
            Modifier
                .fillMaxWidth()
                .padding(16.dp),
            horizontalAlignment = Alignment.CenterHorizontally
        ) {
            item {
                CompositionLocalProvider(LocalLayoutDirection provides LayoutDirection.Ltr) {
                    Column(modifier = Modifier.fillMaxWidth()) {
                        TextField("Hello world 👋🌎")
                        TextField("Numbers: 🔢12345")
                        TextField(bidiText)
                    }
                }
            }
            item {
                CompositionLocalProvider(LocalLayoutDirection provides LayoutDirection.Rtl) {
                    Column(modifier = Modifier.fillMaxWidth()) {
                        TextField(helloWorldArabic)
                        TextField("الأرقام: 🔢١٢٣٤٥")
                        TextField(bidiText)
                    }
                }
            }
        }
    }
}

// Wrap function for BasicTextField() to reduce code duplication
@Composable
internal fun TextField(
    text: String = ""
) {
    val state = rememberSaveable { mutableStateOf(text) }

    BasicTextField(
        modifier = Modifier
            .border(1.dp, Color.LightGray, RoundedCornerShape(8.dp))
            .padding(8.dp),
        value = state.value,
        singleLine = false,
        onValueChange = { state.value = it },
    )
}
```
{default-state="collapsed" collapsible="true" collapsed-title="item { CompositionLocalProvider(LocalLayoutDirection provides LayoutDirection.Rtl) {"}

<img src="compose-rtl-emoji.png" alt="Numbers and emojis" width="600"/>

## Accessibility in RTL layouts

Compose Multiplatform supports accessibility features for RTL layouts, 
including proper text direction and order for screen readers and handling of gestures.

### Screen readers

Screen readers automatically adapt to RTL layouts, keeping the logical reading order for users:

* RTL text is read from right to left, and mixed-direction text follows standard BiDi rules.
* Punctuation and numbers are announced in the correct sequence.

In complex layouts, it's necessary to define traversal semantics to ensure the correct reading order 
for screen readers.

### Focus-based navigation

Focus navigation in RTL layouts follows the layout's mirrored structure:

* Focus moves from right to left and top to bottom, following the natural flow of RTL content.
* Gestures like swipes or taps are automatically adjusted to the mirrored layout.

You can also define traversal semantics to ensure correct navigation between different traversal groups 
with the swipe-up or swipe-down accessibility gestures.

For details on how to define traversal semantics and set traversal indexes, 
refer to the [Accessibility](whats-new-compose-180.md#accessibility-for-container-views) section.

[//]: # (todo: replace accessibility link)

## Known issues

We keep improving support for RTL languages and plan to address the following known issues:

* Fix caret position while typing non-RTL characters in RTL layout ([CMP-3096](https://youtrack.jetbrains.com/issue/CMP-3096))
* Fix caret position for Arabic digits ([CMP-2772](https://youtrack.jetbrains.com/issue/CMP-2772))
* Fix `TextDirection.Content` ([CMP-2446](https://youtrack.jetbrains.com/issue/CMP-2446))