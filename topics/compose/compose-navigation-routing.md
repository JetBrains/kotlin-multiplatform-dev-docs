[//]: # (title: Navigation and routing)

Navigation is a key part of UI applications that allows users to move between different application screens.
Compose Multiplatform adopts the [Jetpack Compose approach to navigation](https://developer.android.com/guide/navigation/design#frameworks).

> The navigation library is currently in [Beta](supported-platforms.md#compose-multiplatform-ui-framework-stability-levels).
> You're welcome to try it in your Compose Multiplatform projects.
> We would appreciate your feedback in [YouTrack](https://youtrack.jetbrains.com/newIssue?project=CMP).
>
{style="tip"}

## Setup

To use the Navigation library, add the following dependency to your `commonMain` source set:

```kotlin
kotlin {
    // ...
    sourceSets {
        // ...
        commonMain.dependencies {
            // ...
            implementation("org.jetbrains.androidx.navigation:navigation-compose:%org.jetbrains.androidx.navigation%")
        }
        // ...
    }
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="org.jetbrains.androidx.navigation:navigation-compose:%org.jetbrains.androidx.navigation%"}

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
See the [](compose-lifecycle.md) page for details of the current implementation in Compose Multiplatform.

## Support for browser navigation in web apps
<secondary-label ref="Experimental"/>

Compose Multiplatform for web fully supports the common Navigation library APIs
and allows your apps to receive navigational input from the browser.
Users can use the **Back** and **Forward** buttons in the browser to move between navigation routes reflected in the browser history,
as well as use the address bar to understand where they are and get to a destination directly.

To bind the web app to the navigation graph defined in your common code,
you can use the `NavController.bindToBrowserNavigation()` method in your Kotlin/Wasm code. 
You can use the same method in Kotlin/JS, but wrap it in the `onWasmReady {}` block to ensure 
that the Wasm application is initialized and Skia is ready to render graphics.
Here is an example of how to set this up:

```kotlin
//commonMain source set
@Composable
fun App(
    onNavHostReady: suspend (NavController) -> Unit = {}
) {
    val navController = rememberNavController()
    NavHost(...) {
        //...
    }
    LaunchedEffect(navController) {
        onNavHostReady(navController)
    }
}

//wasmJsMain source set
@OptIn(ExperimentalComposeUiApi::class)
@ExperimentalBrowserHistoryApi
fun main() {
    val body = document.body ?: return
    ComposeViewport(body) {
        App(
          onNavHostReady = { it.bindToBrowserNavigation() }
        )
    }
}

//jsMain source set
@OptIn(ExperimentalComposeUiApi::class)
@ExperimentalBrowserHistoryApi
fun main() {
    onWasmReady {
        val body = document.body ?: return@onWasmReady
        ComposeViewport(body) {
            App(
                onNavHostReady = { it.bindToBrowserNavigation() }
            )
        }
    }
}
```

After calling `navController.bindToBrowserNavigation()`:
* The URL displayed in the browser reflects the current route (in the URL fragment, after the `#` character).
* The app parses URLs entered manually to translate them into destinations within the app.

By default, when using type-safe navigation, a destination is translated into a URL fragment according to
the [`kotlinx.serialization` default](https://kotlinlang.org/api/kotlinx.serialization/kotlinx-serialization-core/kotlinx.serialization/-serial-name/)
with appended arguments:
`<app package>.<serializable type>/<argument1>/<argument2>`.
For example, `example.org#org.example.app.StartScreen/123/Alice%2520Smith`.

### Customize translating routes into URLs and back

As Compose Multiplatform apps are single-page apps, the framework manipulates the address bar to imitate
usual web navigation.
If you would like to make your URLs more readable and isolate implementation from URL patterns,
you can assign the name to a screen directly or develop fully custom processing for destination routes:

* To simply make a URL readable, use the `@SerialName` annotation to explicitly set the serial name
    for a serializable object or class:

    ```kotlin
    // Instead of using the app package and object name,
    // this route will be translated to the URL simply as "#start"
    @Serializable @SerialName("start") data object StartScreen
    ```
* To fully construct every URL, you can use the optional `getBackStackEntryRoute` lambda.

#### Full URL customization

To implement a fully custom route to URL transformation: 

1. Pass the optional `getBackStackEntryRoute` lambda to the `navController.bindToBrowserNavigation()` function
    to specify how routes should be converted into URL fragments when necessary.
2. If needed, add code that catches URL fragments in the address bar (when someone clicks or pastes your app's URL)
    and translates URLs into routes to navigate users accordingly.

Here's an example of a simple type-safe navigation graph to use with the following samples of web code
(`commonMain/kotlin/org.example.app/App.kt`):

```kotlin
// Serializable object and classes for route arguments in the navigation graph
@Serializable data object StartScreen
@Serializable data class Id(val id: Long)
@Serializable data class Patient(val name: String, val age: Long)

@Composable
internal fun App(
    onNavHostReady: suspend (NavController) -> Unit = {}
) = AppTheme {
    val navController = rememberNavController()
    NavHost(
        navController = navController,
        startDestination = StartScreen
    ) {
        composable<StartScreen> {
            Column(
                modifier = Modifier.fillMaxSize(),
                horizontalAlignment = Alignment.CenterHorizontally,
                verticalArrangement = Arrangement.Center
            ) {
                Text("Starting screen")
                // Button that opens the 'Id' screen with a suitable parameter
                Button(onClick = { navController.navigate(Id(222)) }) {
                    Text("Pass 222 as a parameter to the ID screen")
                }
                // Button that opens the 'Patient' screen with suitable parameters
                Button(onClick = { navController.navigate(Patient( "Jane Smith-Baker", 33)) }) {
                    Text("Pass 'Jane Smith-Baker' and 33 to the Person screen")
                }
            }
        }
        composable<Id> {...}
        composable<Patient> {...}
    }
    LaunchedEffect(navController) {
        onNavHostReady(navController)
    }
}
```
{default-state="collapsed" collapsible="true" collapsed-title="NavHost(navController = navController, startDestination = StartScreen)"}

In `wasmJsMain/kotlin/main.kt`, add the lambda to the `.bindToBrowserNavigation()` call:

```kotlin
@OptIn(
    ExperimentalComposeUiApi::class,
    ExperimentalBrowserHistoryApi::class,
    ExperimentalSerializationApi::class
)
fun main() {
    val body = document.body ?: return
    ComposeViewport(body) {
        App(
            onNavHostReady = { navController ->
                navController.bindToBrowserNavigation() { entry ->
                    val route = entry.destination.route.orEmpty()
                    when {
                        // Identifies the route using its serial descriptor
                        route.startsWith(StartScreen.serializer().descriptor.serialName) -> {
                            // Sets the corresponding URL fragment to "#start"
                            // instead of "#org.example.app.StartScreen"
                            //
                            // This string must always start with the `#` character to keep
                            // the processing at the front end
                            "#start"
                        }
                        route.startsWith(Id.serializer().descriptor.serialName) -> {
                            // Accesses the route arguments
                            val args = entry.toRoute<Id>()

                            // Sets the corresponding URL fragment to "#find_id_222"
                            // instead of "#org.example.app.ID%2F222"
                            "#find_id_${args.id}"
                        }
                        route.startsWith(Patient.serializer().descriptor.serialName) -> {
                            val args = entry.toRoute<Patient>()
                            // Sets the corresponding URL fragment to "#patient_Jane%20Smith-Baker_33"
                            // instead of "#org.company.app.Patient%2FJane%2520Smith-Baker%2F33"
                            "#patient_${args.name}_${args.age}"
                        }
                        // Doesn't set a URL fragment for all other routes
                        else -> ""
                    }
                }
            }
        )
    }
}
```
<!--{default-state="collapsed" collapsible="true" collapsed-title="navController.bindToBrowserNavigation() { entry ->"}-->

> Make sure that every string that corresponds to a route starts with the `#` character to keep the data
> within URL fragments.
> Otherwise, when users copy and paste the URL, the browser will try to access a wrong endpoint instead of passing
> the control to your app.
> 
{style="note"}

