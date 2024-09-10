[//]: # (title: Share more logic between iOS and Android)

<microformat>
    <p>This is the fourth part of the <strong>Create a Kotlin Multiplatform app with shared logic and native UI</strong> tutorial. Before proceeding, make sure you've completed previous steps.</p>
    <p><img src="icon-1-done.svg" width="20" alt="First step"/> <a href="multiplatform-create-first-app.md">Create your first cross-platform app</a><br/>
      <img src="icon-2-done.svg" width="20" alt="Second step"/> <a href="multiplatform-update-ui.md">Update the user interface</a><br/>
      <img src="icon-3-done.svg" width="20" alt="Third step"/> <a href="multiplatform-dependencies.md">Add dependencies</a><br/>
      <img src="icon-4.svg" width="20" alt="Fourth step"/> <strong>Share more logic</strong><br/>
      <img src="icon-5-todo.svg" width="20" alt="Fifth step"/> Wrap up your project<br/>
    </p>
</microformat>

Now that you've implemented common logic using external dependencies, you can start adding more complex logic. Network
requests and data serialization are the [most popular use cases](https://kotlinlang.org/lp/multiplatform/) for sharing code using Kotlin
Multiplatform. Learn how to implement these in your first application, so that after completing this onboarding journey
you can use them in future projects.

The updated app will retrieve data over the internet from the [SpaceX API](https://github.com/r-spacex/SpaceX-API/tree/master/docs#rspacex-api-docs)
and display the date of the last successful launch of a SpaceX rocket.

## Add more dependencies

You'll need to add the following multiplatform libraries in your project:

* [`kotlinx.coroutines`](https://github.com/Kotlin/kotlinx.coroutines), to use coroutines for asynchronous code,
  which allows simultaneous operations.
* [`kotlinx.serialization`](https://github.com/Kotlin/kotlinx.serialization), to deserialize JSON responses into objects of entity classes used to process
  network operations.
* [Ktor](https://ktor.io/), a framework to create an HTTP client for retrieving data over the internet.

### kotlinx.coroutines

To add `kotlinx.coroutines` to your project, specify a dependency in the common source set. To do so, add the following
line to the `build.gradle.kts` file of the shared module:

```kotlin
kotlin {
    // ... 
    sourceSets {
        commonMain.dependencies {
           // ...
           implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:%coroutinesVersion%")
        }
    }
}
```

The Multiplatform Gradle plugin automatically adds a dependency to the platform-specific (iOS and Android) parts
of `kotlinx.coroutines`.

### kotlinx.serialization

To use the `kotlinx.serialization` library, set up a corresponding Gradle plugin.
To do that, add the following line to the existing `plugins {}` block at the very beginning of the `build.gradle.kts` file
in the shared module:

```kotlin
plugins {
    // ...
    kotlin("plugin.serialization") version "%kotlinVersion%"
}
```

### Ktor

You need to add the core dependency (`ktor-client-core`) to the common source set of the shared module.
You also need to add supporting dependencies:

* Add the `ContentNegotiation` functionality (`ktor-client-content-negotiation`), which allows serializing and deserializing
  the content in a specific format.
* Add the `ktor-serialization-kotlinx-json` dependency to instruct Ktor to use the JSON format and `kotlinx.serialization`
  as a serialization library. Ktor will expect JSON data and deserialize it into a data class when receiving responses.
* Provide the platform engines by adding dependencies on the corresponding artifacts in the platform source sets
  (`ktor-client-android`, `ktor-client-darwin`).

```kotlin
kotlin {
    // ...
    val ktorVersion = "%ktorVersion%"

    sourceSets {
        commonMain.dependencies {
            // ...

            implementation("io.ktor:ktor-client-core:$ktorVersion")
            implementation("io.ktor:ktor-client-content-negotiation:$ktorVersion")
            implementation("io.ktor:ktor-serialization-kotlinx-json:$ktorVersion")
        }
        androidMain.dependencies {
            implementation("io.ktor:ktor-client-android:$ktorVersion")
        }
        iosMain.dependencies {
            implementation("io.ktor:ktor-client-darwin:$ktorVersion")
        }
    }
}
```

Synchronize the Gradle files by clicking **Sync Now** in the notification.

## Create API requests

You'll need the [SpaceX API](https://github.com/r-spacex/SpaceX-API/tree/master/docs#rspacex-api-docs) to retrieve data, and you'll use a single method to
get the list of all launches from the **v4/launches** endpoint.

### Add a data model

In `shared/src/commonMain/kotlin`, create a new `RocketLaunch.kt` file and add a data class which stores data from the SpaceX API:

```kotlin
import kotlinx.serialization.SerialName
import kotlinx.serialization.Serializable

@Serializable
data class RocketLaunch (
    @SerialName("flight_number")
    val flightNumber: Int,
    @SerialName("name")
    val missionName: String,
    @SerialName("date_utc")
    val launchDateUTC: String,
    @SerialName("success")
    val launchSuccess: Boolean?,
)
```

* The `RocketLaunch` class is marked with the `@Serializable` annotation, so that the `kotlinx.serialization` plugin can
  automatically generate a default serializer for it.
* The `@SerialName` annotation allows you to redefine field names, making it possible to declare properties in data classes
  with more readable names.

### Connect HTTP client

1. In `shared/src/commonMain/kotlin`, create a new `RocketComponent` class.
2. Add the `httpClient` property to retrieve rocket launch information through an HTTP GET request:

    ```kotlin
    import io.ktor.client.*
    import io.ktor.client.plugins.contentnegotiation.*
    import io.ktor.serialization.kotlinx.json.*
    import kotlinx.serialization.json.Json
    
    class RocketComponent {
        private val httpClient = HttpClient {
            install(ContentNegotiation) {
                json(Json {
                    prettyPrint = true
                    isLenient = true
                    ignoreUnknownKeys = true
                })
            }
        }
    }
    ```

   * The [ContentNegotiation Ktor plugin](https://ktor.io/docs/serialization-client.html#register_json) and the JSON serializer deserialize the result of the GET request.
   * The JSON serializer here is configured in a way that it prints JSON in a more readable manner with the `prettyPrint` property. It
     is more flexible when reading malformed JSON with `isLenient`,
     and it ignores keys that haven't been declared in the rocket launch model with `ignoreUnknownKeys`.

3. Add the `getDateOfLastSuccessfulLaunch()` suspending function to `RocketComponent`:

   ```kotlin
   private suspend fun getDateOfLastSuccessfulLaunch(): String {
       // ...
   }
   ```

4. Call the `httpClient.get()` function to retrieve information about rocket launches:

   ```kotlin
   import io.ktor.client.request.*
   import io.ktor.client.call.*

   private suspend fun getDateOfLastSuccessfulLaunch(): String {
       val rockets: List<RocketLaunch> = httpClient.get("https://api.spacexdata.com/v4/launches").body()
   }
   ```

   * `httpClient.get()` is also a suspending function
     because it needs to retrieve data over the network asynchronously without blocking threads.
   * Suspending functions can only be called from coroutines or other suspending functions. This is why `getDateOfLastSuccessfulLaunch()`
     was marked with the `suspend` keyword. The network request is executed in the HTTP client's thread pool.

5. Update the function again to find the last successful launch in the list:

   ```kotlin
   private suspend fun getDateOfLastSuccessfulLaunch(): String {
       val rockets: List<RocketLaunch> = httpClient.get("https://api.spacexdata.com/v4/launches").body()
       val lastSuccessLaunch = rockets.last { it.launchSuccess == true }
   }
   ```

   The list of rocket launches is sorted by date from oldest to newest.

6. Convert the launch date from UTC to your local date and format the output:

   ```kotlin
   import kotlinx.datetime.Instant
   import kotlinx.datetime.TimeZone
   import kotlinx.datetime.toLocalDateTime

   private suspend fun getDateOfLastSuccessfulLaunch(): String {
       val rockets: List<RocketLaunch> =
           httpClient.get("https://api.spacexdata.com/v4/launches").body()
       val lastSuccessLaunch = rockets.last { it.launchSuccess == true }
       val date = Instant.parse(lastSuccessLaunch.launchDateUTC)
           .toLocalDateTime(TimeZone.currentSystemDefault())
       
       return "${date.month} ${date.dayOfMonth}, ${date.year}"
   }
   ```

   The date will be in the "MMMM DD, YYYY" format, for example, OCTOBER 5, 2022.

7. Add another suspending function, `launchPhrase()`, which will create a message using the `getDateOfLastSuccessfulLaunch()`
   function:

   ```kotlin
   suspend fun launchPhrase(): String =
       try {
           "The last successful launch was on ${getDateOfLastSuccessfulLaunch()} 🚀"
       } catch (e: Exception) {
           println("Exception during getting the date of the last successful launch $e")
           "Error occurred"
       }
   ```
### Create the flow

You can use flows instead of suspending functions. They emit a sequence of values instead of a single value that
suspending functions return.

1. Open the `Greeting.kt` file in the `shared/src/commonMain/kotlin` directory.
2. Add a `rocketComponent` property to the `Greeting` class. The property will store the message with the last successful launch date:

   ```kotlin
   private val rocketComponent = RocketComponent()
   ```

3. Change the `greet()` function to return a `Flow`:

   ```kotlin
   import kotlinx.coroutines.delay
   import kotlinx.coroutines.flow.Flow
   import kotlinx.coroutines.flow.flow
   import kotlin.time.Duration.Companion.seconds

   fun greet(): Flow<String> = flow {
        emit(if (Random.nextBoolean()) "Hi!" else "Hello!")
        delay(1.seconds)
        emit("Guess what this is! > ${platform.name.reversed()}")
        delay(1.seconds)
        emit(daysPhrase())
        emit(rocketComponent.launchPhrase())
   }
   ```

   * The `Flow` is created here with the `flow()` builder function, which wraps all the statements.
   * The `Flow` emits strings with a delay of one second between each emission. The last element is only emitted after
     the network response returns, so the exact delay depends on your network.

### Add internet access permission

To access the internet, the Android application needs the appropriate permission. Since all network requests are made from the
shared module, it makes sense to add the internet access permission to its manifest.

Update your `composeApp/src/androidMain/AndroidManifest.xml` file with the access permission:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <uses-permission android:name="android.permission.INTERNET"/>
    ...
</manifest>
```

## Update native Android and iOS UI

You've already updated the API of the shared module by changing the return type of the `greet()` function to `Flow`.
Now you need to update native (iOS, Android) parts of the project so that they can properly handle the result of calling
the `greet()` function.

### Android app

As both the shared module and the Android application are written in Kotlin, using shared code from Android is straightforward.

#### Introduce a view model

Now that the application is becoming more complex, it's time to introduce a view model to the [Android activity](https://developer.android.com/guide/components/activities/intro-activities)
called `MainActivity`. It invokes the `App()` function that implements the UI.
The view model will manage the data from the activity and won't disappear when the activity undergoes a lifecycle change.

1. Add the following dependencies to your `composeApp/build.gradle.kts` file:

    ```kotlin
    androidMain.dependencies {
        // ...
        implementation("androidx.lifecycle:lifecycle-viewmodel-ktx:2.6.2")
        implementation("androidx.lifecycle:lifecycle-runtime-compose:2.6.2")
        implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.6.2")
    }
    ```

2. In `composeApp/src/androidMain/kotlin`, create a new `MainViewModel` Kotlin class:

    ```kotlin
    import androidx.lifecycle.ViewModel
    
    class MainViewModel : ViewModel() {
        // ...
    }
    ```

   This class extends Android's `ViewModel` class, which ensures the correct behavior regarding lifecycle and configuration changes.

3. Create a `greetingList` value of the [StateFlow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/)
   type and its backing property:

    ```kotlin
    import kotlinx.coroutines.flow.MutableStateFlow
    import kotlinx.coroutines.flow.StateFlow
    
    class MainViewModel : ViewModel() {
        private val _greetingList = MutableStateFlow<List<String>>(listOf())
        val greetingList: StateFlow<List<String>> get() = _greetingList
    }
    ```

   * `StateFlow` here extends the `Flow` interface but has a single value or state.
   * The private backing property `_greetingList` ensures that only clients of this class can access the read-only `greetingList` property.

4. In the `init` function of the View Model, collect all the strings from the `Greeting().greet()` flow:

    ```kotlin
   import androidx.lifecycle.viewModelScope
   import kotlinx.coroutines.launch
   
   class MainViewModel : ViewModel() {
       private val _greetingList = MutableStateFlow<List<String>>(listOf())
       val greetingList: StateFlow<List<String>> get() = _greetingList
       
       init {
           viewModelScope.launch {
               Greeting().greet().collect { phrase ->
                    //...
               }
           }
       }
    }
    ```

   Since the `collect()` function is suspended, the `launch` coroutine is used within the view model's scope.
   This means that the launch coroutine will run only during the correct phases of the view model's lifecycle.

5. Inside the `collect` trailing lambda, update the value of `_greetingList` to append the collected `phrase` to the list of phrases in `list`:

    ```kotlin
    import kotlinx.coroutines.flow.update
   
    class MainViewModel : ViewModel() {
        //...
   
        init {
            viewModelScope.launch {
                Greeting().greet().collect { phrase ->
                    _greetingList.update { list -> list + phrase }
                }
            }
        }
    }
    ```

   The `update()` function will update the value automatically.

#### Use the view model's flow

1. In `composeApp/src/androidMain/kotlin`, locate the `App.kt` file and update it, replacing the previous implementation:

    ```kotlin
    import androidx.lifecycle.compose.collectAsStateWithLifecycle
    import androidx.compose.runtime.getValue
    import androidx.lifecycle.viewmodel.compose.viewModel
    
    @Composable
    fun App(mainViewModel: MainViewModel = viewModel()) {
        MaterialTheme {
            val greetings by mainViewModel.greetingList.collectAsStateWithLifecycle()
    
            Column(
                modifier = Modifier.padding(all = 20.dp),
                verticalArrangement = Arrangement.spacedBy(8.dp),
            ) {
                greetings.forEach { greeting ->
                    Text(greeting)
                    Divider()
                }
            }
        }
    }
    ```

   * The `collectAsStateWithLifecycle()` function calls on `greetingList` to collect the value from the view model's flow
     and represent it as a composable state in a lifecycle-aware manner.
   * When a new flow is created, the compose state will change and display a scrollable `Column` with greeting phrases
     arranged vertically and separated by dividers.

2. To see the results, re-run your **composeApp** configuration in Android Studio:

   ![Final results](multiplatform-mobile-upgrade-android.png){width=300}

### iOS app

For the iOS part of the project, you'll make use of the [Model–view–viewmodel](https://en.wikipedia.org/wiki/Model–view–viewmodel)
pattern again to connect the UI to the shared module, which contains all the business logic.

The module is already imported in the `ContentView.swift` file with the `import Shared` declaration.

> If you see errors in Xcode regarding the shared module or when updating your code, run the **iosApp** configuration
> from Android Studio.
>
{type="tip"}

#### Introducing a view model

1. Go back to your iOS app in Xcode.
2. In `iosApp/iOSApp.swift`, update the entry point for your app:

   ```swift
   @main
   struct iOSApp: App {
       var body: some Scene {
           WindowGroup {
               ContentView(viewModel: ContentView.ViewModel())
           }
       }
   }
   ```

3. In `iosApp/ContentView.swift`, create a `ViewModel` class for `ContentView`, which will prepare and manage data for it.
   Call the `startObserving()` function within a `task()` call to support concurrency:

    ```Swift
    import SwiftUI
    import Shared
    
    struct ContentView: View {
        @ObservedObject private(set) var viewModel: ViewModel
    
        var body: some View {
            ListView(phrases: viewModel.greetings)
                .task { await self.viewModel.startObserving() }
        }
    }
    
    extension ContentView {
        @MainActor
        class ViewModel: ObservableObject {
            @Published var greetings: Array<String> = []
            
            func startObserving() {
                // ...
            }
        }
    }
    
    struct ListView: View {
        let phrases: Array<String>
    
        var body: some View {
            List(phrases, id: \.self) {
                Text($0)
            }
        }
    }
    ```

   * `ViewModel` is declared as an extension to `ContentView`, as they are closely connected.
   * `ViewModel` has a `greetings` property that is an array of `String` phrases.
     SwiftUI connects the view model (`ContentView.ViewModel`) with the view (`ContentView`).
   * `ContentView.ViewModel` is declared as an `ObservableObject`.
   * The `@Published` wrapper is used for the `greetings` property.
   * The `@ObservedObject` property wrapper is used to subscribe to the view model.

Now the view model will emit signals whenever this property changes.

#### Choose a library to consume flows from iOS

In this tutorial, you can choose between the [SKIE](https://github.com/touchlab/SKIE/)
and [KMP-NativeCoroutines](https://github.com/rickclephas/KMP-NativeCoroutines) libraries to help you work with flows in iOS. Both are open-source solutions
that support cancellation and generics with flows, which the Kotlin/Native compiler doesn't yet provide by default.

The SKIE library augments the Objective-C API produced by the Kotlin compiler: SKIE transforms flows into an equivalent of
Swift’s `AsyncSequence`. SKIE directly supports Swift's `async`/`await`, without thread restriction, and with automatic bi-directional
cancellation (Combine and RxSwift require adapters). SKIE offers other features to produce a Swift-friendly API from Kotlin,
including bridging various Kotlin types to Swift equivalents. It also doesn’t require adding additional dependencies in iOS projects.

The KMP-NativeCoroutines library helps you consume suspending functions and flows from iOS by generating necessary
wrappers. KMP-NativeCoroutines supports Swift's `async`/`await` functionality as well as Combine and RxSwift. It has been
available longer than SKIE, and thus you may encounter fewer edge cases with it today. Using KMP-NativeCoroutines requires
adding a Cocoapod or SPM dependency in iOS projects.

#### Option 1. Configure SKIE {initial-collapse-state="collapsed"}

> We recommend using the latest version of the library.
> Check the [SKIE repository](https://github.com/touchlab/SKIE/releases) to see whether a newer version of the plugin is available.
>
{type="note"}

To set up the library, specify the SKIE plugin in `shared/build.gradle.kts` and click the **Sync Now** button.

```kotlin
plugins {
   id("co.touchlab.skie") version "%skieVersion%"
}
```

##### Consume the flow using SKIE

Return to Xcode and update the code using the library:

1. Use a loop and the `await` mechanism to iterate through the `Greeting().greet()` flow and update the `greetings`
   property every time the flow emits a value.
2. Make sure `ViewModel` is marked with the `@MainActor` annotation. The annotation ensures that all asynchronous operations within
   `ViewModel` run on the main thread to comply with the Kotlin/Native requirement:

    ```Swift
    // ...
    extension ContentView {
        @MainActor
        class ViewModel: ObservableObject {
            @Published var greetings: [String] = []
            
            func startObserving() async {
                for await phrase in Greeting().greet() {
                    self.greetings.append(phrase)
                }
            }
        }
    }
    ```

3. Re-run the **iosApp** configuration from Android Studio to make sure your app's logic is synced:

   ![Final results](multiplatform-mobile-upgrade-ios.png){width=300}

> You can find the final state of the project in our [GitHub repository](https://github.com/kotlin-hands-on/get-started-with-kmp/tree/main/step5skie).
>
{type="tip"}

#### Option 2. Configure KMP-NativeCoroutines {initial-collapse-state="collapsed"}

> We recommend using the latest version of the library.
> Check the [KMP-NativeCoroutines repository](https://github.com/rickclephas/KMP-NativeCoroutines/releases) to see whether a newer version of the plugin is available.
>
{type="note"}

1. Return to Android Studio. In the `build.gradle.kts` file of the _whole project_,
   add the KSP (Kotlin Symbol Processor) and KMP-NativeCoroutines plugins to the `plugins {}` block:

    ```kotlin
    plugins {
        // ...
        id("com.google.devtools.ksp").version("2.0.0-1.0.22").apply(false)
        id("com.rickclephas.kmp.nativecoroutines").version("%kmpncVersion%").apply(false)
    }
    ```

2. In the _shared_ `build.gradle.kts` file, configure the KMP-NativeCoroutines plugin:

    ```kotlin
    plugins {
        // ...
        id("com.google.devtools.ksp")
        id("com.rickclephas.kmp.nativecoroutines")
    }
    ```

3. In the _shared_ `build.gradle.kts` file, opt-in to the experimental `@ObjCName` annotation:

    ```kotlin
    kotlin {
        // ...
        sourceSets{
            all {
                languageSettings.optIn("kotlin.experimental.ExperimentalObjCName")
            }
            // ...
        }
    }
    ```

4. Synchronize the Gradle files by clicking **Sync Now** in the notification.

##### Mark the flow with KMP-NativeCoroutines

1. Open the `Greeting.kt` file in the `shared/src/commonMain/kotlin` directory.
2. Add the `@NativeCoroutines` annotation to the `greet()` function. This will ensure that the plugin generates the right
   code to support correct flow handling on iOS:

   ```kotlin
    import com.rickclephas.kmp.nativecoroutines.NativeCoroutines
    
    class Greeting {
        // ...
       
        @NativeCoroutines
        fun greet(): Flow<String> = flow {
            // ...
        }
    }
    ```

##### Import the library using SPM in XCode

1. In Xcode, right-click the `iosApp` project in the left-hand menu and select **Add Package Dependencies**.
2. In the search bar, enter the package name:

     ```none
    https://github.com/rickclephas/KMP-NativeCoroutines.git
    ```

   ![Importing KMP-NativeCoroutines](multiplatform-import-kmp-nativecoroutines.png){width=700}

3. Keep the default options ("Branch" and "master" for **Dependency Rule**), then click the **Add Package** button.
4. In the next window, select "KMPNativeCoroutinesAsync" and "KMPNativeCoroutinesCore", and then click **Add Package**:

   ![Add KMP-NativeCoroutines packages](multiplatform-add-package.png){width=500}

This should install the KMP-NativeCoroutines package necessary to work with the `async/await` mechanism.

##### Consume the flow using the KMP-NativeCoroutines library

1. In `iosApp/ContentView.swift`, update the `startObserving()` function to consume the flow using KMP-NativeCoroutine's
   `asyncSequence()` function for the `Greeting().greet()` function:

    ```Swift
    func startObserving() async {
        do {
            let sequence = asyncSequence(for: Greeting().greet())
            for try await phrase in sequence {
                self.greetings.append(phrase)
            }
        } catch {
            print("Failed with error: \(error)")
        }
    }
    ```

   The loop and the `await` mechanism here are used here to iterate through the flow and update the `greetings` property
every time the flow emits a value.

2. Make sure `ViewModel` is marked with the `@MainActor` annotation. The annotation ensures that all asynchronous operations within
   `ViewModel` run on the main thread to comply with the Kotlin/Native requirement:

    ```Swift
    // ...
    import KMPNativeCoroutinesAsync
    import KMPNativeCoroutinesCore
    
    // ...
    extension ContentView {
        @MainActor
        class ViewModel: ObservableObject {
            @Published var greetings: Array<String> = []
    
            func startObserving() async {
                do {
                    let sequence = asyncSequence(for: Greeting().greet())
                    for try await phrase in sequence {
                        self.greetings.append(phrase)
                    }
                } catch {
                    print("Failed with error: \(error)")
                }
            }
        }
    }
    ```

3. Re-run the **iosApp** configuration from Android Studio to make sure your app's logic is synced:

   ![Final results](multiplatform-mobile-upgrade-ios.png){width=300}

> You can find the final state of the project in our [GitHub repository](https://github.com/kotlin-hands-on/get-started-with-kmp/tree/main/step5).
>
{type="tip"}

## Next step

In the final part of the tutorial, you'll wrap up your project and see what steps to take next.

**[Proceed to the next part](multiplatform-wrap-up.md)**

### See also

* Explore various approaches to [composition of suspending functions](https://kotlinlang.org/docs/composing-suspending-functions.html).
* Learn more about the [interoperability with Objective-C frameworks and libraries](https://kotlinlang.org/docs/native-objc-interop.html).
* Complete this tutorial on [networking and data storage](multiplatform-ktor-sqldelight.md).

## Get help

* **Kotlin Slack**. Get an [invite](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up) and join the [#multiplatform](https://kotlinlang.slack.com/archives/C3PQML5NU) channel.
* **Kotlin issue tracker**. [Report a new issue](https://youtrack.jetbrains.com/newIssue?project=KT).
