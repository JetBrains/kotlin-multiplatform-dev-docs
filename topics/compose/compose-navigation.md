[//]: # (title: Navigation in Compose)

Jetpack Compose relies on the [Android's Navigation component](https://developer.android.com/guide/navigation) 
to help you implement UI navigation between composables.
Below is a summary of the Navigation component's base principles and key features that should help you to better understand
the Compose approach to navigation.

If you feel comfortable enough with the basics, move on to [Compose Multiplatform navigation](compose-navigation-routing.md),
which leverages the Navigation component in cross-platform projects.

## Basic principles

The following principles set a baseline for a consistent and intuitive user experience in general:

* Each app should have a _fixed start destination_: the first screen that a user sees when they open the app
    (not counting conditional screens such as login or onboarding).
* Navigation state is represented as a stack of destinations.
    Each user's journey through app screens is reflected in the _back stack_: the top of the stack is the current screen,
    and previous destinations in the stack represent the history of where the user has been before.
    At the bottom of the stack is the start destination of the app.
    for details and use cases.
* _Deep linking_ simulates manual navigation: even if a deep link brings the user to a particular screen of the app,
    a realistic back stack should be synthesized using a route that could organically bring the user to the screen. 

## Features of the Navigation component

The Navigation component provides core types implementing the key concepts in navigation:

* _Graph_. Defines all possible navigation destinations within the app and the connections between them.
    Implemented in Compose as the `NavGraph` class.
* _Destination._ A composable node in the navigation graph.
    When the user navigates to the node, the app displays content corresponding to the destination.
    Implemented in Compose as the `NavDestination` class.
* _Route._ Identifies a destination and any data required by it. Can be implemented as any serializable data type.
    Routes are separated from destinations to allow you to keep destination composables independent of navigation,
    which makes them easier to test and rearrange in your project.
* _Host_. Contains the current navigation destination. As the user navigates through an app, the app swaps destinations
  in and out of the navigation host. Implemented in Compose as the `NavHost` class.
* _Controller_. Offers core coordination functionality: methods for transitioning between destinations,
    handling deep links, managing the back stack and so on. Implemented in Compose as the `NavController` class.

Besides the core types functionality, the Navigation component provides animations and transitions, support for deep linking,
type safety, `ViewModel` support, and other quality-of-life features for handling app navigation.

To read up on the Navigation component in depth, see Android's [Navigation article](https://developer.android.com/guide/navigation).

## General sequence of implementing navigation

While you need all of the key parts listed above,
there is an order in which it makes the most sense to tackle them:

1. Define your routes. Use a serializable [TODO link?] object or class that describes how to get to a destination and contains all the data
    that the destination requires.
2. Design your navigation graph choosing one of the routes as the start destination.
    In Compose, you create a `NavHost` composable and add the `NavGraph` to it in one of two ways:
    * Directly, as part of creating the NavHost.
    * Programmatically, using the `NavController.createGraph()` method to create the `NavGraph` that you pass to the `NavHost`.
3. Create your `NavController`, high enough in the composable hierarchy that all composables have access to it.
    This NavController will hold the app's back stack and provide a way to transition between destinations in the navigation graph.

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

## Supported navigation scenarios

### Pass arguments to a destination

When designing your navigation graph, you can define routes with classes that have parameters, for instance:

```kotlin
@Serializable
data class Profile(val name: String)
```

When you need to pass arguments to that destination, create an instance of the route class and pass the arguments
to the class constructor.

Navigation arguments are not designed for passing complex data objects.
Consider passing the minimum necessary information, such as unique identifiers of complex objects stored in the data layer.
Using a single source of truth, such as the data layer, helps avoid transmitting complex data between navigation graph destinations.

Starting with the 1.7.0 version, Compose Multiplatform supports type-safe navigation in common code.
This means that as long as you define routes as serializable objects or classes,
the framework will provide compile-time type safety for your navigation graph.

> This is not a requirement: you can still use plain strings to identify your routes
> with Compose Navigation, but you will have to keep track of route names and arguments.
>
{style="note"}

To learn about Navigation Compose type safety in detail, see the [Jetpack's Type safety article](https://developer.android.com/guide/navigation/design/type-safety).

### Managing UI state

Complex objects that reflect the state of the app in general should be stored in the data layer:
when the user lands on a destination, it loads the necessary information from a single source of truth.
This approach helps prevent data loss during configuration changes and any inconsistencies
when the referenced object is updated or mutated.

https://developer.android.com/develop/ui/compose/navigation#retrieving-complex-data

### Back stack

The back stack is held by and controlled by the `NavController` class.
Just like with any other stack, the `NavController` pushes new items onto the top of the stack and pops them from the top
as well.

* When the user opens the app, the `NavController` pushes the start destination into the back stack.
* Each following `NavController.navigate()` call pushes the given destination to the top of the stack.
* Using the back gesture or back button pops the current destination off the stack.
    If the user followed a deep link to the current destination, popping the stack would return them to the previous app.
    Alternatively, the `NavController.navigateUp()` function only navigates the user within the app within the context
    of the `NavController`.

See [JetPack Compose documentation on back stack](https://developer.android.com/guide/navigation/backstack) for details
and use cases.

### Deep links
<primary-label ref="EAP"/>

Compose Multiplatform Navigation lets you associate a specific URL, action or MIME type with a composable using deep linking.
By default, deep links are not exposed to external apps: you need to register the appropriate URI schemas for each
target.

For details on implementing deep links, see [TODO link].

### Predictive back gesture
<primary-label ref="EAP"/>

> This feature is available only when the project depends on the [multiplatform Navigation library](compose-navigation-routing.md)
> of at least `2.9.0-alpha15` version.
>
{style="note"}

Compose Multiplatform adopts the [Android predictive back approach](https://developer.android.com/guide/navigation/custom-back/predictive-back-gesture)
to be used on other platforms.
The multiplatform Navigation library provides the universal back gesture support
and translates back gestures on each platform into navigating to the previous screen
(for iOS, this is a simple back swipe, for desktop – the **Esc** key).

For now, there is no user-facing API for custom implementations of the `BackHandler` class.

## Alternative navigation solutions

If the Compose-based navigation implementation does not work for you,
there are third-party alternatives to evaluate:

| Name                                                | Description                                                                                                                                                     |
|-----------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Voyager](https://voyager.adriel.cafe)              | A pragmatic approach to navigation                                                                                                                              |
| [Decompose](https://arkivanov.github.io/Decompose/) | An advanced approach to navigation that covers the full lifecycle and any potential dependency injection                                                        |
| [Appyx](https://bumble-tech.github.io/appyx/)       | Model-driven navigation with gesture control                                                                                                                    |
| [PreCompose](https://tlaster.github.io/PreCompose/) | A navigation and view model inspired by Jetpack Lifecycle, ViewModel, LiveData, and Navigation                                                                  |
| [Circuit](https://slackhq.github.io/circuit/)       | A Compose-driven architecture for Kotlin applications with navigation and advanced state management.                                                            |
