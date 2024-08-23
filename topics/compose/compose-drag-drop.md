[//]: # (title: Drag and drop operations)

<title label="EAP" annotations="Android,Desktop">Drag and drop operations</title>

> Currently, the drag and drop operations are only supported for Android and desktop, but the support will be extended
> to iOS and web in future releases.
>
{type="note"}

To allow users to drag files and objects in or out of your Compose Multiplatform app,
you can use the `dragAndDropSource` and `dragAndDropTarget` modifiers to specify potential sources or destinations for drag operations.

To prepare a composable to be a drag source:
1. Choose the trigger for the drag event with the `detectDragGestures()` function (for example, `onDragStart`).
2. Call the `startTransfer()` function and describe the drag and drop session with a `DragAndDropTransferData()` call.
3. Describe the data that is supposed to be dragged to the target with a `DragAndDropTransferable()` call.

To prepare a composable to be a drag target:
1. Describe the conditions for the composable to be the target of a drop in the `shouldStartDragAndDrop` lambda.
2. Create (and `remember`) the `DragAndDropTarget` object that will contain your overrides for drag event handlers.
3. Write the necessary overrides: for example, `onDrop` for parsing the received data, or `onEntered` when a draggable
   object enters the composable.

For more details and common use cases, see the [Drag and drop](https://developer.android.com/develop/ui/compose/touch-input/user-interactions/drag-and-drop) article about the corresponding modifiers in Jetpack Compose documentation.