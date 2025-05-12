# Implementing layouts

<show-structure depth="3"/>

To effectively build user interfaces in Compose Multiplatform, it's important to understand the key concepts of layout 
construction, including core principles, layout phases, and common components and tools available for 
structuring your UI. Compose Multiplatform adopts the 
[Jetpack Compose approach to implementing layouts](https://developer.android.com/develop/ui/compose/layouts/basics).

## Layout basics

### Composable functions

You can build user interfaces by defining a set of composable functions. These functions take in data and emit 
UI elements. The `@Composable` annotation informs the Compose compiler that the function converts data into a UI.

A simple composable function that displays text:

```kotlin
@Composable
fun Greeting(name: String) {
    Text(text = "Hello, $name!")
}
```

### Column, Row, and Box

To structure your layouts, you can use these basic building blocks:

* Use `Column` to place items vertically on the screen.
* Use `Row` to place items horizontally on the screen.
* Use `Box` to stack elements on top of each other. 
* Use the `FlowRow` and `FlowColumn` versions of `Row` and `Column` to build responsive layouts. 
Items automatically flow into the next line when the container runs out of space, creating multiple rows or columns:
    ```kotlin
    @Composable
    fun ResponsiveLayout() {
        FlowRow {
            Text(text = "Item 1")
            Text(text = "Item 2")
            Text(text = "Item 3")
        }
    }
    ```

### Modifiers

Modifiers allow you to decorate or adjust the behavior of composables in a declarative way.
They are essential for customizing layouts and interactions by providing control over dimensions, alignment, padding,
interaction behaviors, and much more.

For example, you can add padding and center alignment to text:

```kotlin
@Composable
fun ModifierExample() {
    Text(
        text = "Hello with padding",
        modifier = Modifier.padding(16.dp)
    )
}
```

Learn more in the [Working with modifiers](#working-with-modifiers) section.

### Layout phases

Compose uses a single-pass layout model for rendering the UI tree. Here are the key layout phases:

* **Measuring**. Each composable is measured, passing size constraints down the tree to its children recursively.
* **Sizing**. Leaf nodes determine their sizes based on constraints and content.
* **Placing**. The resolved sizes and placement instructions are passed back up the tree to position the composables.

See [Jetpack Compose documentation on the layout model](https://developer.android.com/develop/ui/compose/layouts/basics#model) for details.

### ConstraintLayout

`ConstraintLayout` allows you to arrange composables relative to one another without nesting the
`Row`, `Column`, or `Box` elements. `ConstraintLayout` positions composables using the following constraints:

* **Guidelines** are visual helpers for aligning composables.
* **Barriers** are virtual guidelines that reference multiple composables, and adjust based on the most extreme 
widget on the specified side.
* **Chains** provide group-like behavior along a single axis (horizontally or vertically).

In this example, we position and align a `Text` element to a guideline placed 25% of the way from the start (left) of the layout:

```kotlin
@Composable
fun ConstraintLayoutExample() {
    ConstraintLayout {
        // Positions 25% from the start
        val guideline = createGuidelineFromStart(0.25f) 
        val text = createRef()

        Text(
            text = "Aligned to guideline",
            modifier = Modifier.constrainAs(text) {
                start.linkTo(guideline)
            }
        )
    }
}
```

## Working with modifiers

Modifiers allow you to decorate or augment a composable. Using modifiers, you can:

* Change the composable's size, layout, behavior, and appearance.
* Add information, such as accessibility labels.
* Handle user input.
* Add high-level interactions like making elements clickable, scrollable, draggable, or zoomable.

### Chaining modifiers 

Modifiers can be chained together to apply multiple effects:

```kotlin
@Composable
private fun Greeting(name: String) {
    Column(
        modifier = Modifier
            .padding(24.dp)
            .fillMaxWidth()
    ) {
        Text(text = "Hello,")
        Text(text = name)
    }
}
```

The order of modifier functions in the chain is significant. Each function makes changes to the `Modifier` returned by 
the previous function, so the sequence of calls directly affects the composable's final behavior and appearance.

### Built-in modifiers

To set a fixed size, use the `size` modifier. When constraints need to be overridden, use the `requiredSize` modifier:

```kotlin
@Composable
fun Card() {
    // Sets the row size to 400x100 dp
    Row(modifier = Modifier.size(width = 400.dp, height = 100.dp)
    ) {
        Image(
            // Sets the required size to 150x150 dp and overrides the parent`s 100 dp limit
            modifier = Modifier.requiredSize(150.dp)
        )
        Column {
            // The content takes the remaining space within the row
        }
    }
}
```

Add padding around an element with the `padding` modifier. You can also apply padding dynamically in relation to 
baselines using `paddingFromBaseline`:

```kotlin
@Composable
fun Card() {
    Row {
        Column {
            // Applies padding to adjust the position relative to the baseline
            Text(
                text = "Title",
                modifier = Modifier.paddingFromBaseline(top = 50.dp)
            )
            // Follows the default arrangement as there is no padding specified
            Text(text = "Subtitle")
        }
    }
}
```

To adjust the position of a layout from its original position, use the `offset` modifier. Specify the offset in the X and Y axes:

```kotlin
@Composable
fun Card() {
    Row {
        Column {
            // Positions the text normally with no offsets applied
            Text(text = "Title")
            
            // Moves the text slightly to the right with the 4.dp offset along the X-axis,
            // while keeping the original vertical position
            Text(
                text = "Subtitle",
                modifier = Modifier.offset(x = 4.dp)
            )
        }
    }
}
```

For more examples, see [Jetpack Compose documentation on modifiers](https://developer.android.com/develop/ui/compose/modifiers#built-in-modifiers).

### Scoped modifiers

Scoped modifiers, also called parent data modifiers, notify the parent layout of specific requirements for a child.
For example, to match the size of a parent `Box`, use the `matchParentSize` modifier:

```kotlin
@Composable
fun MatchParentSizeComposable() {
    Box {
        // Takes size of its parent Box
        Spacer(
            Modifier
                .matchParentSize() 
                .background(Color.LightGray)
        )
        // The largest child, determines the Box size
        Card()
    }
}
```

Another example of a scoped modifier is `weight`, available within the `RowScope` or `ColumnScope`. 
It defines how much space a composable should occupy relative to its siblings:

```kotlin
@Composable
fun Card() {
    Row(
        // Takes the full width of its parent
        modifier = Modifier.fillMaxWidth() 
    ) {
        Image(
            /*...*/
            // Assigns a weight of 2f to ccupy twice the width compared to the Column
            modifier = Modifier.weight(2f) 
        )
        
        Column(
            // Assigns a weight of 1f, taking one fraction of the available space
            modifier = Modifier.weight(1f)
        ) {
            // Content inside the column
        }
    }
}
```

### Extracting and reusing modifiers 

When you chain multiple modifiers together, you can extract the chain into variables or functions for reuse. 
This improves code readability and may enhance performance by reusing modifier instances.

```kotlin
val commonModifier = Modifier
    .padding(16.dp)
    .background(Color.LightGray)

