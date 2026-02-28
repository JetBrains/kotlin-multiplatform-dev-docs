[//]: # (title: Explore composable code)

<secondary-label ref="IntelliJ IDEA"/>
<secondary-label ref="Android Studio"/>

<tldr>
    <p>This tutorial uses IntelliJ IDEA, but you can also follow it in Android Studio – both IDEs share the same core functionality and Kotlin Multiplatform support.</p>
    <br/>
    <p>This is the second part of the <strong>Create a Compose Multiplatform app with shared logic and UI</strong> tutorial. Before proceeding, make sure you've completed previous steps.</p>
    <p><img src="icon-1-done.svg" width="20" alt="First step"/> <a href="compose-multiplatform-create-first-app.md">Create your Compose Multiplatform app</a><br/>
      <img src="icon-2.svg" width="20" alt="Second step"/> <strong>Explore composable code</strong><br/>
      <img src="icon-3-todo.svg" width="20" alt="Third step"/> Modify the project<br/>      
      <img src="icon-4-todo.svg" width="20" alt="Fourth step"/> Create your own application<br/>
    </p>
</tldr>

Let's examine closely the sample composable created by the Kotlin Multiplatform wizard. First, there is the
composable `App()` function that implements the common UI and can be used on all platforms. Second, there is the
platform-specific code that launches this UI on each platform.

## Implementing composable functions

In the `shared/src/commonMain/kotlin/App.kt` file, take a look at the `App()` function:

<include from="compose-multiplatform-create-first-app.md" element-id="common-app-composable" />

The `App()` function is a regular Kotlin function annotated with `@Composable`. These kinds of functions are referred to
as _composable
functions_ or just _composables_. They are the building blocks of a UI based on Compose Multiplatform.

A composable function has the following general structure:

* The `MaterialTheme` sets the look of the application. The default settings can be customized. For example, you can
  choose colors, shapes, and typography.
* The `Column` composable controls the layout of the application. Here, it displays a `Button` above
  the `AnimatedVisibility` composable.
* The `Button` contains the `Text` composable, which renders some text.
* The `AnimatedVisibility` shows and hides the `Image` using an animation.
* The `painterResource` loads a vector icon stored as an XML file.

The `horizontalAlignment` parameter of the `Column` centers its content. But for this to have any effect, the column
should take up the full width of its container. This is achieved using the `modifier` parameter.

Modifiers are a key component of Jetpack Compose and Compose Multiplatform.
This is the primary mechanism you use to adjust the appearance or behavior of composables in the UI.
Modifiers are created using methods of the `Modifier` type. When you chain these
methods, each call can change the `Modifier` returned from the previous call, making the order significant.
See the [Compose Multiplatform introduction to modifiers](https://kotlinlang.org/docs/multiplatform/compose-layout-modifiers.html#built-in-modifiers)
and the extensive [Jetpack Compose modifier documentation](https://developer.android.com/jetpack/compose/modifiers) for more details.

### Managing the state

The final aspect of the sample composable is how the state is managed. The `showContent` property in the `App`
composable is built using the `mutableStateOf()` function, which means it's a state object that can be observed:

```kotlin
var showContent by remember { mutableStateOf(false) }
```

The state object is wrapped in a call to the `remember()` function, meaning that it's built once and then
retained by the framework. By executing this, you create a property whose value is a state object containing a boolean.
The framework caches this state object, allowing composables to observe it.

When the value of the state changes, any composables that observe it are re-invoked. This allows any of the widgets they
produce to be redrawn. This is called a _recomposition_.

In your application, the only place where the state is changed is in the click event of the button. The `onClick` event
handler flips the value of the `showContent` property. As a result, the image gets shown or hidden along with a `Greeting().greet()` call
because the parent `AnimatedVisibility` composable observes `showContent`.

## Launching UI on different platforms

The `App()` function execution is different for each platform. On Android, it's managed by an activity; on iOS, by a view 
controller; on the desktop, by a window; and on the web, by a container. Let's examine each of them.

### On Android

For Android, open the `MainActivity.kt` file within `androidApp/src/main/kotlin`:

```kotlin
class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        enableEdgeToEdge()
        super.onCreate(savedInstanceState)

        setContent {
            App()
        }
    }
}
```

This is an [Android activity](https://developer.android.com/guide/components/activities/intro-activities)
called `MainActivity` that invokes the `App` composable.

### On iOS

For iOS, open the `MainViewController.kt` file within `shared/src/iosMain/kotlin`:

```kotlin
fun MainViewController() = ComposeUIViewController { App() }
```

This is a [view controller](https://developer.apple.com/documentation/uikit/view_controllers) that performs the same
role as an activity on Android. Notice that both the iOS and Android types simply invoke the `App` composable from common code.

### On desktop

For desktop, look for the `main.kt` file in `desktopApp/src/main/kotlin`:

```kotlin
fun main() = application {
    Window(
        onCloseRequest = ::exitApplication,
        title = "ComposeDemo"
    ) {
        App()
    }
}
```

* Here, the `application()` function launches a new desktop application.
* This function takes a lambda, which initializes the UI. Typically, you create a `Window` and specify properties and
  instructions that dictate how the program should react when the window is closed (`onCloseRequest`).
  In this case, the whole application shuts down.
* Inside this window, you can place your content. As with Android and iOS, the only content is the UI provided by the `App()` function.

In this example, the `App()` function doesn't declare any parameters. In a larger application, you typically pass parameters to
platform-specific dependencies. These dependencies could be written manually or passed using a dependency injection library.

### On web

In the `main.kt` file within the `webApp/src/webMain/kotlin/` directory, take a look at the `main()` function:

```kotlin
@OptIn(ExperimentalComposeUiApi::class)
fun main() {
    ComposeViewport {
        App()
    }
}
```

* The `@OptIn(ExperimentalComposeUiApi::class)` annotation tells the compiler that you are using a Compose API marked as
  experimental and may change in future releases.
* The `ComposeViewport{}` function sets up the Compose environment for the application.
* The web app is inserted into the container specified as a parameter for the `ComposeViewport` function.
* The `App()` function is responsible for building the UI components of your application using Jetpack Compose.

## Next step

In the next part of the tutorial, you'll add a dependency to the project and modify the user interface.

**[Proceed to the next part](compose-multiplatform-modify-project.md)**

## Get help

* **Kotlin Slack**. Get an [invite](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up) and join
  the [#multiplatform](https://kotlinlang.slack.com/archives/C3PQML5NU) channel.
* **Kotlin issue tracker**. [Report a new issue](https://youtrack.jetbrains.com/newIssue?project=KT).
