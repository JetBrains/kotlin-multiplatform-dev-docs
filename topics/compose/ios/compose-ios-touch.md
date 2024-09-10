[//]: # (title: Handling touch events with interop on iOS)

On iOS, Compose Multiplatform can integrate with the native UIKit and SwiftUI frameworks.
One challenge of such integration is handling touches: when a Compose Multiplatform app includes native UI elements,
the app may need to react differently to touches in the interop area depending on the context.

Currently, Compose Multiplatform has only one strategy for dealing with touch events in a native view:
all touches are processed entirely by the native UI, with Compose not being aware that they occurred at all.

Both the default behavior and the ability to customize it will be improved with the release of Compose Multiplatform 1.7.0.
Changes are described in the following sections. Consider trying them out with the %composeEapVersion% version.

## New approach to handling touches in interop UI {label="EAP"}

When each touch in an interop area is sent immediately to the underlying native UI element,
the container composable can't react to the same touch.
The most obvious problem this presents is scrolling. If an interop area is in a scrollable container, the user might expect
that the area:

* Reacts to their touch when they want to interact with it.
* Doesn't react to their touch when they want to scroll the parent container.

To solve this, Compose Multiplatform implements behavior inspired by [`UIScrollView`](https://developer.apple.com/documentation/uikit/uiscrollview).
When the touch is first detected, there is a short delay (150 ms) that lets the app decide whether to make the container aware of it:

* If there is movement detected within this time, the user likely wants to scroll the container. Compose Multiplatform does not
  make the native UI element aware of this touch sequence.
* If there is no movement detected within this time, the touch is likely intended for the underlying element. For the rest of the touch sequence, Compose Multiplatform
  hands over control to the native UI.

If your interop view is not meant to be interacted with, you can disable all touch processing in advance.
To do that, call the constructor for a `UIKitView` or a `UIViewController` with the `interactive` parameter set to `false`. 

### Choosing the strategy for touch processing {annotations="Experimental"}

With Compose Multiplatform %composeEapVersion% you can also try out the experimental API for finer control over interop UI.

The new constructors for `UIKitView` or a `UIViewController` accept a `UIKitInteropProperties` object as an argument.
This object allows setting:

* The `interactionMode` parameter for the given interop view, which lets you select a touch processing strategy.
* The `isNativeAccessibilityEnabled` option, which changes accessibility behavior for the interop view.

The `interactionMode` parameter can be set to either `Cooperative` or `NonCooperative`:

* The `Cooperative` mode is the new default, as described above: Compose Multiplatform introduces a delay into touch processing.
The experimental API allows you to fine-tune this delay by trying out different values instead of the default 150 ms.
* The NonCooperative mode uses the previous strategy, where Compose Multiplatform does not handle any touch events in the interop view.
  Despite the general problems listed above, this mode can be useful if you are certain that interop touches never need
  to be handled at the Compose level.
* To disable any interaction with the native UI, pass `interactionMode = null` to the constructor.

## What's next?

Learn more about [UIKit](compose-uikit-integration.md) and [SwiftUI](compose-swiftui-integration.md) integration in Compose Multiplatform.