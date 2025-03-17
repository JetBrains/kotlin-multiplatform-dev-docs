[//]: # (title: Navigation and routing)

> The navigation library is currently [Experimental](supported-platforms.md#core-kotlin-multiplatform-technology-stability-levels).
> You're welcome to try it in your Compose Multiplatform projects.
> We would appreciate your feedback in [YouTrack](https://youtrack.jetbrains.com/newIssue?project=CMP).
>
{style="warning"}

Navigation is a key part of UI applications that allows users to move between different application screens.
Compose Multiplatform adopts the [Jetpack Compose approach to navigation](https://developer.android.com/guide/navigation/design#frameworks).

## Setup

To use the navigation library, add the following dependency to your `commonMain` source set:

```kotlin
kotlin {
    // ...
    sourceSets {
        // ...
        commonMain.dependencies {
            // ...
            implementation("org.jetbrains.androidx.navigation:navigation-compose:%composeNavigationVersion%")
        }
        // ...
    }
}
```

## Type-safe navigation

Starting with 1.7.0, Compose Multiplatform supports type-safe navigation in common code as described in
the [Jetpack documentation](https://developer.android.com/guide/navigation/design/type-safety).

## Sample project

To see the Compose Multiplatform navigation library in action, check out the [nav_cupcake project](https://github.com/JetBrains/compose-multiplatform/tree/master/examples/nav_cupcake),
which was converted from the [Navigate between screens with Compose](https://developer.android.com/codelabs/basic-android-kotlin-compose-navigation#0)
Android codelab.

Just as with Jetpack Compose, to implement navigation, you should:
1. [List routes](https://github.com/JetBrains/compose-multiplatform/blob/a6961385ccf0dee7b6d31e3f73d2c8ef91005f1a/examples/nav_cupcake/composeApp/src/commonMain/kotlin/org/jetbrains/nav_cupcake/CupcakeScreen.kt#L50)
   that should be included in the navigation graph. Each route must be a unique string that defines a path.
2. [Create a `NavHostController` instance](https://github.com/JetBrains/compose-multiplatform/blob/a6961385ccf0dee7b6d31e3f73d2c8ef91005f1a/examples/nav_cupcake/composeApp/src/commonMain/kotlin/org/jetbrains/nav_cupcake/CupcakeScreen.kt#L89)
   as your main composable property to manage navigation.
3. [Add a `NavHost` composable](https://github.com/JetBrains/compose-multiplatform/blob/a6961385ccf0dee7b6d31e3f73d2c8ef91005f1a/examples/nav_cupcake/composeApp/src/commonMain/kotlin/org/jetbrains/nav_cupcake/CupcakeScreen.kt#L109)
   to your app:
    1. Choose the starting destination from the list of routes you defined earlier.
    2. Create a navigation graph, either directly, as part of creating a `NavHost`, or programmatically, using the
       `NavController.createGraph()` function.

Each back stack entry (each navigation route included in the graph) implements the `LifecycleOwner` interface.
A switch between different screens of the app makes it change its state from `RESUMED` to `STARTED` and back.
`RESUMED` is also described as "settled": navigation is considered finished when the new screen is prepared and active.
See the [Lifecycle](compose-lifecycle.md) page for details of the current implementation in Compose Multiplatform.

## Limitations

Current limitations of navigation in Compose Multiplatform, compared to Jetpack Compose:
* [Deep links](https://developer.android.com/guide/navigation/design/deep-link) (handling or following them) are not supported.
* The [BackHandler](https://developer.android.com/develop/ui/compose/libraries#handling_the_system_back_button) function
  and [predictive back gestures](https://developer.android.com/guide/navigation/custom-back/predictive-back-gesture)
  are not supported on any platform besides Android.

## Support for browser navigation in web apps
<primary-label ref="EAP"/>

Compose Multiplatform for web fully supports the common Navigation library APIs,
and on top of that allows your apps to receive navigational input from the browser.
Users can use the **Back** and **Forward** buttons in the browser to move through the back stack,
as well as input URLs with valid routes directly to get to a particular destination.

To bind your web app presentation to the navigation graph defined in your common code,
call the `window.bindToNavigation()` method in your Kotlin/Wasm code:

```kotlin
@OptIn(ExperimentalComposeUiApi::class)
@ExperimentalBrowserHistoryApi
fun main() {
    val body = document.body ?: return
    
    ComposeViewport(body) {
        
        val navController = rememberNavController()
        // Assuming that your main composable function in common code is App()
        App(navController)
        LaunchedEffect(Unit) {
            window.bindToNavigation(navController)
        }
    }
}
```

### Translating routes into URLs and back

User of a website usually expects that the URL displayed in the browser address bar contains all necessary information
about where they are within the app.
So, a URL that is copied and pasted into another browser should lead to the same screen.

As Compose apps are single-page apps, the URL does not change automatically – the relation between
navigation and the displayed address needs to be implemented artificially.
Compose Multiplatform does just that: it alters the displayed URL according to the current destination
and catches changes made to the address bar by the user to parse their intention to move to a different screen.

By default, 

## Third-party alternatives

If the Compose Multiplatform navigation components do not solve your problems,
there are third-party alternatives that you can choose from:

| Name                                                | Description                                                                                                                                                     |
|-----------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Voyager](https://voyager.adriel.cafe)              | A pragmatic approach to navigation                                                                                                                              |
| [Decompose](https://arkivanov.github.io/Decompose/) | An advanced approach to navigation that covers the full lifecycle and any potential dependency injection                                                        |
| [Appyx](https://bumble-tech.github.io/appyx/)       | Model-driven navigation with gesture control                                                                                                                    |
| [PreCompose](https://tlaster.github.io/PreCompose/) | A navigation and view model inspired by Jetpack Lifecycle, ViewModel, LiveData, and Navigation                                                                  |
| [Circuit](https://slackhq.github.io/circuit/)       | A Compose-driven architecture for Kotlin applications with navigation and advanced state management.                                                            |