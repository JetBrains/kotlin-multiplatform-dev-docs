[//]: # (title: Navigation 3 in Compose Multiplatform)
<primary-label ref="alpha"/>

[Android's Navigation library](https://developer.android.com/guide/navigation) has been upgraded to Navigation 3, introducing a redesigned approach to navigation that works with
Compose and takes into account feedback to the previous version of the library.
Starting with version 1.10, Compose Multiplatform supports adopting Navigation 3 in multiplatform projects for all supported platforms:
Android, iOS, desktop, and web.

## Key changes

Navigation 3 is more than a new version of the library — in a lot of ways it's a new library entirely.
To learn more about the philosophy behind this redesign, see the [Android Developers blog post](https://android-developers.googleblog.com/2025/05/announcing-jetpack-navigation-3-for-compose.html).

Key changes in Navigation 3 include:

* **User-owned back stack**. Instead of operating a single library back stack,
  you create and manage a `SnapshotStateList` of states, which the UI observes directly.
* **Low-level building blocks**. Thanks to closer integration with Compose, the library allows more flexibility
  in implementing your own navigation components and behavior.
* **Adaptive layout system**. With adaptive design, you can display multiple destinations at the same time
  and seamlessly switch between layouts. 

Learn more about general design of Navigation 3 in [Android documentation](https://developer.android.com/guide/navigation/navigation-3).

## Dependencies setup

To try out the multiplatform implementation of Navigation 3, add the following dependency to your version catalog:

```text
[versions]
multiplatform-nav3-ui = "1.0.0-alpha05"

[libraries]
jetbrains-navigation3-ui = { module = "org.jetbrains.androidx.navigation3:navigation3-ui", version.ref = "multiplatform-nav3-ui" }
```

> While Navigation 3 is released as two artifacts, `navigation3:navigation3-ui` and `navigation3:navigation3-common`,
> only `navigation3-ui` has a separate Compose Multiplatform implementation.
> A dependency on `navigation3-common` is added transitively.
>
{style="note"}

For projects using the Material 3 Adaptive and ViewModel libraries, also add the following navigation support artifacts:
```text
[versions]
compose-multiplatform-adaptive = "1.3.0-alpha02"
compose-multiplatform-lifecycle = "2.10.0-alpha05"

[libraries]
jetbrains-material3-adaptiveNavigation3 = { module = "org.jetbrains.compose.material3.adaptive:adaptive-navigation3", version.ref = "compose-multiplatform-adaptive" }
jetbrains-lifecycle-viewmodelNavigation3 = { module = "org.jetbrains.androidx.lifecycle:lifecycle-viewmodel-navigation3", version.ref = "compose-multiplatform-lifecycle" }
```

Finally, you can try out the [proof-of-concept library](https://github.com/terrakok/navigation3-browser)
created by a JetBrains engineer. The library integrates the multiplatform Navigation 3 with browser history navigation on the web:

```text
[versions]
compose-multiplatform-navigation3-browser = "0.2.0"

[libraries]
navigation3-browser = { module = "com.github.terrakok:navigation3-browser", version.ref = "compose-multiplatform-navigation3-browser" }
```

Browser history navigation is expected to be supported by the base multiplatform Navigation 3 library in version 1.1.0.

## Multiplatform support

Navigation 3 is closely aligned with Compose, allowing an Android navigation implementation to work in shared
Compose Multiplatform code with minimal changes.
To support non-JVM platforms like web and iOS, the only thing you need is to implement
[polymorphic serialization for destination keys](#polymorphic-serialization-for-destination-keys). 

You can compare extensive examples of Android-only and multiplatform apps using Navigation 3 on GitHub:
* [original Android repository with Navigation 3 recipes](https://github.com/android/nav3-recipes)
* [Compose Multiplatform project with most of the same recipes](https://github.com/terrakok/nav3-recipes)

### Polymorphic serialization for destination keys

On Android, Navigation 3 relies on reflection-based serialization, which is not available when you target non-JVM platforms like iOS.
To address this limitation, the library has two overloads for the `rememberNavBackStack()` function:

* [The first overload](https://developer.android.com/reference/kotlin/androidx/navigation3/runtime/package-summary#rememberNavBackStack(kotlin.Array))
  only takes a set of `NavKey` references and requires a reflection-based serializer.
* [The second overload](https://developer.android.com/reference/kotlin/androidx/navigation3/runtime/package-summary#rememberNavBackStack(androidx.savedstate.serialization.SavedStateConfiguration,kotlin.Array))
  also takes a `SavedStateConfiguration` parameter that allows you to provide a `SerializersModule` and handle open polymorphism
  correctly across all platforms.

In the Navigation 3 [multiplatform examples](https://github.com/terrakok/nav3-recipes/blob/8ff455499877225b638d5fcd82b232834f819422/sharedUI/src/commonMain/kotlin/com/example/nav3recipes/basicdsl/BasicDslActivity.kt#L40), 
you define your routes and a `SavedStateConfiguration` to register them:

```kotlin
@Serializable
private data object RouteA : NavKey

@Serializable
private data class RouteB(val id: String) : NavKey

// Creates the required serialization configuration for open polymorphism
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
    // Consumes the serialization configuration
    val backStack = rememberNavBackStack(config, RouteA)

    NavDisplay(
        backStack = backStack,
        //...
    )
}
```

### Recommended serialization approaches

When implementing multiplatform navigation, you'll need to choose how to organize and serialize your route definitions.
Depending on your project's complexity and modularization, use one of the following three patterns.

#### Single module with sealed type

For small projects where all routes exist in one module, use a sealed interface. 
This is the most straightforward approach as Kotlin serialization handles the hierarchy automatically:

```kotlin
@Serializable
sealed interface Route

@Serializable
data object RouteA : Route

@Serializable
data class RouteB(val id: String) : Route

// Backstack with default serializer
val backStack: MutableList<Route> =
    rememberSerializable(serializer = SnapshotStateListSerializer()) {
        mutableStateListOf(RouteA)
    }
```

Alternatively, if you want to use the `rememberNavBackStack()` function explicitly, 
here's a slightly different configuration:

```kotlin
private val config = SavedStateConfiguration {
    serializersModule = SerializersModule {
        polymorphic(NavKey::class) {
            subclassesOfSealed<Route>()
        }
    }
}
val backStack = rememberNavBackStack(config, RouteA)
```

#### Multi-module with aggregated sealed types

For more complex projects with routes defined in multiple modules, you can define a sealed type for each module. 
Then, aggregate their serializers in the `app` module using the `subclassesOfSealed()` function.

```kotlin
// Module A
@Serializable sealed interface FeatureA : NavKey
@Serializable data object RouteA1 : FeatureA
@Serializable data object RouteA2 : FeatureA

// Module B
@Serializable sealed interface FeatureB : NavKey
@Serializable data class RouteB1(val id: String) : FeatureB
@Serializable data class RouteB2(val id: String) : FeatureB

// Module app
private val config = SavedStateConfiguration {
    serializersModule = SerializersModule {
        polymorphic(NavKey::class) {
            subclassesOfSealed<FeatureA>()
            subclassesOfSealed<FeatureB>()
        }
    }
}
val backStack = rememberNavBackStack(config, RouteA1)
```

With Dependency Injection (DI), you can also use DI containers to collect sealed-type serializers 
from each module into a `Set<KSerializer>` dynamically.

#### Multi-module with individual route registration

If your routes cannot be grouped into sealed types, 
you can manually combine `SerializersModule` instances from different modules.

```kotlin
// Module A
@Serializable data object RouteA1 : NavKey
@Serializable data object RouteA2 : NavKey

val serializerModuleA = SerializersModule {
    polymorphic(NavKey::class) {
        subclass(RouteA1::class, RouteA1.serializer())
        subclass(RouteA2::class, RouteA2.serializer())
    }
}

// Module B
@Serializable data class RouteB1(val id: String) : NavKey
@Serializable data class RouteB2(val id: String) : NavKey

val serializerModuleB = SerializersModule {
    polymorphic(NavKey::class) {
        subclass(RouteB1::class, RouteB1.serializer())
        subclass(RouteB2::class, RouteB2.serializer())
    }
}

// Module app
private val config = SavedStateConfiguration {
    serializersModule = serializerModuleA + serializerModuleB
}
val backStack = rememberNavBackStack(config, RouteA1)
```

This approach offers a high level of flexibility and decoupling, though it requires more manual maintenance.
Similar to the one sealed type per module approach, DI can also help assemble the list of serializers dynamically, 
which can improve flexibility.

## What's next

Navigation 3 is covered in-depth on the Android Developer portal.
While some of the documentation uses Android-specific examples,
the core concepts and navigation principles remain consistent across all platforms:

* [Overview of Navigation 3](https://developer.android.com/guide/navigation/navigation-3)
  with advice on managing state, modularizing navigation code, and animation.
* [Migration from Navigation 2 to Navigation 3](https://developer.android.com/guide/navigation/navigation-3/migration-guide).
  It's easier to see Navigation 3 as a new library than a new version of the existing library,
  so it's less of a migration and more of a rewrite.
  But the guide points out the general steps to take.
