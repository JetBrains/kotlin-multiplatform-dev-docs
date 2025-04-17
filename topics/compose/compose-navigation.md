[//]: # (title: Navigation in Compose)

[Android's Navigation library](https://developer.android.com/guide/navigation) supports navigation in Jetpack Compose.
The Compose Multiplatform team contributes multiplatform support to the AndroidX Navigation library.

Apart from actual navigation between pieces of content within your app,
the library solves basic navigation problems:

* Pass data between destinations in a type-safe manner.
* Make it easy to track a user's journey through the app by keeping a clear and accessible navigation history.
* Support the mechanism of deep linking which allows navigating a user to a particular place in the app
  outside the usual workflows.
* Support uniform animations and transitions when navigating, as well as allow for common patterns such as
  back gestures with minimal additional work.

If you feel comfortable enough with the basics, move on to [Compose Multiplatform Navigation setup](compose-navigation-routing.md),
to learn how to leverage the Navigation library in cross-platform projects.
Otherwise, read on to learn about fundamental concepts the library works with. 

## Basic concepts of Compose Navigation

The Navigation library uses the following concepts to map navigation use cases to:

* _Navigation graph_. Describes all possible destinations within the app and connections between them.
  Navigation graphs can be nested to accommodate subflows in your app.
* _Destination._ A node in the navigation graph that can be navigated to.
    This can be a composable, a nested navigation graph, or a dialog.
    When the user navigates to the destination, the app displays its content.
* _Route_. Identifies a destination and defines arguments required to navigate to it.
    Routes are separate from destinations to allow you to keep each piece of UI implementation independent of the overall app structure.
    This makes it easier, for example, to test and rearrange composables in your project.

Keeping these concepts in mind, the Navigation library implements basic rules to guide your navigation architecture:

<!--* There is a fixed _start destination_, the first screen a user **usually** sees when they launch the app.
  Conditional screens like initial setup or login should not be considered
  start destinations even if they are unavoidable for a new user, think about the primary workflow.-->
<!-- Android introduces this concept, but in our docs there is no use for it so far. Maybe later on. -->

* A user's path in the app is represented as a stack of destinations, or _back stack_.
    By default, whenever the user is navigated to a new destination, that destination is added to the top of the stack.
    You can use the back stack to make the navigation more straightforward:
    instead of directly navigating back and forth, you can pop the current destination off the top of the stack
    and automatically return to the previous one.
* Each destination can have a set of _deep links_ associated with it:
    URI patterns that should lead to that destination when the app receives a link from the operating system.

## Basic navigation example

There is an order in which it makes sense to tackle the necessary steps to set up your navigation:

1. Define your routes.
    Create a [serializable](https://kotlinlang.org/docs/serialization.html)
    object or data class for each destination to hold the arguments that the corresponding destination requires.
2. Create a `NavController`, which will be your navigation interface, high enough in the composable hierarchy that all
   composables have access to it.
   The NavController holds the app's back stack and provides a method to transition between destinations in the navigation graph.
3. Design your navigation graph choosing one of the routes as the start destination.
   To do this, create a `NavHost` composable that holds the navigation graph (describes all navigable destinations).

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
    Provides APIs for core navigation functionality: transitioning between destinations,
    handling deep links, managing the back stack, and so on.
    <!--You should create the `NavController` high in your composable hierarchy, high enough that all the composables
    that need to reference it can do so.
    This way, you can use the `NavController` as the single source of truth for updating composables outside of your screens.
    [NB: This doesn't seem to be useful to people who are trying to cover the basics.]-->
* `NavHost`. Composable that displays the content for the current destination based on the navigation graph.
    Each NavHost has a required `startDestination` parameter: the destination that corresponds to the very first screen
    that users should see when they start the app.
* `NavGraph`.
    Describes all possible destinations within the app and the connections between them.
    Navigation graphs are usually defined as builder lambdas that return a `NavGraph`, for example, in `NavHost` declarations.

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

When designing your navigation graph, you can define routes as data classes with parameters, for instance:

```kotlin
@Serializable
data class Profile(val name: String)
```

To pass arguments to the destination, pass the arguments to the corresponding class constructor when navigating
to the destination.

```kotlin
Button(onClick = { navController.navigate(Profile("Alice")) }) {
    Text("Go to profile")
}
```

Then retrieve the data at the destination:

```kotlin
composable<Profile> { backStackEntry ->
    val profile: Profile = backStackEntry.toRoute()
    
    // Use `profile.name` wherever a user's name is needed.
}
```

### Retrieve complex data when navigating

When navigating between destinations, consider passing only the minimum necessary information between them.
Files or complex objects that reflect the state of the app in general should be stored in the data layer:
when the user lands on a destination, the UI should load the actual data from a single source of truth.

For example:

* **Don't** pass entire user profiles; **do** pass user IDs to retrieve profiles at the destination.
* **Don't** pass image objects; **do** pass URIs or file names that would allow loading an image from the source at the destination.
* **Don't** pass application states or ViewModels; **do** pass only the information necessary for the destination screen to work. 

This approach helps prevent data loss during configuration changes and any inconsistencies
when the referenced object is updated or mutated.   

See the [Android's article on data layer](https://developer.android.com/topic/architecture/data-layer)
for guidance on properly implementing a data layer in your app.

### Manage back stack

The back stack is controlled by the `NavController` class.
Like with any other stack, the `NavController` pushes new items onto the top of the stack and pops them from the top:

* When the app launches, the first entry that appears in the back stack is the start destination defined in NavHost.
* Each `NavController.navigate()` call by default pushes the given destination to the top of the stack.
* Using the back gesture, back button, or the `NavController.popBackStack()` method pops the current destination off the stack
    and returns the user to the previous destination.
    If the user followed a deep link to the current destination, popping the stack would return them to the previous app.
    Alternatively, the `NavController.navigateUp()` function only navigates the user within the app within the context
    of the `NavController`.

The Navigation library allows some flexibility with handling the back stack.
You can:

* Specify a particular destination in the back stack and navigate to it, popping everything on the stack
    that is on top of (came after) that destination.
* Navigate to destination X simultaneously popping the back stack up to destination Y
    (by adding a `popUpTo()` argument to a `.navigate()` call).
* Process popping an empty back stack (which would land the user on an empty screen).
* Maintain several back stacks for different parts of the app.
    For example, for apps with bottom navigation you can maintain separate nested graphs for each tab
    while saving and restoring navigation states when switching between tabs.
    Alternatively, you can create separate NavHosts for each tab, which makes the setup a bit more complex but
    may be easier to keep track of in some cases.

See [Jetpack Compose documentation on the back stack](https://developer.android.com/guide/navigation/backstack) for details
and use cases.

### Deep links
<primary-label ref="navEAP"/>

The Navigation library lets you associate a specific URI, action, or MIME type with a destination.
This association is called a _deep link_.

By default, deep links are not exposed to external apps: you need to register the appropriate URI scheme with the operating
system for each target distribution.



Deep links emulate manual navigation:
following a deep link replaces the current back stack with a synthetic one which looks like a reasonable path
from the start destination to the linked screen.
For routes with defined parameters, the library generates default deep link patterns.
<!--TODO this needs to be transferred to the deep link page and clarified: a path that looks like a path
will only be created for nested graphs, otherwise there is only going to be the start destination in the back stack.-->

For details on creating, registering, and handling deep links, see [TODOlink].

### Back gesture
<primary-label ref="navEAP"/>

The multiplatform Navigation library translates back gestures on each platform into navigating to the previous screen
(for example, on iOS this is a simple back swipe, and on desktop, the **Esc** key).

By default, on iOS the back gesture triggers native-like animation of the swipe transition to another screen.
If you customize the NavHost animation with `enterTransition` or `exitTransition` arguments, the default animation is not
going to trigger.

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