If your URLs have custom formatting, you should add the reverse processing 
to match manually entered URLs to destination routes.
The code that does the matching needs to run before the `navController.bindToBrowserNavigation()` call binds
the browser location to the navigation graph:

<tabs>
    <tab title="Kotlin/Wasm">
        <code-block lang="Kotlin">
        @OptIn(
            ExperimentalComposeUiApi::class,
            ExperimentalBrowserHistoryApi::class,
            ExperimentalSerializationApi::class
        )
        fun main() {
            val body = document.body ?: return
            ComposeViewport(body) {
                App(
                    onNavHostReady = { navController ->
                        // Accesses the fragment substring of the current URL
                        val initRoute = window.location.hash.substringAfter('#', "")
                        when {
                            // Identifies the corresponding route and navigates to it
                            initRoute.startsWith("start") -> {
                                navController.navigate(StartScreen)
                            }
                            initRoute.startsWith("find_id") -> {
                                // Parses the string to extract route parameters before navigating to it
                                val id = initRoute.substringAfter("find_id_").toLong()
                                navController.navigate(Id(id))
                            }
                            initRoute.startsWith("patient") -> {
                                val name = initRoute.substringAfter("patient_").substringBefore("_")
                                val id = initRoute.substringAfter("patient_").substringAfter("_").toLong()
                                navController.navigate(Patient(name, id))
                            }
                        }
                        navController.bindToBrowserNavigation() { ... }
                    }
                )
            }
        }
        </code-block>
    </tab>
    <tab title="Kotlin/JS">
        <code-block lang="kotlin">
        @OptIn(
            ExperimentalComposeUiApi::class,
            ExperimentalBrowserHistoryApi::class,
            ExperimentalSerializationApi::class
        )
        fun main() {
            onWasmReady {
                val body = document.body ?: return@onWasmReady
                ComposeViewport(body) {
                    App(
                        onNavHostReady = { navController ->
                            // Accesses the fragment substring of the current URL
                            val initRoute = window.location.hash.substringAfter('#', "")
                            when {
                                // Identifies the corresponding route and navigates to it
                                initRoute.startsWith("start") -> {
                                    navController.navigate(StartScreen)
                                }
                                initRoute.startsWith("find_id") -> {
                                    // Parses the string to extract route parameters before navigating to it
                                    val id = initRoute.substringAfter("find_id_").toLong()
                                    navController.navigate(Id(id))
                                }
                                initRoute.startsWith("patient") -> {
                                    val name = initRoute.substringAfter("patient_").substringBefore("_")
                                    val id = initRoute.substringAfter("patient_").substringAfter("_").toLong()
                                    navController.navigate(Patient(name, id))
                                }
                            }
                            navController.bindToBrowserNavigation() { ... }
                        }
                    )
                }
            }
        }
        </code-block>
    </tab>
</tabs>