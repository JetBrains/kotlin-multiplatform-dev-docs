[//]: # (title: Common ViewModel)

The Android [ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel)
approach to building UI can be implemented in common code using Compose Multiplatform.

## Adding the common ViewModel to your project

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
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="org.jetbrains.androidx.lifecycle:lifecycle-viewmodel-compose:%composeViewmodelVersion%"}

## Using ViewModel in common code

Compose Multiplatform implements the common `ViewModelStoreOwner` interface, so in general using the `ViewModel` class
in common code is not much different from Android best practices.

Using the [navigation example](https://github.com/JetBrains/compose-multiplatform/tree/0e38f58b42d23ff6d0ad30b119d34fa1cd6ccedb/examples/nav_cupcake):

1. Declare the ViewModel class:

```kotlin
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewmodel.compose.viewModel

class OrderViewModel : ViewModel() {
   private val _uiState = MutableStateFlow(OrderUiState(pickupOptions = pickupOptions()))
   val uiState: StateFlow<OrderUiState> = _uiState.asStateFlow()
   // ...
}
```

2. Add the ViewModel to your composable function:

```kotlin
@Composable
fun CupcakeApp(
   viewModel: OrderViewModel = viewModel { OrderViewModel() },
) {
   // ...
}
```

> When running coroutines in a `ViewModel`, remember that the `ViewModel.viewModelScope` value is tied to the `Dispatchers.Main.immediate` value,
> which might be unavailable on desktop by default.
> To make ViewModel coroutines work correctly with Compose Multiplatform, add the `kotlinx-coroutines-swing` dependency to your project.
> See the [`Dispatchers.Main` documentation](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-main.html) for details.
> 
{style="tip"}

On non-JVM platforms, objects cannot be instantiated using type reflection.
So in common code you cannot call the `viewModel()` function without parameters: every time a `ViewModel` instance is created,
you need to provide at least an initializer as an argument.

If only an initializer is provided, the library creates a default factory under the hood.
But you can implement your own factories and call more explicit versions of the common `viewModel(...)` function,
just like [with Jetpack Compose](https://developer.android.com/topic/libraries/architecture/viewmodel#jetpack-compose).
