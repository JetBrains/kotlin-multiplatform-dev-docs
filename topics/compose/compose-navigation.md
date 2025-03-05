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
* _Deep linking_ simulates manual navigation: even if a deep link brings the user to a particular screen of the app,
    a realistic back stack should be synthesized using a route that could organically bring the user to the screen. 

## Features of the Navigation component

The Navigation component provides core types implementing the key concepts in navigation:

* _Graph_. Defines all possible navigation destinations within the app and the connections between them.
    Implemented in Compose as the `NavGraph` class.
* _Destination._ A node in the navigation graph. When the user navigates to the node, the app displays content corresponding
    to the destination. Implemented in Compose as the `NavDestination` class.
* _Route._ Identifies a destination and any data required by it. Can be implemented as any serializable data type.
* _Host_. Contains the current navigation destination. As the user navigates through an app, the app swaps destinations
  in and out of the navigation host. Implemented in Compose as the `NavHost` class.
* _Controller_. Offers core coordination functionality: methods for transitioning between destinations,
    handling deep links, managing the back stack and so on. Implemented in Compose as the `NavController` class.

Besides the core types functionality, the Navigation component provides animations and transitions, support for deep linking,
type safety, `ViewModel` support, and other quality-of-life features for handling app navigation.

To read on the Navigation component in depth, see [Navigation](https://developer.android.com/guide/navigation) in the Android documentation
on app architecture.

## General sequence of implementing navigation

While you need all of the key components listed above, there is an order in which it makes most sense to tackle creating them:

1. Define your routes. Use a serializable [TODO link?] object or class that describes how to get to a destination and contains all the data
    that the destination requires.
2. Design your navigation graph, choosing one of the routes as the start destination.
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

### Pass arguments to a destination

### Obtain route instance

### Minimal example

https://developer.android.com/guide/navigation/design#compose-minimal
[TODO make sure that the minimal example works with CMP]

instead of passing the NavController to your composables, expose an event to the NavHost