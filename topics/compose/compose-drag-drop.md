[//]: # (title: Drag and drop operations)

<title label="EAP" annotations="Desktop">Drag and drop operations</title>

You can enable your Compose Multiplatform app to accept data that users drag into it from other applications 
or allow users to drag data out of your app.
To implement this, use the `dragAndDropSource` and `dragAndDropTarget` modifiers to specify particular composables
as potential sources or destinations for drag operations.

> Currently, drag and drop operations are only supported in Compose Multiplatform for desktop.
> The support will be extended to iOS and web in future releases.
>
{type="note"}

## Creating a drag source

To prepare a composable to be a drag source:
1. Choose the trigger for the drag event with the `detectDragGestures()` function (for example, `onDragStart`).
2. Call the `startTransfer()` function and describe the drag and drop session with a `DragAndDropTransferData()` call.
3. Describe the data that is supposed to be dragged to the target with a `DragAndDropTransferable()` call.

Example of a `Box()` composable that allows users to drag a string from it:

```kotlin
val exportedText = "Hello, drag and drop!"

Box(Modifier
    .dragAndDropSource(
        // Creates a visual representation for the data being dragged
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
    )
    .size(200.dp)
    .background(Color.LightGray)
    {
        detectDragGestures(
            onDragStart = { offset ->
                startTransfer(
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
                        // or to interpret user's expectations.
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
                )
            },
            onDrag = { _, _ -> },
        )
    }
) {
    Text("Drag Me", Modifier.align(Alignment.Center))
}
```
{initial-collapse-state="collapsed"  collapsed-title="Box(Modifier.dragAndDropSource"}

## Creating a drop target

To prepare a composable to be a drag and drop target:

1. Describe the conditions for the composable to be the target of a drop in the `shouldStartDragAndDrop` lambda.
2. Create (and `remember`) the `DragAndDropTarget` object that will contain your overrides for drag event handlers.
3. Write the necessary overrides: for example, `onDrop` for parsing the received data, or `onEntered` when a draggable
   object enters the composable.

Example of a `Box()` composable that is ready to display text dropped into it:

```kotlin
var showTargetBorder by remember { mutableStateOf(false) }
var targetText by remember { mutableStateOf("Drop Here") }
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
            // a drag and drop operation is concluded.
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
                delay(2000)
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
        // drag and drop operations are enabled unconditionally.    
        shouldStartDragAndDrop = { true },
        target = dragAndDropTarget
    )
) {
    Text(targetText, Modifier.align(Alignment.Center))
}
```
{initial-collapse-state="collapsed"  collapsed-title="val dragAndDropTarget = remember"}

## What's next

For more implementation details and common use cases, see the [Drag and drop](https://developer.android.com/develop/ui/compose/touch-input/user-interactions/drag-and-drop) article about the corresponding modifiers
in the Jetpack Compose documentation.