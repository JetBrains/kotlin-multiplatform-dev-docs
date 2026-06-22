[//]: # (title: Shared logic for REST API requests)

<secondary-label ref="IntelliJ IDEA"/>
<secondary-label ref="Android Studio"/>

Let's build a more complex application that shares the code for network requests and data serialization between iOS and Android.
In this tutorial, you will:

1. Add common and platform-specific dependencies.
2. Implement network requests and data serialization in common code.
3. Implement a coroutine flow to handle the results of network requests.
4. Consume the data provided by the common module in the UI of iOS and Android apps.

Now that you've implemented common logic using external dependencies, you can start adding more complex logic. Network
requests and data serialization are the [most popular use cases](https://kotlinlang.org/lp/multiplatform/) for sharing code using Kotlin
Multiplatform. Learn how to implement these in your first application so that after completing this onboarding journey, 
you can use them in future projects.

The updated app will retrieve data over the internet from the [LaunchLibrary 2](https://lldev.thespacedevs.com/docs)
API and display the date of the last successful launch of a SpaceX rocket.

> You can find the final state of the project in two branches of our GitHub repository, with different coroutine solutions:
> * the [`main`](https://github.com/kotlin-hands-on/get-started-with-kmp/tree/main) branch includes a KMP-NativeCoroutines implementation,
> * the [`main-skie`](https://github.com/kotlin-hands-on/get-started-with-kmp/tree/main-skie) branch includes a SKIE implementation.
>
{style="note"}

## Add more dependencies

You'll need to add the following multiplatform libraries in your project:

* [`kotlinx.coroutines`](https://github.com/Kotlin/kotlinx.coroutines), to use coroutines for simultaneous operations.
* [`kotlinx.serialization`](https://github.com/Kotlin/kotlinx.serialization), to deserialize JSON responses of the SpaceX API into objects of entity classes used to process
  network operations.
* [Ktor](https://ktor.io/), a framework for sending and retrieving data over HTTP.

### Update the Gradle version catalog

Add the following entries to `gradle/libs.versions.toml`, then sync Gradle files to make the references available
in build configuration code:

```text
[versions]
coroutinesVersion = "%coroutinesVersion%"
ktorVersion = "%ktorVersion%"
# A Kotlin version should already be set in the catalog
kotlin = "%kotlinVersion%"

[libraries]
kotlinx-coroutines = { module = "org.jetbrains.kotlinx:kotlinx-coroutines-core", version.ref = "coroutinesVersion" }
ktor-client-core = { module = "io.ktor:ktor-client-core", version.ref = "ktorVersion" }
ktor-client-content-negotiation = { module = "io.ktor:ktor-client-content-negotiation", version.ref = "ktorVersion" }
ktor-serialization-kotlinx-json = { module = "io.ktor:ktor-serialization-kotlinx-json", version.ref = "ktorVersion" }
ktor-client-darwin = { module = "io.ktor:ktor-client-darwin", version.ref = "ktor" }
ktor-client-android = { module = "io.ktor:ktor-client-android", version.ref = "ktor" }

[plugins]
kotlinSerialization = { id = "org.jetbrains.kotlin.plugin.serialization", version.ref = "kotlin" }
```

### Add dependencies to corresponding source sets

Add the library references to corresponding source sets in the `sharedLogic/build.gradle.kts` file:

```kotlin
plugins {
    // ...
    alias(libs.plugins.kotlinSerialization)
}

kotlin {
    sourceSets {
        commonMain.dependencies {
            // ...
            // The Kotlin Multiplatform Gradle plugin adds
            // platform-specific coroutines artifacts automatically
            implementation(libs.kotlinx.coroutines.core)
            // Main Ktor dependency
            implementation(libs.ktor.client.core)
            // Dependencies that allow Ktor to use serialization
            // with a specific format
            implementation(libs.ktor.client.content.negotiation)
            implementation(libs.ktor.serialization.kotlinx.json)
        }
        androidMain.dependencies {
            // Provides the Android engine for Ktor
            implementation(libs.ktor.client.android)
        }
        iosMain.dependencies {
            // Provides the Darwin engine for Ktor
            implementation(libs.ktor.client.darwin)
        }
    }
}
```

Synchronize the Gradle files by clicking the **Sync Gradle Changes** button.

## Set up API requests

You'll use the [Launch Library API](https://github.com/r-spacex/SpaceX-API/tree/master/docs#rspacex-api-docs) to retrieve data,
specifically the list of all launches from the **/2.3.0/launches** endpoint.

### Create a data model

In the `sharedLogic/src/commonMain/.../greetingkmp` directory, create a new `RocketLaunch.kt` file
and add a data class which stores data from the SpaceX API:

```kotlin
import kotlinx.serialization.SerialName
import kotlinx.serialization.Serializable

@Serializable
data class RocketLaunch(
    @SerialName("id")
    val id: String,
    @SerialName("name")
    val missionName: String,
    @SerialName("net")
    val launchDateUTC: String,
    @SerialName("status")
    val status: LaunchStatus,
)

@Serializable
data class LaunchStatus(
    @SerialName("id")
    val id: Int,
    @SerialName("name")
    val name: String,
)

@Serializable
data class LaunchListResponse(
    @SerialName("results")
    val results: List<RocketLaunch>,
)
```

* The `RocketLaunch` class is marked with the `@Serializable` annotation so that the `kotlinx.serialization` plugin can
  automatically generate a default serializer for it.
* The `@SerialName` annotation allows you to redefine field names, making it possible to declare properties with more readable names
  in data classes.

### Connect HTTP client

1. In the `sharedLogic/src/commonMain/.../greetingkmp` directory, create a new `RocketComponent` class.
2. Add the `httpClient` property to retrieve rocket launch information through an HTTP GET request:

    ```kotlin
    import io.ktor.client.HttpClient
    import io.ktor.client.plugins.contentnegotiation.ContentNegotiation
    import io.ktor.serialization.kotlinx.json.json
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

   * The [`ContentNegotiation`](https://ktor.io/docs/serialization-client.html#register_json) Ktor plugin and the JSON serializer deserialize the result of the GET request.
   * The JSON serializer here is configured in such a way that it prints JSON in a more readable manner with the `prettyPrint` property.
     This is more flexible when reading malformed JSON with `isLenient`,
     and it ignores keys that haven't been declared in the rocket launch model with `ignoreUnknownKeys`.

3. Add the `getDateOfLastSuccessfulLaunch()` [suspending function](https://kotlinlang.org/docs/coroutines-basics.html) to `RocketComponent`,
   which will retrieve information about rocket launches asynchronously:

   ```kotlin
   import io.ktor.client.request.get
   import io.ktor.client.call.body
   
   class RocketComponent {
       // ...
       
       private suspend fun getDateOfLastSuccessfulLaunch(): String {
           val rockets: List<RocketLaunch> = httpClient.get("https://api.spacexdata.com/v4/launches").body()
   
           // Initialized with a stub date for now
           val date: String = "October 5, 2026"
        
           return "$date"
       }
   }
   ```

   * `httpClient.get()` is also a suspending function
     because it needs to retrieve data over the network asynchronously without blocking threads.
   * Suspending functions can only be called from coroutines or other suspending functions.
     This is why `getDateOfLastSuccessfulLaunch()` was marked with the `suspend` keyword.
     The network request is executed in the HTTP client's thread pool.

4. After the HTTP request call, add the call to get the last successful launch in the list
   (the list of launches is sorted by date from oldest to newest):

   ```kotlin
   class RocketComponent {
       // ...
       
       private suspend fun getDateOfLastSuccessfulLaunch(): String {
           val response: LaunchListResponse =
               httpClient.get("https://lldev.thespacedevs.com/2.3.0/launches/previous/?mode=list&limit=10&format=json").body()
           val lastSuccessLaunch = response.results.first { it.status.id == 3 }
           val date: String = "October 5, 2026"
           
           return "$date"
       }
   }
   ```

5. Conversion the UTC date and time of a launch to your local date and assign the result to `date`.
   Then return the formatted output:

   ```kotlin
   import kotlinx.datetime.TimeZone
   import kotlinx.datetime.toLocalDateTime
   import kotlin.time.ExperimentalTime
   import kotlin.time.Instant

   class RocketComponent {
       // ...
       
       private suspend fun getDateOfLastSuccessfulLaunch(): String {
           val response: LaunchListResponse =
               httpClient.get("https://lldev.thespacedevs.com/2.3.0/launches/previous/?mode=list&limit=10&format=json").body()
           val lastSuccessLaunch = response.results.first { it.status.id == 3 }
           val date = Instant.parse(lastSuccessLaunch.launchDateUTC)
               .toLocalDateTime(TimeZone.currentSystemDefault())
       
           return "${date.month} ${date.day}, ${date.year}"
       }
   }
   ```

   The date will be displayed in the "MMMM DD, YYYY" format, for example, "OCTOBER 5, 2022".

6. To the same class, add another suspending function, `launchPhrase()`,
   which will create a message using the `getDateOfLastSuccessfulLaunch()` function:

    ```kotlin
    class RocketComponent {
        // ...
    
        suspend fun launchPhrase(): String =
            try {
                "The last successful launch was on ${getDateOfLastSuccessfulLaunch()} 🚀"
            } catch (e: Exception) {
                println("Exception during getting the date of the last successful launch $e")
                "Error occurred"
            }
    }
    ```

### Create a coroutine flow

Instead of simply calling a suspending function, you can use [flows](https://kotlinlang.org/docs/flow.html)
when you need to produce a sequence of values.
Flows can emit a sequence of values as the values are produced instead of returning a single value like suspending functions.

1. Open the `Greeting.kt` file in the `shared/src/commonMain/kotlin` directory.
2. Add a `rocketComponent` property to the `Greeting` class. The property will store the message with the last successful launch date:

   ```kotlin
   class Greeting {
       private val rocketComponent = RocketComponent()
       //...
   }
   ```

3. Change the `greet()` function to return a `Flow`:

    ```kotlin
    import kotlinx.coroutines.delay
    import kotlinx.coroutines.flow.Flow
    import kotlinx.coroutines.flow.flow
    import kotlin.time.Duration.Companion.seconds
    
    class Greeting {
        // ...
        fun greet(): Flow<String> = flow {
            emit(if (Random.nextBoolean()) "Hi!" else "Hello!")
            delay(1.seconds)
            emit("Guess what this is! > ${platform.name.reversed()}")
            delay(1.seconds)
            emit(daysPhrase())
            emit(rocketComponent.launchPhrase())
        }
    }
    ```

   * The `Flow` is created here with the `flow()` builder function, which wraps all the statements.
   * The `Flow` emits strings with a delay of one second between each emission. The last element is only emitted after
     the network response returns, so the exact delay depends on your network.

You've updated the API of the shared module by changing the return type of the `greet()` function to `Flow`.
Now you need to update native parts of the project so that they can properly handle the result of calling
the `greet()` function.

## Update native Android UI

As both the shared module and the Android application are written in Kotlin, using shared code from Android is straightforward.

### Introduce a view model

View models are a popular pattern in Android development that helps manage data and other app components that should
persist through [Android activity](https://developer.android.com/guide/components/activities/intro-activities) lifecycle.
Now that the application is becoming more complex, it's time to introduce a view model into our app as well.
It will store the data received from the SpaceX API and make it available to the UI.

Create the view model class in the Android platform code:

1. In the `sharedUI/src/commonMain/.../greetingkmp` directory, create a new `MainViewModel` Kotlin class:

    ```kotlin
    import androidx.lifecycle.ViewModel
    
    class MainViewModel : ViewModel() {
        // ...
    }
    ```

   This class extends Android's `ViewModel` class to align with the platform's expectations regarding lifecycle and configuration changes.

2. Create a `greetingList` value of the [StateFlow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/)
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

3. In the `init` function of the view model, collect all the strings from the `Greeting().greet()` flow:

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

   Since the `Flow.collect()` function is suspending, the `launch` coroutine is used within the view model's scope.
   This means that the launch coroutine will run only during the correct phases of the view model's lifecycle.

4. Inside the `collect` trailing lambda, append the collected `phrase` to the list of phrases in `_greetingList` using
   the `update()` function:

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

### Use the view model's flow

1. In `sharedUI/src/commonMain/.../greetingkmp`, open the `App.kt` file and update it,
   replacing the previous implementation to use the newly implemented view model:

    ```kotlin
    import androidx.lifecycle.compose.collectAsStateWithLifecycle
    import androidx.compose.runtime.getValue
    import androidx.lifecycle.viewmodel.compose.viewModel
    
    @Composable
    @Preview
    fun App(mainViewModel: MainViewModel = viewModel()) {
        MaterialTheme {
            val greetings by mainViewModel.greetingList.collectAsStateWithLifecycle()
    
            Column(
                modifier = Modifier
                    .safeContentPadding()
                    .fillMaxSize(),
                verticalArrangement = Arrangement.spacedBy(8.dp),
            ) {
                greetings.forEach { greeting ->
                    Text(greeting)
                    HorizontalDivider()
                }
            }
        }
    }
    ```

   * The `collectAsStateWithLifecycle()` function calls on `greetingList` to collect the value from the ViewModel's flow
     and represent it as a composable state in a lifecycle-aware manner.
   * When a new flow is created, the composition state will change and display a scrollable `Column` with greeting phrases
     arranged vertically and separated by dividers.

### Add internet access permission

To access the internet, the Android application needs the appropriate permission. Since all network requests are made from the
shared module, it makes sense to add the internet access permission to its manifest.

Update your `androidApp/src/main/AndroidManifest.xml` file with the access permission:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <uses-permission android:name="android.permission.INTERNET"/>
    ...
</manifest>
```

### Run the app

To see the final result, rerun your **androidApp** run configuration:

![Final result for Android](multiplatform-mobile-upgrade-android.png){width=350}

## Update native iOS UI

For the iOS part of the project, you'll make use of the [Model–view–viewmodel](https://en.wikipedia.org/wiki/Model–view–viewmodel)
pattern, like you did for the Android app, to connect the UI to the `sharedLogic` module.

The module is already imported in the `ContentView.swift` file with the `import SharedLogic` declaration.

### Introducing a ViewModel

In `iosApp/ContentView.swift`, create a `ViewModel` class for `ContentView`, which will prepare and manage data for it.
Call the `startObserving()` function within a `task()` call to support concurrency:

```swift
import SwiftUI
import SharedLogic/*  */

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

SwiftUI connects the view model (`ContentView.ViewModel`) with the view (`ContentView`):

* `ContentView.ViewModel` is declared as an `ObservableObject`.
  The `@ObservedObject` wrapper for the `viewModel` property in `ContentView` subscribes the view to the view model.
* The `greetings` property of the view model uses the `@Published` wrapper.
  It allows SwiftUI to automatically update the view when this property changes.

Now you need to implement the `startObserving()` function to consume flows.

### Choose a library to consume flows from iOS

In this tutorial, you can use [SKIE](https://skie.touchlab.co/) the [KMP-NativeCoroutines](https://github.com/rickclephas/KMP-NativeCoroutines) library
to help you work with flows in iOS.
Both are open-source solutions that support cancellation and generics with flows,
which the Kotlin/Native compiler doesn't yet provide by default:

* The KMP-NativeCoroutines library helps you consume suspending functions and flows from iOS by generating the necessary
  wrappers.
  KMP-NativeCoroutines supports Swift's `async`/`await` functionality as well as Combine and RxSwift.
  Using KMP-NativeCoroutines requires adding a SwiftPM or CocoaPod dependency in iOS projects.
* The SKIE library augments the Objective-C API produced by the Kotlin compiler: SKIE transforms flows into an equivalent of
Swift’s `AsyncSequence`. SKIE directly supports Swift's `async`/`await`, without thread restriction, and with automatic bidirectional
cancellation (Combine and RxSwift require adapters). SKIE offers other features to produce a Swift-friendly API from Kotlin,
including bridging various Kotlin types to Swift equivalents. It also doesn’t require adding additional dependencies in iOS projects.

### Option 1. Configure KMP-NativeCoroutines {initial-collapse-state="collapsed" collapsible="true"}

> We recommend using the latest version of the library.
> Check the [KMP-NativeCoroutines repository](https://github.com/rickclephas/KMP-NativeCoroutines/releases)
> to see whether a newer version of the plugin is available and whether it is compatible with your Kotlin version.
>
{style="note"}

1. Add the KMP-NativeCoroutines version and plugin reference to the Gradle version catalog:

    ```text
    [versions]
    kmpNativeCoroutines = "%kmpncVersion%"
    
    [plugins]
    kmpNativeCoroutines = { id = "com.rickclephas.kmp.nativecoroutines", version.ref = "kmpNativeCoroutines" }
    ```

2. In the root `build.gradle.kts` file of your project (**not** the `shared/build.gradle.kts` file),
   add the KMP-NativeCoroutines plugin to the `plugins {}` block:

    ```kotlin
    plugins {
        // ...
        alias(libs.plugins.kmpNativeCoroutines) apply false
    }
    ```

3. In the `sharedLogic/build.gradle.kts` file, add the KMP-NativeCoroutines plugin to the `plugins {}` block:

    ```kotlin
    plugins {
        // ...
        alias(libs.plugins.kmpNativeCoroutines)
    }
    ```

4. In the same `sharedLogic/build.gradle.kts` file, opt-in to the experimental `@ObjCName` annotation:

    ```kotlin
    kotlin {
        // ...
        sourceSets{
            all {
                languageSettings {
                    optIn("kotlin.experimental.ExperimentalObjCName")
                }
            }
            // ...
        }
    }
    ```

5. Click the **Sync Gradle Changes** button to synchronize the Gradle files.

#### Mark the flow with KMP-NativeCoroutines

1. Open the `Greeting.kt` file in the `sharedLogic/src/commonMain/kotlin` directory.
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

#### Import the library using SwiftPM in XCode

Installs the parts of the KMP-NativeCoroutines Swift package necessary to work with the `async/await` mechanism.

1. Go to **File | Open Project in Xcode**.
2. In Xcode, right-click the `iosApp` project in the left-hand menu and select **Add Package Dependencies**.
3. In the search bar, enter the package name:

     ```none
    https://github.com/rickclephas/KMP-NativeCoroutines.git
    ```

   ![Importing KMP-NativeCoroutines](multiplatform-import-kmp-nativecoroutines.png){width=700}

4. In the **Dependency Rule** dropdown, select the **Exact Version** item and enter the `%kmpncVersion%` version in the adjacent field.
5. Click the **Add Package** button. Xcode will fetch the package from GitHub and open another window to choose package products.
6. Add "KMPNativeCoroutinesAsync" and "KMPNativeCoroutinesCore" to your app as shown, then click **Add Package**:

   ![Add KMP-NativeCoroutines packages](multiplatform-add-package.png){width=500}
7. Return to IntelliJ IDEA and select the **Tools | Swift Package Manager | Resolve Dependencies menu item**.
   This creates a `Package.resolved` lock file that is used by the Kotlin build
   and can be commited to the repository to keep the versions of Swift packages consistent.  

#### Consume the flow using the KMP-NativeCoroutines library

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

### Option 2. Configure SKIE {initial-collapse-state="collapsed" collapsible="true"}

To set up the library, add the SKIE version and plugin reference to your Gradle version catalog:

```text
[versions]
skie = "%skieVersion%"

[plugins]
skie = { id = "co.touchlab.skie", version.ref = "skie" }
```

> SKIE may not support the latest Kotlin version.
> If your Kotlin version is too new, this is reported during Gradle sync along with the list of versions you can safely
> downgrade to.
> 
{style="note"}

Then add it to the list of plugins in the `sharedLogic/build.gradle.kts` file and click the **Sync Gradle Changes** button:

```kotlin
plugins {
    //...
    alias(libs.plugins.skie)
}
```

#### Consume the flow using SKIE

You'll use a loop and the `await` mechanism to iterate through the `Greeting().greet()` flow and update the `greetings`
property every time the flow emits a value.

> IntelliJ IDEA and Android Studio can incorrectly report Swift errors in calls to Kotlin code while using SKIE.
> This is a known issue with the library, which doesn't affect building and running the app.
>
{style="warning"}

Make sure `ViewModel` is marked with the `@MainActor` annotation.
The annotation ensures that all asynchronous operations within `ViewModel` run on the main thread
to comply with the Kotlin/Native requirement:

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

### Consume the ViewModel and run the iOS app

In `iosApp/iOSApp.swift`, update the entry point for your app:

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

Run the **iosApp** configuration from IntelliJ IDEA to make sure your app's logic is synced:

![Final results](multiplatform-mobile-upgrade-ios.png){width=350}

> You can find the final state of the project in two branches of our GitHub repository, with different coroutine solutions:
> * the [`main`](https://github.com/kotlin-hands-on/get-started-with-kmp/tree/main) branch includes a KMP-NativeCoroutines implementation,
> * the [`main-skie`](https://github.com/kotlin-hands-on/get-started-with-kmp/tree/main-skie) branch includes a SKIE implementation.
>
{style="note"}

### See also

* Explore various approaches to [composition of suspending functions](https://kotlinlang.org/docs/composing-suspending-functions.html).
* Learn more about the [interoperability with Objective-C frameworks and libraries](https://kotlinlang.org/docs/native-objc-interop.html).
* Complete this tutorial on [networking and data storage](multiplatform-ktor-sqldelight.md).

## Get help

* **Kotlin Slack**. Get an [invite](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up) and join the [#multiplatform](https://kotlinlang.slack.com/archives/C3PQML5NU) channel.
* **Kotlin issue tracker**. [Report a new issue](https://youtrack.jetbrains.com/newIssue?project=KT).
