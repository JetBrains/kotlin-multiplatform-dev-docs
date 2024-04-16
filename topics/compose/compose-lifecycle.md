[//]: # (title: Lifecycle)

> The Lifecycle library is available in Compose Multiplatform starting with version 1.6.10-beta01.
>
{type="warning"}

Lifecycle of components in Compose Multiplatform is adopted from the Jetpack Compose [Lifecycle](https://developer.android.com/topic/libraries/architecture/lifecycle)
concept. Lifecycle-aware components can react to changes in the lifecycle status of other components and help you
produce better-organized, and often lighter code, that is easier to maintain.

Lifecycle functionality on Android originally was represented by [a set of callbacks](https://developer.android.com/guide/components/activities/activity-lifecycle).
Android Jetpack introduced [the dedicated library](https://developer.android.com/reference/kotlin/androidx/lifecycle/package-summary.html)
with the `Lifecycle` class to observe activity states.

Compose Multiplatform provides a common `LifecycleOwner` implementation,
which allows extending this functionality to other platforms and observing lifecycle states in common code

## States and events

The flow of lifecycle states and events
(diagram borrowed from the [Jetpack lifecycle documentation](https://developer.android.com/topic/libraries/architecture/lifecycle)):

![Lifecycle diagram](lifecycle-states.svg){width="700"}


## Mapping Android lifecycle to other platforms

### iOS

| Native event/notification | Lifecycle event | Lifecycle state change |
|---------------------------|-----------------|------------------------|
| `viewDidDisappear`        | `ON_STOP`       | `STARTED` → `CREATED`  |
| `viewWillAppear`          | `ON_START`      | `CREATED` → `STARTED`  |
| `didEnterBackground`      | `ON_PAUSE`      | `RESUMED` → `STARTED`  |
| `willEnterForeground`     | `ON_RESUME`     | `STARTED` → `RESUMED`  |

### Web

Due to limitations of the Wasm target, lifecycles skip the `CREATED` state as the application is always attached to the page,
and never reach the `DESTROYED` state as web pages are usually terminated only when the user closes the tab.

| Native event | Lifecycle event | Lifecycle state change |
|--------------|-----------------|------------------------|
| `blur`       | `ON_PAUSE`      | `RESUMED` → `STARTED`  |
| `focus`      | `ON_RESUME`     | `STARTED` → `RESUMED`  |

### Desktop

| Swing listener callbacks | Lifecycle event | Lifecycle state change  |
|--------------------------|-----------------|-------------------------|
| `windowIconified`        | `ON_STOP`       | `STARTED` → `CREATED`   |
| `windowDeiconified`      | `ON_START`      | `CREATED` → `STARTED`   |
| `windowGainedFocus`      | `ON_PAUSE`      | `RESUMED` → `STARTED`   |
| `windowLostFocus`        | `ON_RESUME`     | `STARTED` → `RESUMED`   |
| `dispose`                | `ON_DESTROY`    | `CREATED` → `DESTROYED` |


## Lifecycle implementation

Composables generally don't need unique lifecycles: there is usually a common `LifecycleOwner` which provides a lifecycle
for all interconnected entities. All composables created by Compose Multiplatform share the same lifecycle
by default: they can subscribe to its events, refer to the lifecycle state, and so on.

> The `LifecycleOwner` object is provided as a [CompositionLocal](https://developer.android.com/reference/kotlin/androidx/compose/runtime/CompositionLocal).
> If you would like to manage a lifecycle separately for a particular composable subtree, you [create your own](https://developer.android.com/topic/libraries/architecture/lifecycle#implementing-lco)
> `LifecycleOwner` implementation.
> 
{type="tip"}

For details on how lifecycle is used in navigation components, see [Navigation and routing](compose-navigation-routing.md).

## ViewModel implementation

The Android [ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel)
approach to building UI can now be implemented in common code, with a couple of restrictions:

* Current `ViewModel` implementation is considered [Experimental](supported-platforms.md#core-kotlin-multiplatform-technology-stability-levels).
  * For this reason, Compose Multiplatform does not yet implement a common `ViewModelStoreOwner` interface. Such an 
    implementation is planned for a future version of the library. You can, however, implement it yourself for your app.
  * `ViewModelStoreOwner` is, however, currently implemented in the scope needed for the experimental [navigation library](compose-navigation-routing.md).
* The `ViewModel` class works out of the box only for Android and desktop, where objects of the needed class can be created
  through class reflection. For iOS and web targets, you have to implement factories that create new `ViewModel` instances when
  needed.
