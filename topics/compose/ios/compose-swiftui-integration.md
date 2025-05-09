[//]: # (title: Integration with the SwiftUI framework)

<show-structure depth="3"/>

Compose Multiplatform is interoperable with the [SwiftUI](https://developer.apple.com/xcode/swiftui/) framework.
You can embed Compose Multiplatform within a SwiftUI application as well as embed native SwiftUI components within
Compose Multiplatform UI. This page provides examples both for using Compose Multiplatform inside SwiftUI
and for embedding SwiftUI inside a Compose Multiplatform app.

> To learn about UIKit interoperability, see the [](compose-uikit-integration.md) article.
> 
{style="tip"}

## Use Compose Multiplatform inside a SwiftUI application

To use Compose Multiplatform inside a SwiftUI application, create a Kotlin function `MainViewController()` that
returns [`UIViewController`](https://developer.apple.com/documentation/uikit/uiviewcontroller/) from UIKit
and contains Compose Multiplatform code:

```kotlin
fun MainViewController(): UIViewController =
    ComposeUIViewController {
        Box(Modifier.fillMaxSize(), contentAlignment = Alignment.Center) {
            Text("This is Compose code", fontSize = 20.sp)
        }
    }
```

[`ComposeUIViewController()`](https://github.com/JetBrains/compose-multiplatform-core/blob/5b487914cc20df24187f9ddf54534dfec30f6752/compose/ui/ui/src/uikitMain/kotlin/androidx/compose/ui/window/ComposeWindow.uikit.kt)
is a Compose Multiplatform library function that accepts a composable function as the `content` argument.
The function passed in this way can call other composable functions, for example, `Text()`.

> Composable functions are functions that have the `@Composable` annotation.
>
{style="tip"}

Next, you need a structure that represents Compose Multiplatform in SwiftUI.
Create the following structure that converts a `UIViewController` instance to a SwiftUI view:

```swift
struct ComposeViewController: UIViewControllerRepresentable {
    func makeUIViewController(context: Context) -> UIViewController {
        return Main_iosKt.MainViewController()
    }

    func updateUIViewController(_ uiViewController: UIViewController, context: Context) {
    }
}
```

Now you can use the `ComposeView` structure in other SwiftUI code.

`Main_iosKt.MainViewController` is a generated
name. You can learn more about accessing Kotlin code from Swift on the [Interoperability with Swift/Objective-C](https://kotlinlang.org/docs/native-objc-interop.html#top-level-functions-and-properties)
page.

In the end, your application should look like this:

![ComposeView](compose-view.png){width=300}

You can use this `ComposeView` in any SwiftUI view hierarchy and control its size from within SwiftUI code.

If you want to embed Compose Multiplatform into your existing applications, use the `ComposeView` structure wherever
SwiftUI is used.
For an example, see our [sample project](https://github.com/JetBrains/compose-multiplatform/tree/master/examples/interop/ios-compose-in-swiftui).

## Use SwiftUI inside Compose Multiplatform

To use SwiftUI inside Compose Multiplatform, add your Swift code to an
intermediate [`UIViewController`](https://developer.apple.com/documentation/uikit/uiviewcontroller/).
Currently, you can't write SwiftUI structures directly in Kotlin. Instead, you have to write them in Swift
and pass them to a Kotlin function.

To start, add an argument to your entry point function to create a `ComposeUIViewController` component:

```kotlin
@OptIn(ExperimentalForeignApi::class)
fun ComposeEntryPointWithUIViewController(
    createUIViewController: () -> UIViewController
): UIViewController =
    ComposeUIViewController {
        Column(
            Modifier
                .fillMaxSize()
                .windowInsetsPadding(WindowInsets.systemBars),
            horizontalAlignment = Alignment.CenterHorizontally
        ) {
            Text("How to use SwiftUI inside Compose Multiplatform")
            UIKitViewController(
                factory = createUIViewController,
                modifier = Modifier.size(300.dp).border(2.dp, Color.Blue),
            )
        }
    }
```

In your Swift code, pass the `createUIViewController` to your entry point function.
You can use a `UIHostingController` instance to wrap SwiftUI views:

```swift
Main_iosKt.ComposeEntryPointWithUIViewController(createUIViewController: { () -> UIViewController in
    let swiftUIView = VStack {
        Text("SwiftUI in Compose Multiplatform")
    }
    return UIHostingController(rootView: swiftUIView)
})
```

In the end, your application should look like this:

![UIView](uiview.png){width=300}

Explore the code for this example in
the [sample project](https://github.com/JetBrains/compose-multiplatform/tree/master/examples/interop/ios-swiftui-in-compose).

### Map view

You can implement a map view in Compose Multiplatform using SwiftUI's [`Map`](https://developer.apple.com/documentation/mapkit/map)
component. This allows your application to display fully interactive SwiftUI maps.

For the same [Kotlin entry point function](#use-swiftui-inside-compose-multiplatform), in Swift,
pass the `UIViewController` that wraps the `Map` view using a `UIHostingController`:

```swift
import SwiftUI
import MapKit

Main_iosKt.ComposeEntryPointWithUIViewController(createUIViewController: {
    let region = Binding.constant(
        MKCoordinateRegion(
            center: CLLocationCoordinate2D(latitude: 37.7749, longitude: -122.4194),
            span: MKCoordinateSpan(latitudeDelta: 0.05, longitudeDelta: 0.05)
        )
    )

    let mapView = Map(coordinateRegion: region)
    return UIHostingController(rootView: mapView)
})
```

Now, let's look at an advanced example. This code adds a custom annotation to the SwiftUI map and allows you to update
the view state from Swift:

```swift
import SwiftUI
import MapKit

struct AnnotatedMapView: View {
    @State private var region = MKCoordinateRegion(
        center: CLLocationCoordinate2D(latitude: 51.5074, longitude: -0.1278),
        span: MKCoordinateSpan(latitudeDelta: 0.1, longitudeDelta: 0.1)
    )

    var body: some View {
        Map(coordinateRegion: $region, annotationItems: [Landmark.example]) { landmark in
            MapMarker(coordinate: landmark.coordinate, tint: .blue)
        }
    }
}

struct Landmark: Identifiable {
    let id = UUID()
    let name: String
    let coordinate: CLLocationCoordinate2D


    static let example = Landmark(
        name: "Big Ben",
        coordinate: CLLocationCoordinate2D(latitude: 51.5007, longitude: -0.1246)
    )
}
```

You can then wrap this annotated map in a `UIHostingController` and pass it to your Compose Multiplatform code:

```swift
Main_iosKt.ComposeEntryPointWithUIViewController(createUIViewController: {
    return UIHostingController(rootView: AnnotatedMapView())
})
```

The `AnnotatedMapView` performs the following tasks:

* Defines a SwiftUI `Map` view and embeds it inside a custom view called `AnnotatedMapView`.
* Manages internal state for map positioning using `@State` and `MKCoordinateRegion`, allowing Compose Multiplatform to
  display an interactive, state-aware map.
* Displays a `MapMarker` on the map using a static `Landmark` model that conforms to `Identifiable`, which is required
  for annotations in SwiftUI.
* Uses `annotationItems` to declaratively place custom markers on the map.
* Wraps the SwiftUI component inside a `UIHostingController`, which is then passed to Compose Multiplatform as a `UIViewController`.

### Camera

You can implement a camera view in Compose Multiplatform using SwiftUI and UIKit's [`UIImagePickerController`](https://developer.apple.com/documentation/uikit/uiimagepickercontroller),
wrapped in a SwiftUI-compatible component. This allows your application to launch the system camera and capture photos.

For the same [Kotlin entry point function](#use-swiftui-inside-compose-multiplatform), in Swift, define a basic
`CameraView` using `UIImagePickerController` and embed it using `UIHostingController`:

```swift
Main_iosKt.ComposeEntryPointWithUIViewController(createUIViewController: {
    return UIHostingController(rootView: CameraView { image in
        // Handle captured image here
    })
})
```

Now, let's look at an advanced example. This code presents a camera view and displays a thumbnail of the captured image
in the same SwiftUI view:

```swift
import SwiftUI
import UIKit

struct CameraPreview: View {
    @State private var showCamera = false
    @State private var capturedImage: UIImage?

    var body: some View {
        VStack {
            if let image = capturedImage {
                Image(uiImage: image)
                    .resizable()
                    .scaledToFit()
                    .frame(height: 200)
            } else {
                Text("No image captured")
            }

            Button("Open Camera") {
                showCamera = true
            }
            .sheet(isPresented: $showCamera) {
                CameraView { image in
                    capturedImage = image
                }
            }
        }
    }
}
```

The `CameraPreview` view performs the following tasks:

* Presents a `CameraView` in a modal `.sheet` when the user taps a button.
* Uses the `@State` property wrapper to store and display the captured image.
* Embeds SwiftUI's native `Image` view to preview the photo.
* Reuses the same `UIViewControllerRepresentable`-based `CameraView` as before, but integrates it more deeply into
  the SwiftUI state system.

### Web view

You can implement a web view in Compose Multiplatform using SwiftUI by wrapping UIKit's [`WKWebView`](https://developer.apple.com/documentation/webkit/wkwebview)
component with `UIViewRepresentable`. This allows you to display embedded web content with full native rendering.

For the same [Kotlin entry point function](#use-swiftui-inside-compose-multiplatform), in Swift, define a basic `WebView`
embedded using `UIHostingController`:

```swift
Main_iosKt.ComposeEntryPointWithUIViewController(createUIViewController: {
    let url = URL(string: "https://www.jetbrains.com")!
    return UIHostingController(rootView: WebView(url: url))
})
```

Now, let's look at an advanced example. This code adds navigation tracking and loading state display to the web view:

```swift
import SwiftUI
import UIKit
import WebKit

struct AdvancedWebView: UIViewRepresentable {
    let url: URL
    @Binding var isLoading: Bool
    @Binding var currentURL: String

    func makeUIView(context: Context) -> WKWebView {
        let webView = WKWebView()
        webView.navigationDelegate = context.coordinator
        webView.load(URLRequest(url: url))
        return webView
    }

    func updateUIView(_ uiView: WKWebView, context: Context) {}

    func makeCoordinator() -> Coordinator {
        Coordinator(isLoading: $isLoading, currentURL: $currentURL)
    }

    class Coordinator: NSObject, WKNavigationDelegate {
        @Binding var isLoading: Bool
        @Binding var currentURL: String

        init(isLoading: Binding<Bool>, currentURL: Binding<String>) {
            _isLoading = isLoading
            _currentURL = currentURL
        }

        func webView(_ webView: WKWebView, didStartProvisionalNavigation navigation: WKNavigation?) {
            isLoading = true
        }

        func webView(_ webView: WKWebView, didFinish navigation: WKNavigation?) {
            isLoading = false
            currentURL = webView.url?.absoluteString ?? ""
        }
    }
}
```

Use it in a SwiftUI view as follows:

```swift
struct WebViewContainer: View {
    @State private var isLoading = false
    @State private var currentURL = ""

    var body: some View {
        VStack {
            if isLoading {
                ProgressView()
            }
            Text("URL: \(currentURL)")
                .font(.caption)
                .lineLimit(1)
                .truncationMode(.middle)

            AdvancedWebView(
                url: URL(string: "https://www.jetbrains.com")!,
                isLoading: $isLoading,
                currentURL: $currentURL
            )
        }
    }
}
```

The `AdvancedWebView` and `WebViewContainer` perform the following tasks:

* Create a `WKWebView` with a custom navigation delegate to track loading progress and URL changes.
* Use SwiftUI's `@State` bindings to dynamically update the UI in response to navigation events.
* Display a `ProgressView` spinner while a page is loading.
* Show the current URL at the top of the view using a `Text` component.
* Use `UIHostingController` to integrate this component into your Compose UI.

## What's next

You can also explore the way Compose Multiplatform can be [integrated with the UIKit framework](compose-uikit-integration.md).