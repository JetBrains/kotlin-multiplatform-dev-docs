[//]: # (title: Integration with the UIKit and SwiftUI frameworks)

Compose Multiplatform is interoperable with both [UIKit](https://developer.apple.com/documentation/uikit)
and [SwiftUI](https://developer.apple.com/xcode/swiftui/) frameworks. You can embed Compose Multiplatform within an iOS
application as well as embed native iOS components within Compose Multiplatform. This page provides some examples of how
you can do this when working with UIKit and SwiftUI frameworks:

* [Use Compose Multiplatform inside a SwiftUI application](#use-compose-multiplatform-inside-a-swiftui-application)
* [Use Compose Multiplatform inside a UIKit application](#use-compose-multiplatform-inside-a-uikit-application)
* [Use UIKit inside Compose Multiplatform](#use-uikit-inside-compose-multiplatform)
* [Use SwiftUI inside Compose Multiplatform](#use-swiftui-inside-compose-multiplatform)

> In our templates, we use a SwiftUI wrapper that helps us to write robust code.
>
{type="tip"}

## Use Compose Multiplatform inside a SwiftUI application

To use Compose Multiplatform inside a SwiftUI application, create a Kotlin function `MainViewController()` that
returns [`UIViewController`](https://developer.apple.com/documentation/uikit/uiviewcontroller/) from UIKit and contains
the following Compose Multiplatform code inside:

```kotlin
fun MainViewController(): UIViewController =
    ComposeUIViewController {
        Box(Modifier.fillMaxSize(), contentAlignment = Alignment.Center) {
            Text("This is Compose code", fontSize = 20.sp)
        }
    }
```

[`ComposeUIViewController()`](https://github.com/JetBrains/compose-multiplatform-core/blob/5b487914cc20df24187f9ddf54534dfec30f6752/compose/ui/ui/src/uikitMain/kotlin/androidx/compose/ui/window/ComposeWindow.uikit.kt)
is a library function from Compose Multiplatform that accepts another function passed using the _content_ argument. This
function can call other composable functions, for example, `Text()`.

> Composable functions are functions that have the `@Composable` annotation.
>
{type="tip"}

Next, you need a structure that represents Compose Multiplatform in SwiftUI. Create the following `ComposeView` structure
that converts `UIViewController` to a SwiftUI view:

```swift
struct ComposeView: UIViewControllerRepresentable {
    func makeUIViewController(context: Context) -> UIViewController {
        Main_iosKt.MainViewController()
    }

    func updateUIViewController(_ uiViewController: UIViewController, context: Context) {
}
```

Now you can use the `ComposeView` structure in other SwiftUI structures. `Main_iosKt.MainViewController` is a generated
name, you can learn more about name translation from Swift on the Kotlin [Interoperability with Swift/Objective-C](https://kotlinlang.org/docs/native-objc-interop.html#top-level-functions-and-properties) page.

In the end, your application should look like this:

![ComposeView](compose-view.png){width=300}

In addition, you can use this `ComposeView` in any SwiftUI view hierarchy and control its size from within SwiftUI code.

If you want to embed Compose Multiplatform into your existing applications, use the `ComposeView` structure in any place
where SwiftUI is used. For an example, see
our [sample project](https://github.com/JetBrains/compose-multiplatform/tree/master/examples/interop/ios-compose-in-swiftui).

## Use Compose Multiplatform inside a UIKit application

To use Compose Multiplatform inside a UIKit application, add your Compose Multiplatform code to
any [container view controller]( https://developer.apple.com/documentation/uikit/view_controllers). This example uses
Compose Multiplatform inside the `UITabBarController` class:

```swift
let composeViewController = Main_iosKt.ComposeOnly()
composeViewController.title = "Compose Multiplatform inside UIKit"

let anotherViewController = UIKitViewController()
anotherViewController.title = "UIKit"

// Set up the UITabBarController
let tabBarController = UITabBarController()
tabBarController.viewControllers = [
    // Wrap them in a UINavigationController for the titles
    UINavigationController(rootViewController: composeViewController),
    UINavigationController(rootViewController: anotherViewController)
]
tabBarController.tabBar.items?[0].title = "Compose"
tabBarController.tabBar.items?[1].title = "UIKit"
```

With this code, your application should look like this:

![UIKit](uikit.png){width=300}

Explore the code for the example in
this [sample project](https://github.com/JetBrains/compose-multiplatform/tree/master/examples/interop/ios-compose-in-uikit).

## Use UIKit inside Compose Multiplatform

To use UIKit elements inside Compose Multiplatform, add the UIKit elements that you want to use to
[UIKitView](https://github.com/JetBrains/compose-multiplatform-core/blob/47c012bfe2d4570fb08432253298b8e2b6e38ade/compose/ui/ui/src/uikitMain/kotlin/androidx/compose/ui/interop/UIKitView.uikit.kt)
from Compose Multiplatform.

You can write this code purely in Kotlin or use Swift as well. In this example,
UIKit's [`MKMapView`](https://developer.apple.com/documentation/mapkit/mkmapview) component is displayed in Compose
Multiplatform. Set the component size by using the `Modifier.size(...)` or `Modifier.fillMaxSize()` functions from Compose
Multiplatform:

```kotlin
UIKitView(modifier = Modifier.size(300.dp), factory = { MKMapView() })
```

With this code, your application should look like this:

![MapView](mapview.png){width=300}

Now, let's look at an advanced example. This code wraps
UIKit's [`UITextField`](https://developer.apple.com/documentation/uikit/uitextfield/) in Compose Multiplatform:

```kotlin
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
```

The factory parameter contains the `editingChanged()` function and the `textField.addTarget()`listener to detect any
changes to `UITextField`. The `editingChanged()` function is annotated with `@ObjCAction` so that it can interoperate
with Objective-C code. The later `action` parameter passes the name of the function that should be called in response to
a `UIControlEventEditingChanged` event.

The next interesting part is the `update` parameter:

```kotlin
update = { textField ->
    textField.text = message
}
```

This function is called when the observable message state changes its value. The function updates the internal state of
the `UITextField` so that the user sees the updated value.

Explore the code for this example in
this [sample project](https://github.com/JetBrains/compose-multiplatform/tree/master/examples/interop/ios-uikit-in-compose).

## Use SwiftUI inside Compose Multiplatform

To use SwiftUI inside Compose Multiplatform, add your Swift code to an
intermediate [`UIViewController`](https://developer.apple.com/documentation/uikit/uiviewcontroller/).
Currently, you can't write SwiftUI structures directly in Kotlin. Instead, you have to write them in Swift
and pass them to a Kotlin function.

To start, add an argument to your entry point function to create a `ComposeUIViewController` component:

```kotlin
fun ComposeEntryPointWithUIView(createUIView: () -> UIView): UIViewController =
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

Then, pass the created `createUIViewController` to your Swift code. You can use a `UIHostingController` to wrap
SwiftUI views like this:

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
