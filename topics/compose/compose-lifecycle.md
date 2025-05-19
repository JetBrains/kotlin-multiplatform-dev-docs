[//]: # (title: Lifecycle)

Lifecycle of components in Compose Multiplatform is adopted from the Jetpack Compose [lifecycle](https://developer.android.com/topic/libraries/architecture/lifecycle)
concept.
Lifecycle-aware components can react to changes in the lifecycle state of other components and help you
produce better-organized, and often lighter, code that is easier to maintain.

Compose Multiplatform provides a common `LifecycleOwner` implementation, which extends the original Jetpack Compose
functionality to other platforms and helps observe lifecycle states in common code.

## States and events

The flow of lifecycle states and events
(same as for the [Jetpack lifecycle](https://developer.android.com/topic/libraries/architecture/lifecycle)):

![Lifecycle diagram](lifecycle-states.svg){width="700"}

## Lifecycle implementation

Composables usually don't need unique lifecycles: a common `LifecycleOwner` provides a lifecycle
for all interconnected entities. By default, all composables created by Compose Multiplatform share the same lifecycle –
they can subscribe to its events, refer to the lifecycle state, and so on.

> The `LifecycleOwner` object is provided as a [CompositionLocal](https://developer.android.com/reference/kotlin/androidx/compose/runtime/CompositionLocal).
> If you would like to manage a lifecycle separately for a particular composable subtree, you can [create your own](https://developer.android.com/topic/libraries/architecture/lifecycle#implementing-lco)
> `LifecycleOwner` implementation.
>
{style="tip"}

When working with coroutines in multiplatform lifecycles,
remember that the `Lifecycle.coroutineScope` value is tied to the `Dispatchers.Main.immediate` value,
which might be unavailable on desktop targets by default.
To make coroutines and flows in lifecycles work correctly with Compose Multiplatform, add the `kotlinx-coroutines-swing` dependency to your project.
See [`Dispatchers.Main` documentation](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-main.html) for details.

* Learn how the lifecycle works in navigation components in [](compose-navigation-routing.md).
* Learn more about the multiplatform ViewModel implementation on the [](compose-viewmodel.md) page.

## Mapping Android lifecycle to other platforms

### iOS

| Native events and&nbsp;notifications    | Lifecycle event | Lifecycle state change  |
|-----------------------------------------|-----------------|-------------------------|
| `viewDidDisappear`                      | `ON_STOP`       | `STARTED` → `CREATED`   |
| `viewWillAppear`                        | `ON_START`      | `CREATED` → `STARTED`   |
| `willResignActive`                      | `ON_PAUSE`      | `RESUMED` → `STARTED`   |
| `didBecomeActive`                       | `ON_RESUME`     | `STARTED` → `RESUMED`   |
| `didEnterBackground`                    | `ON_STOP`       | `STARTED` → `CREATED`   |
| `willEnterForeground`                   | `ON_START`      | `CREATED` → `STARTED`   |
| `viewControllerDidLeaveWindowHierarchy` | `ON_DESTROY`    | `CREATED` → `DESTROYED` |

### Web

Due to limitations of the Wasm target, lifecycles:

* Skip the `CREATED` state, as the application is always attached to the page.
* Never reach the `DESTROYED` state, as web pages are usually terminated only when the user closes the tab.

| Native event | Lifecycle event | Lifecycle state change |
|--------------|-----------------|------------------------|
| `blur`       | `ON_PAUSE`      | `RESUMED` → `STARTED`  |
| `focus`      | `ON_RESUME`     | `STARTED` → `RESUMED`  |

### Desktop

| Swing listener callbacks | Lifecycle event | Lifecycle state change  |
|--------------------------|-----------------|-------------------------|
| `windowIconified`        | `ON_STOP`       | `STARTED` → `CREATED`   |
| `windowDeiconified`      | `ON_START`      | `CREATED` → `STARTED`   |
| `windowLostFocus`        | `ON_PAUSE`      | `RESUMED` → `STARTED`   |
| `windowGainedFocus`      | `ON_RESUME`     | `STARTED` → `RESUMED`   |
| `dispose`                | `ON_DESTROY`    | `CREATED` → `DESTROYED` |
