[//]: # (title: Integration with the SwiftUI framework)

Compose Multiplatform is interoperable with the [SwiftUI](https://developer.apple.com/xcode/swiftui/) framework.
You can embed Compose Multiplatform within a SwiftUI application as well as embed native SwiftUI components within
Compose Multiplatform UI. This page provides examples of how you can do this:

* [Use Compose Multiplatform inside a SwiftUI application](#use-compose-multiplatform-inside-a-swiftui-application)
* [Use SwiftUI inside Compose Multiplatform](#use-swiftui-inside-compose-multiplatform)

> In our templates, we use a SwiftUI wrapper that helps us write robust code.
>
{type="tip"}

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
{type="tip"}

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

In your Swift code, pass the `createUIViewController` to your entry point function code.
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

## What's next

You can also explore the way Compose Multiplatform can be [integrated with the UIKit framework](compose-uikit-integration).