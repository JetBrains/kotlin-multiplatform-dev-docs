[//]: # (title: Integration with the UIKit framework)

Compose Multiplatform is interoperable with the [UIKit](https://developer.apple.com/documentation/uikit) framework. You can embed Compose Multiplatform within an UIKit
application as well as embed native UIKit components within Compose Multiplatform. This page provides examples both
for using Compose Multiplatform inside a UIKit application and for embedding UIKit components inside Compose Multiplatform UI.

> To learn about SwiftUI interoperability, see the [](compose-swiftui-integration.md) article.
>
{style="tip"}

## Use Compose Multiplatform inside a UIKit application

To use Compose Multiplatform inside a UIKit application, add your Compose Multiplatform code to
any [container view controller]( https://developer.apple.com/documentation/uikit/view_controllers). This example uses Compose Multiplatform inside the `UITabBarController` class:

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

In this example, UIKit's [`MKMapView`](https://developer.apple.com/documentation/mapkit/mkmapview) component is displayed
in Compose Multiplatform. Set the component size by using the `Modifier.size()` or `Modifier.fillMaxSize()` functions
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

The factory parameter contains the `editingChanged()` function and the `textField.addTarget()` listener to detect any
changes to `UITextField`. The `editingChanged()` function is annotated with `@ObjCAction` so that it can interoperate
with Objective-C code. The `action` parameter of the `addTarget()` function later on passes the name of the `editingChanged()` function,
so it would be called in response to a `UIControlEventEditingChanged` event.

The `update` parameter of `UIKitView()` is called when the observable message state changes its value:

```kotlin
update = { textField ->
    textField.text = message
}
```

The function updates the `text` attribute of the `UITextField` so that the user sees the updated value.

Explore the code for this example in
this [sample project](https://github.com/JetBrains/compose-multiplatform/tree/master/examples/interop/ios-uikit-in-compose).

## What's next

You can also explore the way Compose Multiplatform can be [integrated with the SwiftUI framework](compose-swiftui-integration.md).