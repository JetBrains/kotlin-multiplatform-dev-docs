[//]: # (title: Navigation 3 in Compose Multiplatform)
<primary-label ref="alpha"/>

[Android's Navigation library](https://developer.android.com/guide/navigation) is upgraded to Navigation 3, a redesigned approach to navigation that works with
Compose and takes into account feedback to the previous version of the library.
Starting with the version 1.10, Compose Multiplatform helps adopt Navigation 3 into multiplatform projects.

## General overview

Navigation 3 is more than a new version of the library — in a lot of ways it's a new library entirely.
To dive deeper into the philosophy behind it, see the [Android Developers blog post](https://android-developers.googleblog.com/2025/05/announcing-jetpack-navigation-3-for-compose.html).

In short, the major changes in the new library are:

* **User-owned back stack**. Instead of operating a single library back stack,
  you create and manage a `SnapshotStateList` of states which is observed by the UI.
* **Low-level building blocks**. Thanks to closer integration with Compose, the library allows more flexibility
  in implementing your own navigation components and behavior.
* **Adaptive layout system** that allows to display multiple destinations at the same time and seamlessly switch between layouts. 

Navigation 3 is closely aligned with Compose and therefore a navigation implementation written for Android works in common
Compose Multiplatform code almost without changes.
The only thing you need to add to support non-JVM platforms like web and iOS is implement
[polymorphic serialization for destination keys](#polymorphic-serialization-for-destination-keys). 

> You can find and compare extensive examples of Android and multiplatform implementations on GitHub:
> * [original Android repository with Navigation 3 recipes](https://github.com/android/nav3-recipes)
> * [Compose Multiplatform project with most of the same recipes](https://github.com/terrakok/nav3-recipes)

Learn more about general design of Navigation 3 in [Android documentation](https://developer.android.com/guide/navigation/navigation-3).

### Polymorphic serialization for destination keys

On Android, Navigation 3 relies on reflection-based serialization, which is not available when you target non-JVM platforms like iOS.
To take this into account, the library has two overloads for the `rememberNavBackStack()` function:

* [The first overload](https://developer.android.com/reference/kotlin/androidx/navigation3/runtime/package-summary#rememberNavBackStack(kotlin.Array))
  only takes a set of `NavKey` and requires a reflection-based serializer.
* [The second overload](https://developer.android.com/reference/kotlin/androidx/navigation3/runtime/package-summary#rememberNavBackStack(androidx.savedstate.serialization.SavedStateConfiguration,kotlin.Array))
  also takes a `SavedStateConfiguration` parameter that allows you to provide a `SerializersModule` and handle open polymorphism
  correctly across all platforms.

In the multiplatform examples of Navigation 3 this looks [like this](https://github.com/terrakok/nav3-recipes/blob/8ff455499877225b638d5fcd82b232834f819422/sharedUI/src/commonMain/kotlin/com/example/nav3recipes/basicdsl/BasicDslActivity.kt#L40),
for example:

```kotlin
@Serializable
private data object RouteA : NavKey

@Serializable
private data class RouteB(val id: String) : NavKey


private val config = SavedStateConfiguration {
    serializersModule = SerializersModule {
        polymorphic(NavKey::class) {
            subclass(RouteA::class, RouteA.serializer())
            subclass(RouteB::class, RouteB.serializer())
        }
    }
}

@Composable
fun BasicDslActivity() {
    val backStack = rememberNavBackStack(config, RouteA)

    NavDisplay(
        backStack = backStack,
        //...
    )
}
```

## Setup of the dependencies

To try out the multiplatform implementation of Navigation 3,
add the following dependency to your version catalog:

```text
[versions]
compose-multiplatform-navigation3 = "1.0.0-alpha05"

[libraries]
jetbrains-navigation3-ui = { module = "org.jetbrains.androidx.navigation3:navigation3-ui", version.ref = "compose-multiplatform-navigation3" }
```

Additional support for Navigation 3 is implemented in Material 3 Adaptive and ViewModel.
If you're using these libraries, add the navigation support artifacts as well:
```text
[versions]
compose-multiplatform-adaptive = "1.3.0-alpha02"
compose-multiplatform-lifecycle = "2.10.0-alpha05"

[libraries]
jetbrains-material3-adaptiveNavigation3 = { module = "org.jetbrains.compose.material3.adaptive:adaptive-navigation3", version.ref = "androidx-adaptive" }
jetbrains-lifecycle-viewmodelNavigation3 = { module = "org.jetbrains.androidx.lifecycle:lifecycle-viewmodel-navigation3", version.ref = "compose-multiplatform-lifecycle" }
```

Finally, you can try out the [proof-of-concept library](https://github.com/terrakok/navigation3-browser)
authored by a JetBrains engineer that adds support for Navigation 3 to web targets:

```text
[versions]
navigation3-browser = "0.2.0"

[libraries]
navigation3-browser = { module = "com.github.terrakok:navigation3-browser", version.ref = "navigation3-browser" }
```

> Web support is expected to be added to the base multiplatform Navigation 3 library in version 1.1.0. 
>
{style="note"}

## What's next

Compose navigation is covered in-depth on the Android Developer portal.
Sometimes this documentation uses Android-only examples,
but fundamental guidance and navigation principles are the same for Multiplatform:

* [Overview of navigation with Compose](https://developer.android.com/develop/ui/compose/navigation).
* [Starting page for Jetpack Navigation](https://developer.android.com/guide/navigation) with subpages about
    navigation graphs, moving around them, and other navigation use cases.