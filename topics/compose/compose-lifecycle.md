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

## Mapping Android lifecycle to other platforms

### iOS

| Native events and&nbsp;notifications | Lifecycle event | Lifecycle state change |
|--------------------------------------|-----------------|------------------------|
| `viewDidDisappear`                   | `ON_STOP`       | `STARTED` → `CREATED`  |
| `viewWillAppear`                     | `ON_START`      | `CREATED` → `STARTED`  |
| `didEnterBackground`                 | `ON_PAUSE`      | `RESUMED` → `STARTED`  |
| `willEnterForeground`                | `ON_RESUME`     | `STARTED` → `RESUMED`  |

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
| `windowGainedFocus`      | `ON_PAUSE`      | `RESUMED` → `STARTED`   |
| `windowLostFocus`        | `ON_RESUME`     | `STARTED` → `RESUMED`   |
| `dispose`                | `ON_DESTROY`    | `CREATED` → `DESTROYED` |


## Lifecycle implementation

> The lifecycle implementation is available in Compose Multiplatform starting with 1.6.10-beta01.
>
{type="note"}

Composables generally don't need unique lifecycles: a common `LifecycleOwner` usually provides a lifecycle
for all interconnected entities. By default, all composables created by Compose Multiplatform share the same lifecycle
: they can subscribe to its events, refer to the lifecycle state, and so on.

> The `LifecycleOwner` object is provided as a [CompositionLocal](https://developer.android.com/reference/kotlin/androidx/compose/runtime/CompositionLocal).
> If you would like to manage a lifecycle separately for a particular composable subtree, you can [create your own](https://developer.android.com/topic/libraries/architecture/lifecycle#implementing-lco)
> `LifecycleOwner` implementation.
>
{type="tip"}

For details on how lifecycle works in navigation components, see [Navigation and routing](compose-navigation-routing.md).

## ViewModel implementation

The Android [ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel)
approach to building UI can now be implemented in common code, with a couple of restrictions:

To use the multiplatform `ViewModel` implementation, add the following dependency to your `commonMain` source set:

```kotlin
kotlin {
    // ...
    sourceSets {
        // ...
        commonMain.dependencies {
            // ...
            implementation("org.jetbrains.androidx.lifecycle:lifecycle-viewmodel-compose:%composeViewmodelVersion%")
        }
        // ...
    }
}
```

Keep in mind the current limitations of the library:

* Current `ViewModel` implementation is considered [Experimental](supported-platforms.md#core-kotlin-multiplatform-technology-stability-levels).
  
  That's why the `ViewModelStoreOwner` interface is currently implemented only in the scope of the [navigation library](compose-navigation-routing.md).
  We plan to add a common implementation of the interface to Compose Multiplatform in future releases.
  However, you can implement it yourself for your specific project.

* The `ViewModel` class works out of the box only for Android and desktop, where objects of the needed class can be created
  through class reflection. For iOS and web targets, you have to implement factories that explicitly create
  new `ViewModel` instances.