@Composable
fun Example() {
    // Applies the reusable modifier with padding and background color
    Text("Reusable modifier", modifier = commonModifier)

    // Reuses the same modifier for the Button
    Button(
        onClick = { /* Do something */ },
        modifier = commonModifier
    )
    {
        Text("Button with the same modifier")
    }
}
```

### Custom modifiers

While Compose Multiplatform provides many built-in modifiers for common use-cases right out of the box, you can also 
create your own custom modifiers.

There are several approaches to creating custom modifiers:

* Chaining existing modifiers
* Using a composable modifier factory
* Lower the level `Modifier.Node` API 

## Adaptive layouts

To provide a consistent user experience on all types of devices, adapt your app's UI to different display sizes, orientations, 
and input modes.

### Designing adaptive layouts

Follow these key guidelines when designing adaptive layouts:

* Prefer canonical layouts patterns like list-detail, feed, and supporting pane.
* Maintain consistency by reusing shared styles for padding, typography, and other design elements. Keep navigation patterns 
consistent across devices while following platform-specific guidelines.
* Break complex layouts into reusable composables for flexibility and modularity.
* Adjust for screen density and orientation using tools like `LocalConfiguration` and `LocalDensity`.

For a detailed guide, check out [Adaptive layout do's and don'ts](https://developer.android.com/develop/ui/compose/layouts/adaptive/adaptive-dos-and-donts).

### Using WindowSizeClass

Window size classes are a set of opinionated breakpoints that help you design, develop, and test adaptive layouts.

With `WindowSizeClass`, you can change your app layout as the display space available to your app changes. For example,
when the device orientation changes, or the app window is resized in multi‑window mode.

The window size classes categorize the display area available to your app into three categories for both width and height:
compact, medium, and expanded. As you make layout changes, test the layout behavior across all window sizes, 
especially at the different breakpoint widths.

<!--- waiting for a new plugin
### Previewing layouts

We have three different @Preview:

* Android-specific, for androidMain, from Android Studio.
* Separate desktop annotation plugin with our own implementation (only for desktop source set) + uiTooling plugin.
* Common annotation, also suddenly supported in Android Studio, works for Android only but from common code.
-->

## What's next

* Learn about the [lifecycle](compose-lifecycle.md) of components.
* For a deep dive, see [Jetpack Compose documentation](https://developer.android.com/develop/ui/compose/layouts).