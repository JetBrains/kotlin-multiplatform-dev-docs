# Working with modifiers

Modifiers allow you to decorate or augment a composable. Using modifiers, you can:

* Change the composable's size, layout, behavior, and appearance.
* Add information, such as accessibility labels.
* Handle user input.
* Add high-level interactions like making elements clickable, scrollable, draggable, or zoomable.

## Chaining modifiers 

Modifiers can be chained together to apply multiple effects:

```kotlin
@Composable
private fun Greeting(name: String) {
    Column(
        // Chained `Modifier` functions:
        modifier = Modifier
            // `Modifier.padding(24.dp)` adds padding around the column
            .padding(24.dp)
            // `Modifier.fillMaxWidth()` makes the column expand to fill the available width
            .fillMaxWidth()
    ) {
        Text(text = "Hello,")
        Text(text = name)
    }
}
```

**The order of modifier functions in the chain is significant**. Each function makes changes to the `Modifier` returned by 
the previous function, so the sequence of calls directly affects the composable's final behavior and appearance.

## Built-in modifiers

Compose Multiplatform provides built-in modifiers, such as `size`, `padding`, and `offset`, for handling common 
layout and positioning tasks.

### Size modifiers

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

### Padding modifiers

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

### Offset modifiers

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

## Scoped modifiers

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
It determines how much space a composable should occupy relative to its siblings:

```kotlin
@Composable
fun Card() {
    Row(
        // Takes the full width of its parent
        modifier = Modifier.fillMaxWidth() 
    ) {
        Image(
            /* Placeholder for loading an image */,
            // Assigns a weight of 1f to occupy one fraction of the available space 
            modifier = Modifier.weight(1f) 
        )
        
        Column(
            // Assigns a weight of 2f, taking twice the width compared to the Image
            modifier = Modifier.weight(2f)
        ) {
            // Content inside the column
        }
    }
}
```

## Extracting and reusing modifiers 

When you chain modifiers together, you can extract the chain into variables or functions for reuse. 
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

## Custom modifiers

While Compose Multiplatform provides many built-in modifiers for common use-cases right out of the box, you can also 
create your own custom modifiers.

There are several approaches to creating custom modifiers:

* [Chaining existing modifiers](https://developer.android.com/develop/ui/compose/custom-modifiers#chain-existing)
* [Using a composable modifier factory](https://developer.android.com/develop/ui/compose/custom-modifiers#create_a_custom_modifier_using_a_composable_modifier_factory)
* [Lower the level `Modifier.Node` API](https://developer.android.com/develop/ui/compose/custom-modifiers#implement-custom)

## What's next

Learn more about modifiers in [Jetpack Compose documentation](https://developer.android.com/develop/ui/compose/modifiers).