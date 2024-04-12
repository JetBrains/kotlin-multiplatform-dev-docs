[//]: # (title: Lifecycle)

Lifecycle of components in Compose Multiplatform is adopted from the Jetpack Compose [Lifecycle](https://developer.android.com/topic/libraries/architecture/lifecycle)
concept. Lifecycle-aware components can react to changes in the lifecycle status of other components and help you
produce better-organized, and often lighter code, that is easier to maintain.

Things to consider in the implementation of the multiplatform version:
* The `LifecycleOwner` interface is implemented on the level of Compose Multiplatform itself, providing a `LifecycleOwner`
  entity for all common composables.
* The `ViewModel` class can be used on any platform, but it needs a factory for platforms where the necessary type cannot
  be discovered by class reflection (iOS and Web). There is no common `ViewModelStoreOwner` interface yet.

### Lifecycle implementation

Composables generally don't need unique lifecycles: there is usually a common `LifecycleOwner` which provides a lifecycle
for all interconnected entities. All composables created by Compose Multiplatform share the same lifecycle
by default: they can subscribe to its events, refer to the lifecycle state, and so on.

> The `LifecycleOwner` object is provided as a [CompositionLocal](https://developer.android.com/reference/kotlin/androidx/compose/runtime/CompositionLocal).
> If you would like to manage a lifecycle separately for a particular composable subtree, you [create your own](https://developer.android.com/topic/libraries/architecture/lifecycle#implementing-lco)
> `LifecycleOwner` implementation.
> 
{type="tip"}

For details on how lifecycle is used in navigation components, see [Navigation and routing](compose-navigation-routing.md).

### ViewModel implementation

The Android [ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel)
approach to building UI can now be implemented in common code, with a couple of restrictions:

* Current `ViewModel` implementation is considered [Experimental](supported-platforms.md#core-kotlin-multiplatform-technology-stability-levels).
  * For this reason, Compose Multiplatform does not yet implement a common `ViewModelStoreOwner` interface. Such an 
    implementation is planned for a future version of the library. You can, however, implement it yourself for your app.
  * `ViewModelStoreOwner` is, however, currently implemented in the scope needed for the experimental [navigation library](compose-navigation-routing.md).
* The `ViewModel` class works out of the box only for Android and desktop, where objects of the needed class can be created
  through class reflection. For iOS and web targets, you have to implement factories that create new `ViewModel` instances when
  needed.
