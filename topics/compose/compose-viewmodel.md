[//]: # (title: Multiplatform ViewModel)

The Android [ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel) connects the business logic of your app with the UI components.
With Compose Multiplatform, you can also use ViewModels in common code.

## Set up dependencies

To share the ViewModel implementation and UI across platforms:

1. Define the dependencies in the `gradle/libs.versions.toml` file:

    ```toml
    [versions]
    androidx-viewmodel = "2.10.0"
    
    [libraries]
    androidx-lifecycle-viewmodel-compose = { module = "org.jetbrains.androidx.lifecycle:lifecycle-viewmodel-compose", version.ref = "androidx-viewmodel" }
    androidx-lifecycle-viewmodel-navigation3 = { module = "androidx.lifecycle:lifecycle-viewmodel-navigation3", version.ref = "androidx-viewmodel" }
    ```

2. In the `build.gradle.kts` file, add the following dependencies to the `commonMain` source set:

    ```kotlin
    kotlin {
        // ...
        sourceSets {
            // ...
            commonMain.dependencies {
                implementation(libs.androidx.lifecycle.viewmodel.compose)
                implementation(libs.androidx.lifecycle.viewmodel.navigation3)
            }
            // ...
        }
    }
    ```
   {initial-collapse-state="collapsed" collapsible="true" collapsed-title="implementation(libs.androidx.lifecycle.viewmodel.compose)"}

3. If you have a desktop target, add the `kotlinx-coroutines-swing` dependency.

   In the `libs.versions.toml` file:

    ```toml
    [versions]
    kotlinx-coroutines = "1.10.2"
    
    [libraries]
    kotlinx-coroutines-swing = { module = "org.jetbrains.kotlinx:kotlinx-coroutines-swing", version.ref = "kotlinx-coroutines" }
    ```

   In the `build.gradle.kts` file:

    ```kotlin
    kotlin {
        // ...
        sourceSets {
            // ...
            jvmMain.dependencies {
                implementation("org.jetbrains.kotlinx:kotlinx-coroutines-swing:kotlinx-coroutines")
            }
            // ...
        }
    }
    ```
    {initial-collapse-state="collapsed" collapsible="true" collapsed-title="implementation('org.jetbrains.kotlinx:kotlinx-coroutines-swing:kotlinx-coroutines')"}

    When running coroutines in a `ViewModel`, `ViewModel.viewModelScope` is tied
    to `Dispatchers.Main.immediate`, which might be unavailable on desktop by default.
    The Kotlinx Coroutines Swing library makes ViewModel coroutines work correctly with Compose Multiplatform.
    See the [`Dispatchers.Main` documentation](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-dispatchers/-main.html) for details.

If your project has a different level of code sharing, refer to the [](#levels-of-code-sharing) section.
    
## Using ViewModel in common code

Compose Multiplatform provides a common `ViewModelStoreOwner` implementation, so using the `ViewModel` class
in common code is generally not much different from Android best practices.

On non-JVM platforms, type reflection for instantiating objects is not available, so
you cannot call the `viewModel()` function without parameters in common code.
Every time you create a `ViewModel` instance, you need to provide at least an initializer as an argument.

If only an initializer is provided, Compose Multiplatform creates a default factory under the hood.
However, you can implement your own factories and call more explicit versions of the common `viewModel(...)` function,
just like [with Jetpack Compose](https://developer.android.com/topic/libraries/architecture/viewmodel#jetpack-compose).

1. Create a ViewModel class named `OrderViewModel`, based on the [navigation sample](https://github.com/JetBrains/compose-multiplatform/tree/0e38f58b42d23ff6d0ad30b119d34fa1cd6ccedb/examples/nav_cupcake):

    ```kotlin
    data class OrderUiState(val quantity: Int = 0, val price: String = "$0.00")
    
    class OrderViewModel : ViewModel() {
        private val _uiState = MutableStateFlow(OrderUiState())
        val uiState = _uiState.asStateFlow()
    
        fun setQuantity(n: Int) {
            _uiState.update { it.copy(quantity = n, price = "$${n * 2}.00") } 
        }
        // ...
    }
    ```

2. Add the ViewModel to your composable function:

    ```kotlin
    import com.example.ui.OrderViewModel
    
    @Composable
    fun CupcakeApp(
        viewModel: OrderViewModel = viewModel { OrderViewModel() },
    ) {
        // ...
    }
    ```

## ViewModel scoping with Navigation 3

When using ViewModels with Navigation 3 in a multiplatform project,
ViewModels are not automatically scoped to navigation entries by default. 
Without explicit scoping, the ViewModel will be tied to `Activity` rather than the screen, 
even after the user has navigated away.

To scope ViewModels and saveable Compose state per navigation entry, 
pass the Navigation 3 entry decorators to `NavDisplay` when defining the navigation destinations:

```kotlin
import androidx.lifecycle.viewmodel.navigation3.rememberViewModelStoreNavEntryDecorator
import androidx.navigation3.runtime.rememberSaveableStateHolderNavEntryDecorator

//...

NavDisplay(
    entryDecorators = listOf(
        // Saves Compose state per entry
        rememberSaveableStateHolderNavEntryDecorator(),
        // Scopes ViewModel per entry
        rememberViewModelStoreNavEntryDecorator()
    ),
    backStack = backStack,
    entryProvider = entryProvider { }
)
```

## ViewModel and Dependency Injection

A dependency injection (DI) framework allows you to inject different dependencies into components 
based on the current environment or target platform.
To manage ViewModels, you can use Koin, Metro, or another DI framework that supports Kotlin Multiplatform.

For an advanced example of dependency injection usage, 
see the [Share data access layer](multiplatform-ktor-sqldelight.md) tutorial.

### Koin

Koin is a runtime DI framework that uses a DSL to declare modules. 
To use Koin with Compose ViewModels, add the following packages:

* `koin-core-viewmodel`
* `koin-compose-viewmodel`

You can then inject a ViewModel into a Composable function using `koinViewModel()`:

```kotlin
@Composable
fun CupcakeApp(
    viewModel: UserViewModel = koinViewModel()
) {
    val state by viewModel.state.collectAsState()
    // ...
}
```

For implementation details, see the [Koin ViewModel documentation](https://insert-koin.io/docs/reference/koin-compose/compose-viewmodel).

### Metro

Metro is a compile-time DI framework implemented as a Kotlin compiler plugin.
To use Metro with Compose ViewModels, add the following packages:

* `metrox-viewmodel`
* `metrox-viewmodel-compose`

Then you can inject a ViewModel into a Composable function using `metroViewModel()`:

```kotlin
@Composable
fun CupcakeApp(viewModel: HomeViewModel = metroViewModel()) {
    // ...
}
```

For implementation details, see the [MetroX ViewModel Compose documentation](https://zacsweers.github.io/metro/latest/metrox-viewmodel-compose/).

## Levels of code sharing

You can choose which parts of your code to share and which to keep platform-specific:

* To share both UI and business logic between platforms,
  see the [shared logic and UI tutorial](compose-multiplatform-create-first-app.md).
* To share some code without sharing UI implementation,
  see the [shared logic tutorial](multiplatform-create-first-app.md).

The following examples show how to use ViewModel at different levels of code sharing. 
All examples are based on the `OrderViewModel` class introduced above.

### Shared ViewModel and UI

In this approach, everything, including the `ViewModel` and the UI, is shared via Compose Multiplatform. 
You can write your app's UI once, and it will work on all platforms.

```kotlin
@Composable
fun CupcakeApp(
    viewModel: OrderViewModel = viewModel { OrderViewModel() }
) {
    val uiState by viewModel.uiState.collectAsState()

    Column(modifier = Modifier.padding(16.dp)) {
        Text("Quantity: ${uiState.quantity}")
        Text("Price: ${uiState.price}")

        Button(onClick = { viewModel.setQuantity(6) }) {
            Text("Set Quantity to '6'")
        }
    }
}
```

### Shared ViewModel, platform-specific UI

In this approach, the `ViewModel` (business logic) is shared, but platforms have native UI implementations.
Learn more in [Set up ViewModel for Kotlin Multiplatform](https://developer.android.com/kotlin/multiplatform/viewmodel).

1. Since the UI is not shared in this case, you can switch from the Compose Multiplatform version of the ViewModel 
   library to the `androidx.lifecycle` library.
   Update the dependencies in the `libs.versions.toml` file:

    ```toml
    [versions]
    androidx-viewmodel = "2.10.0"
    
    [libraries]
    androidx-lifecycle-viewmodel = { module = "androidx.lifecycle:lifecycle-viewmodel", version.ref = "androidx-viewmodel" }
    ```
    
    In the `build.gradle.kts` file, declare the dependency as `api`, as it will be exported to the binary framework:

    ```kotlin
    kotlin {
        // ...
        sourceSets {
            // ...
            commonMain.dependencies {
                api(libs.androidx.lifecycle.viewmodel)
            }
            // ...
        }
    }
    ```
   {initial-collapse-state="collapsed" collapsible="true" collapsed-title="api(libs.androidx.lifecycle.viewmodel)"}

2. On Android, Jetpack Compose automatically finds the `ViewModelStoreOwner` provided by the Activity and supplies the `OrderViewModel`:

    ```kotlin
    @Composable
    fun AndroidCupcakeApp(
        viewModel: OrderViewModel = viewModel { OrderViewModel() }
    ) {
        val uiState by viewModel.uiState.collectAsState()
    
        Column {
            Text("Quantity: ${uiState.quantity}")
            Text("Price: ${uiState.price}")
            Button(onClick = { viewModel.setQuantity(6) }) {
            Text("Set Quantity to 6")
            }
        }
    }
    ```

3. On iOS, the SwiftUI `App` struct uses a `StateObject` (`OrderObserver`), which manually manages an `IosViewModelOwner` to ensure the ViewModel lifecycle is respected.

   1. Export ViewModel APIs for access from Swift:
   
      ```kotlin
      listOf(
         iosArm64(),
         iosSimulatorArm64(),
      ).forEach {
         it.binaries.framework {
            export(libs.androidx.lifecycle.viewmodel)
            baseName = "shared"
         }
      }
      ```
   
   2. There is no built-in `ViewModelStoreOwner` on iOS. We recommend using [KMP-ObservableViewModel](https://github.com/rickclephas/KMP-ObservableViewModel), 
   which lets SwiftUI observe Kotlin Multiplatform ViewModels directly and also handles the required ViewModel lifecycle/store-owner boilerplate for iOS.

   First, define your ViewModel in shared Kotlin code using KMP-ObservableViewModel APIs:

    ```kotlin    
    // commonMain
    import com.rickclephas.kmp.observableviewmodel.ViewModel
    import com.rickclephas.kmp.nativecoroutines.NativeCoroutinesState
    import kotlinx.coroutines.flow.MutableStateFlow
    import kotlinx.coroutines.flow.StateFlow
    import kotlinx.coroutines.flow.asStateFlow
    
    class OrderViewModel : ViewModel() {
        private val _uiState = MutableStateFlow(OrderUiState())
    
        @NativeCoroutinesState
        val uiState: StateFlow<OrderUiState> = _uiState.asStateFlow()
    
        fun setQuantity(n: Int) {
            _uiState.value = _uiState.value.copy(quantity = n)
        }
    }
    ```
    
    Then you can use the ViewModel in the iOS UI entry point:

    ```swift
    import SwiftUI
    import shared
    import KMPObservableViewModelSwiftUI
    
    @main
    struct iOSCupcakeApp: App {
        var body: some Scene {
            WindowGroup {
                CupcakeView()
            }
        }
    }
    
    struct CupcakeView: View {
        @StateViewModel private var viewModel = OrderViewModel()
    
        var body: some View {
            VStack {
                Text("Quantity: \(viewModel.uiState.quantity)")
                Text("Price: \(viewModel.uiState.price)")
    
                Button("Set Quantity to '6'") {
                    viewModel.setQuantity(n: 6)
                }
            }
        }
    }
    ```

### Shared repo/data layer, platform-specific ViewModels and UI

You can also share only the data and repository layer, while using platform-specific ViewModel implementations. 
This allows you to use native patterns on each platform, such as Hilt for Android dependency injection, 
or `ObservableObject` with Combine for iOS.

1. Create a shared repository class with the data logic:

    ```kotlin
    class OrderRepository {
        fun calculatePrice(quantity: Int) = "$${quantity * 2}.00"
    }
    ```

2. Implement platform-specific ViewModels.

   * On Android, use the standard Android ViewModel and inject the repository:

      ```kotlin
      class AndroidOrderViewModel(private val repo: OrderRepository) : ViewModel() {
          private val _uiState = MutableStateFlow(OrderUiState())
          val uiState = _uiState.asStateFlow()

          fun setQuantity(n: Int) { 
              _uiState.update { it.copy(quantity = n, price = repo.calculatePrice(n)) }
          }
          // ...
      }
      ```

   * On iOS, implement the ViewModel natively in Swift using `ObservableObject`:

    ```swift
    import shared
    
    class IOSOrderViewModel: ObservableObject {
        private let repo: OrderRepository
        @Published var uiState: OrderUiState = OrderUiState()
    
        init(repo: OrderRepository) {
            self.repo = repo
        }
    
        func setQuantity(n: Int32) {
            uiState = OrderUiState(quantity: n, price: repo.calculatePrice(quantity: n))
        }
    }
    ```

3. Implement platform-specific UI.
    
    * On Android:

    ```kotlin
    @Composable
    fun AndroidCupcakeApp(
        viewModel: AndroidOrderViewModel = viewModel { AndroidOrderViewModel(OrderRepository()) }
    ) {
        val uiState by viewModel.uiState.collectAsState()
    
        Column {
            Text("Quantity: ${uiState.quantity}")
            Text("Price: ${uiState.price}")
            Button(onClick = { viewModel.setQuantity(6) }) {
                Text("Set Quantity to '6'")
            }
        }
    }
    ```
   
   * On iOS:
    
    ```swift
    struct IOSCupcakeApp: App {
        @StateObject var viewModel = IOSOrderViewModel(repo: OrderRepository())
    
        var body: some View {
            VStack {
                Text("Quantity: \(viewModel.uiState.quantity)")
                Text("Price: \(viewModel.uiState.price)")
                Button("Set Quantity to '6'") {
                    viewModel.setQuantity(n: 6)
                }
            }
        }
    }
    ```

## What's next

* Check out the [full navigation sample](https://github.com/JetBrains/compose-multiplatform/tree/master/examples/nav_cupcake).
* See [Set up ViewModel for Kotlin Multiplatform](https://developer.android.com/kotlin/multiplatform/viewmodel) for additional Android-focused guidance.
* Learn how to [integrate Compose Multiplatform with SwiftUI](compose-swiftui-integration.md) when using shared ViewModels with native UI.