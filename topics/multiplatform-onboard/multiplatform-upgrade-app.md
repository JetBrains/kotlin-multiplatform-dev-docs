[//]: # (title: Shared logic for REST API requests)

<secondary-label ref="IntelliJ IDEA"/>
<secondary-label ref="Android Studio"/>

In this tutorial you will start with the wizard-generated KMP project and build a more complex application
that shares the code for network requests and data serialization between iOS and Android.

In the course of the tutorial, you will:

1. Add common and platform-specific dependencies.
2. Implement network requests and data serialization in common code.
3. Implement a coroutine flow to handle the results of network requests.
4. Consume the data provided by the common module in the UI of iOS and Android apps.

Network requests and data serialization are the [most popular use cases](https://kotlinlang.org/lp/multiplatform/) for sharing code using Kotlin
Multiplatform.
You will implement both in the shared module of the final project:
An app that retrieves data over the internet from the [LaunchLibrary 2](https://lldev.thespacedevs.com/docs)
API and displays the date of the latest successful launch.

The updated app will retrieve data over the internet from the [LaunchLibrary 2](https://lldev.thespacedevs.com/docs)
API and display descriptions of latest space launches.

> You can find the final state of the project in two branches of our GitHub repository, with different iOS coroutine solutions:
> * the [`main`](https://github.com/kotlin-hands-on/get-started-with-kmp/tree/main) branch includes a KMP-NativeCoroutines implementation,
> * the [`main-skie`](https://github.com/kotlin-hands-on/get-started-with-kmp/tree/main-skie) branch includes a SKIE implementation.
>
{style="note"}

## Create a project

Create the same project as in the [basic KMP tutorial](multiplatform-create-first-app.md):
Targeting Android and iOS, with the **Do not share UI** option selected for iOS.

## Add dependencies

You'll need to add the following multiplatform libraries in your project:

* [`kotlinx-datetime`](https://github.com/Kotlin/kotlinx-datetime), to process and format timestamps.
* [Ktor](https://ktor.io/), a framework for sending and retrieving data over HTTP.
* [`kotlinx.coroutines`](https://github.com/Kotlin/kotlinx.coroutines), to use coroutines for simultaneous operations.
* [`kotlinx.serialization`](https://github.com/Kotlin/kotlinx.serialization), to deserialize JSON responses of the SpaceX API into objects of entity classes used to process
  network operations.

### Update the Gradle version catalog

Add the following entries to `gradle/libs.versions.toml`, then sync Gradle files to make the references available
in build configuration code:

```toml
[versions]
kotlinx-coroutines = "%coroutinesVersion%"
kotlinx-datetime = "%dateTimeVersion%"
ktor = "%ktorVersion%"
# A Kotlin version should already be set in a generated project
kotlin = "%kotlinVersion%"

[libraries]
kotlinx-coroutines = { module = "org.jetbrains.kotlinx:kotlinx-coroutines-core", version.ref = "kotlinx-coroutines" }
kotlinx-datetime = { module = "org.jetbrains.kotlinx:kotlinx-datetime", version.ref = "kotlinx-datetime" }
ktor-client-core = { module = "io.ktor:ktor-client-core", version.ref = "ktor" }
ktor-client-content-negotiation = { module = "io.ktor:ktor-client-content-negotiation", version.ref = "ktor" }
ktor-serialization-kotlinx-json = { module = "io.ktor:ktor-serialization-kotlinx-json", version.ref = "ktor" }
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
            // platform-specific artifacts for coroutines and datetime
            // automatically
            implementation(libs.kotlinx.coroutines)
            implementation(libs.kotlinx.datetime)
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
    import io.ktor.client.call.body
    import io.ktor.client.plugins.contentnegotiation.ContentNegotiation
    import io.ktor.client.request.get
    import io.ktor.serialization.kotlinx.json.json
    import kotlinx.datetime.TimeZone
    import kotlinx.datetime.toLocalDateTime
    import kotlinx.serialization.json.Json
    import kotlin.time.Instant
    
    class RocketComponent {
        private val httpClient = HttpClient {
            // ContentNegotiation Ktor plugin and the JSON serializer
            // deserialize the result of the GET request
            install(ContentNegotiation) {
                json(Json {
                    // Produces more readable JSON
                    prettyPrint = true
                    // Relaxed requirements for the JSON format
                    isLenient = true
                    // Ignores keys that haven't been declared in the model
                    ignoreUnknownKeys = true
                })
            }
        }

        // Returns the date string for the latest successful launch.
        // Marked as suspending because it calls
        // the suspending httpClient.get() function
        private suspend fun getDateOfLastSuccessfulLaunch(): String {
           // Asynchronously retrieves information about rocket launches
           val response: LaunchListResponse =
               httpClient.get("https://lldev.thespacedevs.com/2.3.0/launches/previous/?mode=list&limit=10&format=json").body()
           // Gets the latest successful launch
           // (in the response they are sorted from newest to oldest)
           val lastSuccessLaunch = response.results.first { it.status.id == 3 }
           // Converts the launch timestamp to local time.
           // Date is displayed in the "MMMM DD, YYYY" format,
           // for example, "OCTOBER 5, 2022"
           val date = Instant.parse(lastSuccessLaunch.launchDateUTC)
               .toLocalDateTime(TimeZone.currentSystemDefault())
        
           return "$date"
       }
   
       // Builds the final string for the UI using
       // the suspending getDateOfLastSuccessfulLaunch() function
       suspend fun launchPhrase(): String =
            try {
                "The last successful launch was on ${getDateOfLastSuccessfulLaunch()} 🚀"
            } catch (e: Exception) {
                println("Exception during getting the date of the last successful launch $e")
                "Error occurred"
            }
    }
    ```

   Suspending functions can only be called from coroutines or other suspending functions.
   For example, `httpClient.get()` is a suspending function because it needs to retrieve data over the network asynchronously without blocking threads.
   Since the `getDateOfLastSuccessfulLaunch()` function calls `httpClient.get()`, it is also marked with the `suspend` keyword.

### Create a coroutine flow

Instead of simply calling a suspending function, you can use [flows](https://kotlinlang.org/docs/flow.html)
when you need to produce a sequence of values.
Flows can emit a sequence of values as the values are produced instead of returning a single value like suspending functions.

1. Open the `Greeting.kt` file in the `shared/src/commonMain/kotlin` directory.
2. Update the `greet()` function in the `Greeting` class to return a `Flow` of strings,
   primarily to accommodate the network request.
   As a part of the `Flow`, emit the launch date using a `RocketComponent` property:

    ```kotlin
    import kotlinx.coroutines.delay
    import kotlinx.coroutines.flow.Flow
    import kotlinx.coroutines.flow.flow
    import kotlin.random.Random
    import kotlin.time.Duration.Companion.seconds
    
    class Greeting {
        private val platform = getPlatform()
   
        // Stores the last successful launch date
        private val rocketComponent = RocketComponent()
        // Builds and asynchronously emits greeting strings one by one
        fun greet(): Flow<String> = flow {
            emit(if (Random.nextBoolean()) "Hi!" else "Hello!")
            delay(1.seconds)
            emit("Guess what this is! > ${platform.name.reversed()}")
            emit(rocketComponent.launchPhrase())
        }
    }
    ```

    The `Flow` is created with the [`flow()`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/flow.html)
    builder function that wraps a suspendable block.

The `greet()` function now returns a `Flow` of strings instead of a single string.
To make the native UI parts of the project work,
update them to handle the new result of calling the `greet()` function.

## Update native Android UI

As both the shared module and the Android application are written in Kotlin, using shared code from Android is straightforward.

### Introduce a view model

View models are a popular pattern in Android development that helps manage data and other app components that should
persist through [Android activity](https://developer.android.com/guide/components/activities/intro-activities) lifecycle.
Now that the application is becoming more complex, it's time to introduce a view model into our app as well.
It will store the data received from the SpaceX API and make it available to the UI.

Create the view model class in the Android platform code:

1. In the `sharedUI/src/commonMain/.../greetingkmp` directory, create a new `MainViewModel` class that extends Android's `ViewModel`
   to use the platform's lifecycle mechanism and configuration tracking:

    ```kotlin
    import androidx.lifecycle.ViewModel
    import androidx.lifecycle.viewModelScope
    import kotlinx.coroutines.flow.MutableStateFlow
    import kotlinx.coroutines.flow.StateFlow
    import kotlinx.coroutines.flow.update
    import kotlinx.coroutines.launch
    
    class MainViewModel: ViewModel() {
        // StateFlow extends the Flow interface
        // but has a single value or state
        val greetingList: StateFlow<List<String>>
            field = MutableStateFlow<List<String>>(listOf())
    
        // Collects all strings emitted by a Greeting().greet() call
        init {
            // Wraps the call to the suspending Flow.collect() function.
            // The launch() coroutine is used within the ViewModel scope,
            // so it will run only in correct phases of the lifecycle
            viewModelScope.launch {
                // Appends each new phrase to greetingList
                Greeting().greet().collect { phrase ->
                    greetingList.update { list -> list + phrase }
                }
            }
        }
    }
    ```
   
    The `greetingList` property is declared with an [explicit backing field](https://kotlinlang.org/docs/properties.html#explicit-backing-fields)
    to make sure that it is read-only outside the class and mutable internally.

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

In `sharedUI/src/commonMain/.../greetingkmp`, open the `App.kt` file
and replace the previous implementation to use the newly implemented view model.

When a new flow is created, the composition state will change and display a scrollable `Column` with greeting phrases
arranged vertically and separated by dividers:

```kotlin
import androidx.lifecycle.compose.collectAsStateWithLifecycle
import androidx.compose.runtime.getValue
import androidx.lifecycle.viewmodel.compose.viewModel

@Composable
@Preview
fun App(mainViewModel: MainViewModel = viewModel()) {
    MaterialTheme {
        // Collects the value of greetingList from the ViewModel's flow
        // and represents it as a composable state in a lifecycle-aware manner
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
import SharedLogic

struct ContentView: View {
    @ObservedObject private(set) var viewModel: ViewModel

    var body: some View {
        ListView(phrases: viewModel.greetings)
            .task { await self.viewModel.startObserving() }
    }
}

// ViewModel is declared as an extension to ContentView,
// as they are closely connected
extension ContentView {
    @MainActor
    class ViewModel: ObservableObject {
        // This property will hold the greeting phrases
        // emitted by the ViewModel's flow
        @Published var greetings: Array<String> = []
        
        func startObserving() {
            // The implementation will depend
            // on the chosen iOS coroutine library (see below)
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

    ```toml
    [versions]
    kmpNativeCoroutines = "%kmpncVersion%"
    
    [plugins]
    kmpNativeCoroutines = { id = "com.rickclephas.kmp.nativecoroutines", version.ref = "kmpNativeCoroutines" }
    ```

2. In the root `build.gradle.kts` file of your project (**not** the `sharedLogic/build.gradle.kts` file),
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

```toml
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

## Possible issues and solutions

### Xcode reports errors in the code calling the shared framework

If you work in Xcode, your Xcode project may be using an old version of the framework.
To resolve this, return to IntelliJ IDEA and rebuild the project or start the iOS run configuration.

### Xcode reports an error when importing the shared framework

If you are using Xcode, you may need to clear cached binaries: try resetting the environment by choosing
**Product | Clean Build Folder** in the main menu.

### See also

* Explore various approaches to [composition of suspending functions](https://kotlinlang.org/docs/composing-suspending-functions.html).
* Learn more about the [interoperability with Objective-C frameworks and libraries](https://kotlinlang.org/docs/native-objc-interop.html).
* Complete this tutorial on [networking and data storage](multiplatform-ktor-sqldelight.md).

## Get help

* **Kotlin Slack**. Get an [invite](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up) and join the [#multiplatform](https://kotlinlang.slack.com/archives/C3PQML5NU) channel.
* **Kotlin issue tracker**. [Report a new issue](https://youtrack.jetbrains.com/newIssue?project=KT).
