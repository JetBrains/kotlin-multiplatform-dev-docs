[//]: # (title: Navigation in Compose)

Jetpack Compose relies on the [Android's Navigation component](https://developer.android.com/guide/navigation) 
to help you implement UI navigation between composables.
Below is a summary of the Navigation component's key features that should help you to better understand
the Compose approach to navigation.

The Navigation component solves the following basic problems:

* Clarify requirements of each screen: which data needs to be available when a user navigates to a composable.
* Make it easy to track a user's journey through the app by keeping a clear and accessible navigation history.
* Support the operating system mechanism of deep linking which allows navigating a user to a particular point of the app
  outside the usual workflow.
* Support uniform animations and transitions when navigating, as well as allow for common patterns such as
  back handling with minimal additional work.

If you feel comfortable enough with the basics, move on to [Compose Multiplatform navigation](compose-navigation-routing.md),
which leverages the Navigation component in cross-platform projects.

## Basic concepts of the Navigation component

The Navigation component offers the following concepts to map your app workflows to:

* _Destination._ A composable that can be navigated to.
* _Route_. A data type that identifies a destination and defines data required to navigate to it.
    Routes identify destinations but are separate from them to allow you to keep destination composables independent of navigation.
    This makes it easier to test and rearrange composables in your project.
* _Navigation graph_. Describes all possible destinations within the app and connections between them.
    Navigation graphs can be nested to accomodate subflows in your app.

Using these concepts, the Navigation component implements basic rules that guide your navigation:

* There is a fixed _start destination_ (not counting conditional screens for setup or login). 
* A user's path in the app is represented as a stack of destinations, or _back stack_.
    Whenever the user navigates to a different screen, it is added to the top of the stack.
    When the user uses a back gesture or button, the top of the stack is popped, and they return to the previous destination
    recorded in the stack.
* Links that lead inside the app from other apps or the browser (usually called [_deep links_](#deep-links)).

## Basic navigation example

There is an order in which it makes sense to tackle the necessary steps to set up your navigation:

1. Define your routes.
    Use a serializable [TODO link?] object or class that holds the data that the destination requires as its arguments.
2. Create a `NavController`, which will be your navigation interface, high enough in the composable hierarchy that all
   composables have access to it.
   The NavController will hold the app's back stack and provide a method to transition between destinations in the navigation graph.
3. Design your navigation graph choosing one of the routes as the start destination.
   To do this, create a `NavHost` composable that holds the navigation graph and describe all navigable destinations
   as part of the declaration.

The following is a basic example of a foundation for navigating within an app:

```kotlin
// Creates routes.
@Serializable
object Profile
@Serializable
object FriendsList

// Creates the NavController.
val navController = rememberNavController()

// Creates the NavHost with the navigation graph consisting of supplied destinations.
NavHost(navController = navController, startDestination = Profile) {
    composable<Profile> { ProfileScreen( /* ... */ ) }
    composable<FriendsList> { FriendsListScreen( /* ... */ ) }
    // You can add more destinations similarly.
}
```

### Main classes of the Navigation library

The Navigation library provides the following core types:

* `NavController`.
    Contains core navigation functionality: methods for transitioning between destinations,
    handling deep links, managing the back stack, and so on.
    <!--You should create the `NavController` high in your composable hierarchy, high enough that all the composables
    that need to reference it can do so.
    This way, you can use the `NavController` as the single source of truth for updating composables outside of your screens.
    [NB: This doesn't seem to be useful to people who are trying to cover the basics.]-->
* `NavHost`.
    Binds a `NavController` to a navigation graph.
    A navigation graph is usually defined as a lambda in a `NavHost` call that builds a `NavGraph` instance from a set of composables.
* `NavGraph`.
    Contains all possible navigation destinations within the app and the connections between them.
    Navigation graphs are defined as builder lambdas that return a `NavGraph`, for example, in `NavHost` declarations.
* `NavDestination`.
    Contains a navigable composable.
    Apart from the composable content, holds the deep links assigned to the destination.
    Destinations are defined as `composable<T>()` calls within a navigation graph builder.

Besides the core types functionality, the Navigation component provides animations and transitions, support for deep linking,
type safety, `ViewModel` support, and other quality-of-life features for handling app navigation.



## Navigation use cases

### Go to a destination

To navigate to a destination, call the `NavController.navigate()` function. To continue the example above:

```kotlin
Button(onClick = { navController.navigate(Profile) }) {
    Text("Go to profile")
}
```

### Pass arguments to a destination

When designing your navigation graph, you can define routes as classes with parameters, for instance:

```kotlin
@Serializable
data class Profile(val id: String)
```

When you need to pass arguments to that destination, pass the arguments to the class constructor when navigating
to the destination.

### Retrieve complex data when navigating

When navigating, consider passing the minimum necessary information such as unique identifiers of complex objects stored in the data layer.
Complex objects that reflect the state of the app in general should be stored in the data layer:
when the user lands on a destination, it loads the necessary information from a single source of truth.

This approach helps prevent data loss during configuration changes and any inconsistencies
when the referenced object is updated or mutated.

See the [Android's article on data layer](https://developer.android.com/topic/architecture/data-layer)
for guidance properly implementing a data layer in your app.

### Manage back stack

The back stack is held by and controlled by the `NavController` class.
Just like with any other stack, the `NavController` pushes new items onto the top of the stack and pops them from the top
as well:

* When the user opens the app, the `NavController` pushes the start destination into the back stack.
* Each following `NavController.navigate()` call pushes the given destination to the top of the stack.
* Using the back gesture, back button, or the `NavController.popBackStack()` method pops the current destination off the stack.
    If the user followed a deep link to the current destination, popping the stack would return them to the previous app.
    Alternatively, the `NavController.navigateUp()` function only navigates the user within the app within the context
    of the `NavController`.

See [Jetpack Compose documentation on back stack](https://developer.android.com/guide/navigation/backstack) for details
and use cases.

### Deep links
<primary-label ref="navEAP"/>

Compose Multiplatform Navigation lets you associate a specific URL, action or MIME type with a composable using deep linking.
By default, deep links are not exposed to external apps: you need to register the appropriate URI schemas for each
target.

Deep links emulate manual navigation:
following a deep link replaces the current back stack with a synthetic one which looks like a reasonable path
from the start destination to the linked screen.

For details on registering implementing deep links, see [TODOlink].

### Back gesture
<primary-label ref="navEAP"/>

Compose Multiplatform adopts Android back gesture handling to automatically navigate to the previous destination
in the back stack when the user uses the gesture.

The Compose Multiplatform Navigation library provides the universal back gesture support
and translates back gestures on each platform into navigating to the previous screen
(for iOS, this is a simple back swipe, for desktop – the **Esc** key).
On iOS, 

For now, there is no user-facing API for custom implementations of the `BackHandler` class.

### Creating a navigation graph dynamically

Sometimes you may want to implement conditional navigation:
create different navigation graphs for users with different access levels,
or create custom onboarding experiences for users from different countries (taking European users through a GDPR screen, for example).

In these cases, you can create navigation graphs programmatically using the `NavController.createGraph()` method:

```kotlin
val navGraph = navController.createGraph(startDestination = startStep) {
    if (needsGDPR) {
        composable("gdpr") { ... }
    }
    composable("step1") { ... }
    if (userNeedsExtraStep) {
        composable("extraStep") { ... }
    }
}
navController.graph = navGraph
```


## Alternative navigation solutions

If the Compose-based navigation implementation does not work for you,
there are third-party alternatives to evaluate:

| Name                                                | Description                                                                                                                                                     |
|-----------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Voyager](https://voyager.adriel.cafe)              | A pragmatic approach to navigation                                                                                                                              |
| [Decompose](https://arkivanov.github.io/Decompose/) | An advanced approach to navigation that covers the full lifecycle and any potential dependency injection                                                        |
| [Circuit](https://slackhq.github.io/circuit/)       | A Compose-driven architecture for Kotlin applications with navigation and advanced state management.                                                            |
| [Appyx](https://bumble-tech.github.io/appyx/)       | Model-driven navigation with gesture control                                                                                                                    |
| [PreCompose](https://tlaster.github.io/PreCompose/) | A navigation and view model inspired by Jetpack Lifecycle, ViewModel, LiveData, and Navigation                                                                  |


## What's next

Compose navigation is covered in-depth on the Android Developer portal.
Sometimes this documentation uses Android-only examples,
but fundamental guidance and navigation principles are the same for Multiplatform:

* [Overview of navigation with Compose](https://developer.android.com/develop/ui/compose/navigation).
* [Starting page for Jetpack Navigation](https://developer.android.com/guide/navigation) with subpages about
    navigation graphs, moving around them, and other navigation use cases.