[//]: # (title: Drag-and-drop operations)

You can enable your Compose Multiplatform app to accept data that users drag into it from other applications 
or allow users to drag data out of your app.
To implement this, use the `dragAndDropSource` and `dragAndDropTarget` modifiers to specify particular composables
as potential sources or destinations for drag operations.

> Both `dragAndDropSource` and `dragAndDropTarget` modifiers are experimental, subject to change, and require an opt-in annotation.
> 
{style="warning"}

## Creating a drag source

To prepare a composable to be a drag source:
1. Apply the `dragAndDropSource` modifier to your composable.
2. Add a `drawDragDecoration` lambda to render the visual representation shown while the user drags the data.
3. Describe the data that is supposed to be dragged to the target with a `DragAndDropTransferable()` call.

The following example sets up a `Box` composable as a drag source that exports a string:

```kotlin
val exportedText = "Hello, drag and drop!"

Box(Modifier
    .size(200.dp)
    .background(Color.LightGray)
    .dragAndDropSource(
        // Creates a visual representation of the data being dragged
        // (white rectangle with the exportedText string centered on it).
        drawDragDecoration = {
            drawRect(
                color = Color.White, 
                topLeft = Offset(x = 0f, y = size.height/4),
                size = Size(size.width, size.height/2)
            )
            val textLayoutResult = textMeasurer.measure(
                text = AnnotatedString(exportedText),
                layoutDirection = layoutDirection,
                density = this
            )
            drawText(
                textLayoutResult = textLayoutResult,
                topLeft = Offset(
                    x = (size.width - textLayoutResult.size.width) / 2,
                    y = (size.height - textLayoutResult.size.height) / 2,
                )
            )
        }
    ) { offset ->
        // Defines transferable data and supported transfer actions.
        // When an action is concluded, prints the result into
        // system output with onTransferCompleted().    
        DragAndDropTransferData(
            transferable = DragAndDropTransferable(
                StringSelection(exportedText)
            ),

            // List of actions supported by this drag source. A type of action
            // is passed to the drop target together with data.
            // The target can use this to reject an inappropriate drop operation
            // or to interpret user expectations.
            supportedActions = listOf(
                DragAndDropTransferAction.Copy,
                DragAndDropTransferAction.Move,
                DragAndDropTransferAction.Link,
            ),
            dragDecorationOffset = offset,
            onTransferCompleted = { action -> 
                println("Action at the source: $action")
            }
        )
    }
) {
    Text("Drag Me", Modifier.align(Alignment.Center))
}
```
{initial-collapse-state="collapsed"  collapsed-title="Box(Modifier.dragAndDropSource"}

## Creating a drop target

To prepare a composable to be a drag-and-drop target:

1. Create (and `remember`) a `DragAndDropTarget` object with overrides for the drag
   events you want to handle, such as `onStarted`, `onEnded`, and `onDrop`.
2. Apply the `dragAndDropTarget` modifier to your composable. Pass a
   `shouldStartDragAndDrop` lambda that determines whether the target accepts the
   session, along with the `DragAndDropTarget` object.
3. In `onDrop`, read the dropped data from the `DragAndDropEvent` and return `true`
   to indicate that the drop was accepted.

The following desktop example sets up a `Box` composable as a drop target that displays text dropped into it:

```kotlin
var showTargetBorder by remember { mutableStateOf(false) }
var targetText by remember { mutableStateOf("Drop here") }
val coroutineScope = rememberCoroutineScope()
val dragAndDropTarget = remember {
    object: DragAndDropTarget {

        // Highlights the border of a potential drop target
        override fun onStarted(event: DragAndDropEvent) {
            showTargetBorder = true
        }

        override fun onEnded(event: DragAndDropEvent) {
            showTargetBorder = false
        }

        override fun onDrop(event: DragAndDropEvent): Boolean {
            // Prints the type of action into system output every time
            // a drag-and-drop operation is concluded.
            println("Action at the target: ${event.action}")

            val result = (targetText == "Drop here")

            // Changes the text to the value dropped into the composable.
            targetText = event.awtTransferable.let {
                if (it.isDataFlavorSupported(DataFlavor.stringFlavor))
                    it.getTransferData(DataFlavor.stringFlavor) as String
                else
                    it.transferDataFlavors.first().humanPresentableName
            }

            // Reverts the text of the drop target to the initial
            // value after 2 seconds.
            coroutineScope.launch {
                delay(2.seconds)
                targetText = "Drop here"
            }
            return result
        }
    }
}

Box(Modifier
    .size(200.dp)
    .background(Color.LightGray)
    .then(
        if (showTargetBorder)
            Modifier.border(BorderStroke(3.dp, Color.Black))
        else
            Modifier
    )
    .dragAndDropTarget(
        // With "true" as the value of shouldStartDragAndDrop,
        // drag-and-drop operations are enabled unconditionally.    
        shouldStartDragAndDrop = { true },
        target = dragAndDropTarget
    )
) {
    Text(targetText, Modifier.align(Alignment.Center))
}
```
{initial-collapse-state="collapsed"  collapsed-title="val dragAndDropTarget = remember"}

## What's next

For more details on implementation and common use cases, see the [Drag and drop](https://developer.android.com/develop/ui/compose/touch-input/user-interactions/drag-and-drop) article about the corresponding modifiers
in the Jetpack Compose documentation.