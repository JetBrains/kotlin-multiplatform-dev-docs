# Layout basics

To effectively build user interfaces in Compose Multiplatform, it's important to understand the key concepts of layout 
construction, including core principles, layout phases, and common components and tools available for 
structuring your UI.

## Composable functions

You can build user interfaces by defining a set of composable functions. These functions take in data and emit 
UI elements. The `@Composable` annotation informs the Compose compiler that the function converts data into a UI.

A simple composable function that displays text:

```kotlin
@Composable
fun Greeting(name: String) {
    Text(text = "Hello, $name!")
}
```

## Column, Row, and Box

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

## Modifiers

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

Learn more in [](compose-layout-modifiers.md).

## What's next

* For a deep dive on layouts, see [Jetpack Compose documentation](https://developer.android.com/develop/ui/compose/layouts).
* Learn about the [lifecycle](compose-lifecycle.md) of components.