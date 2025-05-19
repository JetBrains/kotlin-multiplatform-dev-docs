[//]: # (title: Integration with the UIKit framework)

<show-structure depth="3"/>

Compose Multiplatform is interoperable with the [UIKit](https://developer.apple.com/documentation/uikit) framework.
You can embed Compose Multiplatform within an UIKit application as well as embed native UIKit components within Compose
Multiplatform. This page provides examples both for using Compose Multiplatform inside a UIKit application and for
embedding UIKit components inside Compose Multiplatform UI.

> To learn about SwiftUI interoperability, see the [](compose-swiftui-integration.md) article.
>
{style="tip"}

## Use Compose Multiplatform inside a UIKit application

To use Compose Multiplatform inside a UIKit application, add your Compose Multiplatform code to
any [container view controller](https://developer.apple.com/documentation/uikit/view_controllers).
This example uses Compose Multiplatform inside the `UITabBarController` class:

```swift
let composeViewController = Main_iosKt.ComposeOnly()
composeViewController.title = "Compose Multiplatform inside UIKit"

let anotherViewController = UIKitViewController()
anotherViewController.title = "UIKit"

// Set up the UITabBarController
let tabBarController = UITabBarController()
tabBarController.viewControllers = [
    // Wrap the created ViewControllers in a UINavigationController to set titles
    UINavigationController(rootViewController: composeViewController),
    UINavigationController(rootViewController: anotherViewController)
]
tabBarController.tabBar.items?[0].title = "Compose"
tabBarController.tabBar.items?[1].title = "UIKit"
```

With this code, your application should look like this:

![UIKit](uikit.png){width=300}

Explore this code in
the [sample project](https://github.com/JetBrains/compose-multiplatform/tree/master/examples/interop/ios-compose-in-uikit).

## Use UIKit inside Compose Multiplatform

To use UIKit elements inside Compose Multiplatform, add the UIKit elements that you want to use to a
[UIKitView](https://github.com/JetBrains/compose-multiplatform-core/blob/47c012bfe2d4570fb08432253298b8e2b6e38ade/compose/ui/ui/src/uikitMain/kotlin/androidx/compose/ui/interop/UIKitView.uikit.kt)
from Compose Multiplatform. You can write this code purely in Kotlin or use Swift as well.

### Map view

You can implement a map view in Compose Multiplatform using the UIKit's [`MKMapView`](https://developer.apple.com/documentation/mapkit/mkmapview)
component. Set the component size by using the `Modifier.size()` or `Modifier.fillMaxSize()` functions
from Compose Multiplatform:

```kotlin
UIKitView(
    factory = { MKMapView() },
    modifier = Modifier.size(300.dp),
)
```

With this code, your application should look like this:

![MapView](mapview.png){width=300}

Now, let's look at an advanced example. This code wraps
UIKit's [`UITextField`](https://developer.apple.com/documentation/uikit/uitextfield/) in Compose Multiplatform:

```kotlin
@OptIn(ExperimentalForeignApi::class)
@Composable
fun UseUITextField(modifier: Modifier = Modifier) {
    var message by remember { mutableStateOf("Hello, World!") }
    UIKitView(
        factory = {
            val textField = object : UITextField(CGRectMake(0.0, 0.0, 0.0, 0.0)) {
                @ObjCAction
                fun editingChanged() {
                    message = text ?: ""
                }
            }
            textField.addTarget(
                target = textField,
                action = NSSelectorFromString(textField::editingChanged.name),
                forControlEvents = UIControlEventEditingChanged
            )
            textField
        },
        modifier = modifier.fillMaxWidth().height(30.dp),
        update = { textField ->
            textField.text = message
        }
    )
}
```

* The factory parameter contains the `editingChanged()` function and the `textField.addTarget()` listener to detect any
  changes to `UITextField`.
* The `editingChanged()` function is annotated with `@ObjCAction` so that it can interoperate with Objective-C code.
* The `action` parameter of the `addTarget()` function later on passes the name of the `editingChanged()` function,
  so it would be called in response to a `UIControlEventEditingChanged` event.
* The `update` parameter of `UIKitView()` is called when the observable message state changes its value.
* The function updates the `text` attribute of the `UITextField` so that the user sees the updated value.

Explore the code for this example in our [sample project](https://github.com/JetBrains/compose-multiplatform/tree/master/examples/interop/ios-uikit-in-compose).

### Camera view

You can implement a camera view in Compose Multiplatform using UIKit's [`AVCaptureSession`](https://developer.apple.com/documentation/avfoundation/avcapturesession)
and [`AVCaptureVideoPreviewLayer`](https://developer.apple.com/documentation/avfoundation/avcapturevideopreviewlayer) components.

This allows your application to access the device's camera and display a live preview.

Here's an example of how to implement a basic camera view:

```kotlin
UIKitView(
    factory = {
        val session = AVCaptureSession().apply {
            val device = AVCaptureDevice.defaultDeviceWithMediaType(AVMediaTypeVideo)!!
            val input = AVCaptureDeviceInput.deviceInputWithDevice(device, null)!!
            addInput(input)
        }
        val previewLayer = AVCaptureVideoPreviewLayer(session)
        session.startRunning()

        object : UIView() {
            override fun layoutSubviews() {
                super.layoutSubviews()
                previewLayer.frame = bounds
            }
        }.apply {
            layer.addSublayer(previewLayer)
        }
    },
    modifier = Modifier.size(300.dp)
)
```

Now, let's look at an advanced example. This code captures a photo using, handles device orientation, and includes
location metadata:

```kotlin
@OptIn(ExperimentalForeignApi::class)
@Composable
fun CameraView(
    modifier: Modifier = Modifier,
    onCapture: (picture: PictureData.Camera, image: PlatformStorableImage) -> Unit
) {
    var cameraAccess by remember { mutableStateOf(CameraAccess.Undefined) }
    LaunchedEffect(Unit) {
        when (AVCaptureDevice.authorizationStatusForMediaType(AVMediaTypeVideo)) {
            AVAuthorizationStatusAuthorized -> cameraAccess = CameraAccess.Authorized
            AVAuthorizationStatusDenied, AVAuthorizationStatusRestricted -> cameraAccess = CameraAccess.Denied
            AVAuthorizationStatusNotDetermined -> {
                AVCaptureDevice.requestAccessForMediaType(AVMediaTypeVideo) {
                    cameraAccess = if (it) CameraAccess.Authorized else CameraAccess.Denied
                }
            }
        }
    }

    Box(modifier.fillMaxSize(), contentAlignment = Alignment.Center) {
        when (cameraAccess) {
            CameraAccess.Authorized -> AuthorizedCamera(onCapture)
            CameraAccess.Denied -> Text("Camera access denied", color = Color.White)
            CameraAccess.Undefined -> Unit
        }
    }
}
```

The `AuthorizedCamera` composable performs the following tasks:

* Sets up `AVCaptureSession` with input and output.
* Creates a `UIKitView` to render the camera preview using `AVCaptureVideoPreviewLayer`.
* Observes device orientation and updates the video feed's orientation accordingly.
* Captures photos using `AVCapturePhotoOutput`.
* Retrieves GPS location via `CLLocationManager`.
* Shows a loading indicator while a photo is being processed.
* Passes the photo and its metadata (name, description, GPS position) back to the compose layer using a callback.

The preview is displayed using a custom `UIView` subclass, which updates the preview layer size dynamically in `layoutSubviews()`.
A floating capture button allows users to take a photo, and `CircularProgressIndicator` gives feedback during capture.

Explore the code for this example in the [ImageViewer sample project](https://github.com/JetBrains/compose-multiplatform/tree/master/examples/imageviewer).

### Web view

You can implement a web view in Compose Multiplatform using UIKit's [`WKWebView`](https://developer.apple.com/documentation/webkit/wkwebview)
component. This allows your application to display and interact with web content within the UI.
Set the component size by using the `Modifier.size()` or `Modifier.fillMaxSize()` functions from Compose Multiplatform:

```kotlin
UIKitView(
    factory = {
        WKWebView().apply {
            loadRequest(NSURLRequest(URL = NSURL(string = "https://www.jetbrains.com")))
        }
    },
    modifier = Modifier.size(300.dp)
)
```
Now, let's look at an advanced example. This code configures the web view with navigation delegates and allows
communication between Kotlin and JavaScript:

```kotlin
@Composable
fun WebViewWithDelegate(
    modifier: Modifier = Modifier,
    initialUrl: String = "https://www.jetbrains.com",
    onNavigationChange: (String) -> Unit = {}
) {
    val delegate = remember {
        object : NSObject(), WKNavigationDelegateProtocol {
            override fun webView(
                webView: WKWebView,
                didFinishNavigation: WKNavigation?
            ) {
                onNavigationChange(webView.URL?.absoluteString ?: "")
            }
        }
    }
    UIKitView(
        modifier = modifier,
        factory = {
            val webView = WKWebView().apply {
                navigationDelegate = delegate
                loadRequest(NSURLRequest(uRL = NSURL(string = initialUrl)))
            }
            webView
        },
        update = { webView ->
            if (webView.URL?.absoluteString != initialUrl) {
                webView.loadRequest(NSURLRequest(uRL = NSURL(string = initialUrl)))
            }
        }
    )
}
```

The `WebViewWithDelegate` composable performs the following tasks:

* Creates a stable delegate object implementing the `WKNavigationDelegateProtocol` interface.
  This object is remembered across recompositions using Compose's `remember`.
* Instantiates a `WKWebView`, embeds it using `UIKitView`, and configures it assigning the remembered delegate.
* Loads an initial web page provided by the `initialUrl` parameter.
* Observes navigation changes via the delegate and passes the current URL through the `onNavigationChange` callback.
* Uses the `update` parameter to observe changes in the requested URL and reloads the web page accordingly.

## What's next

You can also explore the way Compose Multiplatform can be [integrated with the SwiftUI framework](compose-swiftui-integration.md).