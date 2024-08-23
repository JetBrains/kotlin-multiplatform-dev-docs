[//]: # (title: Handling touch events with interop on iOS)

<title label="EAP" annotation="Beta">Handling touch events with interop on iOS</title>


On iOS, Compose Multiplatform can integrate with the native UIKit and SwiftUI frameworks. In particular,
a Compose Multiplatform app can include native UI elements. One of the challenges of such integration is handling touches:
the app may need to react differently to touches in the interop area depending on context.

Without any additional processing, each touch in an interop area is sent immediately to the underlying native UI element.
However, that makes it impossible for the container element to react to the same touch as well.
The most obvious problem this presents is scrolling. If an interop area is in a scrollable container, the user might expect
that the area:

* reacts to their touch when they want to interact with it,
* doesn't react to their touch when they want to scroll the parent container.

To solve this, Compose Multiplatform implements behavior inspired by [UIScrollView](https://developer.apple.com/documentation/uikit/uiscrollview).
When the touch is first detected, there is a short delay that lets the app decide whether to make the container aware of it:

* If there is movement detected within this time, the user likely wants to scroll without activating the child element.
* If not, the touch is likely intended for the underlying element and should be forwarded.

TODO Right now, in the (EAP version) there is no option to customize this behavior, but an API is coming in future releases.

## What's next? {label="EAP" annotation="Beta"}

Learn more about UIKit and SwiftUI integration in Compose Multiplatform.