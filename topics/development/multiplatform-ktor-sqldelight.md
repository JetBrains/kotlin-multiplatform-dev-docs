[//]: # (title: Create a multiplatform app using Ktor and SQLDelight)

This tutorial demonstrates how to use Android Studio to create a mobile application for iOS and Android using Kotlin
Multiplatform.
This application is going to:

* retrieve data over the internet from the public [SpaceX API](https://docs.spacexdata.com/?version=latest) using Ktor,
* save the data in a local database using SQLDelight,
* and display a list of SpaceX rocket launches together with the launch date, results, and a detailed description of the launch.

The application will include a module with shared code for both the iOS and Android platforms. The business logic and data
access layers will be implemented only once in the shared module, while the UI of both applications will be native.

TODO change screenshot
![Emulator and Simulator](android-and-ios.png){width=600}

You will use the following multiplatform libraries in the project:

* [Ktor](https://ktor.io/docs/create-client.html) as an HTTP client for retrieving data over the internet.
* [`kotlinx.serialization`](https://github.com/Kotlin/kotlinx.serialization) to deserialize JSON responses into objects
  of entity classes.
* [`kotlinx.coroutines`](https://github.com/Kotlin/kotlinx.coroutines) to write asynchronous code.
* [SQLDelight](https://github.com/cashapp/sqldelight) to generate Kotlin code from SQL queries and create a type-safe
  database API.
* [Koin](https://insert-koin.io/) to use dependency injection for platform-specific database drivers. (TODO check wording)

> You can find the [template project](https://github.com/kotlin-hands-on/kmm-networking-and-data-storage) as well as the
> source code of the [final application](https://github.com/kotlin-hands-on/kmm-networking-and-data-storage/tree/final)
> in the corresponding GitHub repository. (TODO check links)
>
{type="note"}

## Create your project

1. Prepare your environment for multiplatform development. [Check the list of necessary tools and update them to the latest versions if necessary](multiplatform-setup.md).
2. Open the [Kotlin Multiplatform wizard](https://kmp.jetbrains.com).
3. On the **New project** tab, ensure that the **Android** and **iOS** options are selected.
4. For iOS, choose the **Do not share UI** option. You will implement native UI for both platforms.
5. Click the **Download** button and unpack the downloaded archive.

![Kotlin Multiplatform wizard](multiplatform-web-wizard-test.png){width=450}

<!-- You can find the configured project [on the `master` branch](https://github.com/kotlin-hands-on/kmm-networking-and-data-storage).
> TODO check link and uncomment this
{type="note"} -->

## Add dependencies to the multiplatform library

To add a multiplatform library to the shared module, you need to add dependency instructions (`implementation`)
to the `dependencies` block of the relevant source sets in the `build.gradle.kts` file.

Both the `kotlinx.serialization` and SQLDelight libraries also require additional configuration.

1. Launch Android Studio.
2. On the Welcome screen, click **Open**, or select **File | Open** in the editor.
3. Navigate to the unpacked project folder and then click **Open**.

   Android Studio detects that the folder contains a Gradle build file, opens the folder as a new project,
   and starts the initial Gradle Sync.

4. The default view in Android Studio is optimized for Android development. To see the full file structure of the project,
   which is more convenient for multiplatform development, switch the view from **Android** to **Project**:

   ![Select the project view](select-project-view.png){width=200}

5. Change or add lines to the version catalog in the `gradle/libs.versions/toml` file to reflect all needed dependencies:

   ```Ini
   [versions]
   agp = "8.2.2"
   ...
   coroutinesVersion = "1.7.3"
   dateTimeVersion = "0.4.1"
   koin = "3.5.3"
   ktor = "2.3.7"
   sqlDelight = "2.0.1"
   lifecycleViewmodelCompose = "2.7.0"
   material3 = "1.2.0"
   
   [libraries]
   ...
   android-driver = { module = "app.cash.sqldelight:android-driver", version.ref = "sqlDelight" }
   koin-androidx-compose = { module = "io.insert-koin:koin-androidx-compose", version.ref = "koin" }
   koin-core = { module = "io.insert-koin:koin-core", version.ref = "koin" }
   kotlinx-coroutines-core = { module = "org.jetbrains.kotlinx:kotlinx-coroutines-core", version.ref = "coroutinesVersion" }
   kotlinx-datetime = { module = "org.jetbrains.kotlinx:kotlinx-datetime", version.ref = "dateTimeVersion" }
   ktor-client-android = { module = "io.ktor:ktor-client-android", version.ref = "ktor" }
   ktor-client-content-negotiation = { module = "io.ktor:ktor-client-content-negotiation", version.ref = "ktor" }
   ktor-client-core = { module = "io.ktor:ktor-client-core", version.ref = "ktor" }
   ktor-client-darwin = { module = "io.ktor:ktor-client-darwin", version.ref = "ktor" }
   ktor-serialization-kotlinx-json = { module = "io.ktor:ktor-serialization-kotlinx-json", version.ref = "ktor" }
   native-driver = { module = "app.cash.sqldelight:native-driver", version.ref = "sqlDelight" }
   runtime = { module = "app.cash.sqldelight:runtime", version.ref = "sqlDelight" }
   androidx-lifecycle-viewmodel-compose = { group = "androidx.lifecycle", name = "lifecycle-viewmodel-compose", version.ref = "lifecycleViewmodelCompose" }
   androidx-compose-material3 = { module = "androidx.compose.material3:material3", version.ref="material3" }
   
   [plugins]
   ...
   kotlinxSerialization = { id = "org.jetbrains.kotlin.plugin.serialization", version.ref = "kotlin" }
   sqldelight = { id = "app.cash.sqldelight", version.ref = "sqlDelight" }
   ```
   {initial-collapse-state="collapsed"}
   (TODO collapsed title)

6. Run a Gradle Sync, as prompted by the IDE. (TODO is this necessary here, not later?)
7. At the very beginning of the `build.gradle.kts` file in the `shared` directory, add the following lines to the
   `plugins` block:

   ```kotlin
       plugins {
       // ...
       alias(libs.plugins.kotlinxSerialization)
       alias(libs.plugins.sqldelight)
   }
   ```

8. In the same `build.gradle.kts` file, specify all the required dependencies:

    ```kotlin
    kotlin {
        // ...
   
        sourceSets {
            commonMain.dependencies {
                implementation(libs.kotlinx.coroutines.core)
                implementation(libs.ktor.client.core)
                implementation(libs.ktor.client.content.negotiation)
                implementation(libs.ktor.serialization.kotlinx.json)
                implementation(libs.runtime)
                implementation(libs.kotlinx.datetime)
                implementation(libs.koin.core)
            }
            androidMain.dependencies {
                implementation(libs.ktor.client.android)
                implementation(libs.android.driver)
            }
            iosMain.dependencies {
                implementation(libs.ktor.client.darwin)
                implementation(libs.native.driver)
            }
        }
    }
    ```

   * Each library requires a core artifact in the common source set.
   * Both the SQLDelight and Ktor libraries need platform drivers in the iOS and Android source sets, as well.
   * Ktor also needs the [serialization feature](https://ktor.io/docs/serialization-client.html) to use
     `kotlinx.serialization` for processing network requests and responses.

9. Once the dependencies are added, you're prompted to resync the project. Click **Sync Now** to synchronize Gradle files:
   (TODO if this also needs to be done after we're done with lib.versions.toml, put these instructions there.)
   ![Synchronize Gradle files](gradle-sync.png)

After the Gradle Sync is complete, you are done with project configuration and can start writing code.

> For an in-depth guide on multiplatform dependencies, see [Dependency on Kotlin Multiplatform libraries](https://kotlinlang.org/docs/multiplatform-add-dependencies.html).
>
{type="tip"}

## Create an application data model

The Kotlin Multiplatform app will contain the public `SpaceXSDK` class, the facade over networking and cache services.
The application data model will have three entity classes with:

* General information about the launch
* Patches and article URLs associated with the launch

Create the necessary data classes:
1. In the `shared/src/commonMain/kotlin` directory, create the `com.jetbrains.spacetutorial` directory.
2. Inside, create the `entity` directory, and the `Entity.kt` file inside that.
3. Declare all the data classes for basic entities:

   ```kotlin
   ```
   {src="multiplatform-tutorial/Entity.kt" initial-collapse-state="collapsed" collapsed-title="data class RocketLaunch" lines="3-41" }

Each serializable class must be marked with the `@Serializable` annotation. The `kotlinx.serialization` plugin
automatically generates a default serializer for `@Serializable` classes unless you explicitly pass a link to a
serializer in the annotation argument.

However, you don't need to do that in this case. (TODO what don't you need to do?) The `@SerialName` annotation allows you to redefine field names, which helps
to declare properties in data classes with more easily readable names.

## Configure SQLDelight and implement cache logic

### Configure SQLDelight

The SQLDelight library allows you to generate a type-safe Kotlin database API from SQL queries. During compilation, the
generator validates the SQL queries and turns them into Kotlin code that can be used in the shared module.

The SQLDelight dependency is already included in the project. To configure the library, open the `shared/build.gradle.kts` file
and add the `sqldelight` block in the end. This block contains a list of databases and their parameters:

```kotlin
sqldelight {
   databases {
      create("AppDatabase") {
         packageName.set("com.jetbrains.spacetutorial.cache")
      }
   }
}
```

The `packageName` parameter specifies the package name for the generated Kotlin sources.

Sync the Gradle project files when prompted.

> Consider installing the official [SQLDelight plugin](https://plugins.jetbrains.com/plugin/8191-sqldelight)
> to work with `.sq` files.
>
{type="tip"}

### Generate the database API

First, create the `.sq` file with all the necessary SQL queries. By default, the SQLDelight plugin looks for
`.sq` files in the `sqldelight` folder of the source set:

1. In the `shared/src/commonMain` directory, create a new `sqldelight` directory.
2. Right-click the `sqldelight` directory and select **New | Directory**, then select
   the `com/jetbrains/spacetutorial/cache` option in the suggested list. This will create the `com.jetbrains.spacetutorial.cache`
   package.
3. Inside the package, create the `AppDatabase.sq` file (with the same name as the database you specified
   in the `build.gradle.kts` file).
   All the SQL queries for your application will be stored in this file.
4. The database will contain a table with data about launches. To create the table, add the following
   code to the `AppDatabase.sq` file:

   ```text
   import kotlin.Boolean;
   
   CREATE TABLE Launch (
       flightNumber INTEGER NOT NULL,
       missionName TEXT NOT NULL,
       details TEXT,
       launchSuccess INTEGER AS Boolean DEFAULT NULL,
       launchDateUTC TEXT NOT NULL,
       patchUrlSmall TEXT,
       patchUrlLarge TEXT,
       articleUrl TEXT
   );
   ```

5. To insert data into the tables, add the `insertLaunch` function:

   ```text
   insertLaunch:
   INSERT INTO Launch(flightNumber, missionName, details, launchSuccess, launchDateUTC, patchUrlSmall, patchUrlLarge, articleUrl)
   VALUES(?, ?, ?, ?, ?, ?, ?, ?);
   ```

6. To clear data in the tables, add the `removeAllLaunches` function:

   ```text
   removeAllLaunches:
   DELETE FROM Launch;
   ```

7. Finally, declare the `selectAllLaunchesInfo` function to retrieve data:

   ```text
   selectAllLaunchesInfo:
   SELECT Launch.*
   FROM Launch;
   ```

(TODO: we need to actually run the generation, or the Database class later won't be able to access the code.)
The generated Kotlin code is stored in the `shared/build/generated/sqldelight` directory.
The important part of this code is the interface named `AppDatabase` which you will initialize with database drivers later on.


### Create factories for platform-specific database drivers

To initialize the `AppDatabase` interface, you need to pass an `SqlDriver` instance to it. SQLDelight provides multiple platform-specific
implementations of the SQLite driver, so you need to create them for each platform separately. In this project, we will use
[Koin](https://insert-koin.io/), the dependency injection framework. (TODO explain what we get from this: no expect/actual stuff)

1. Create an interface for database drivers. To do this, in the `shared/src/commonMain/kotlin` directory, create
   the `com.jetbrains.spacetutorial` directory.
2. In the `com.jetbrains.spacetutorial` directory, create the `cache` directory, and then the `DatabaseDriverFactory` interface inside it:

   ```kotlin
   package com.jetbrains.spacetutorial.cache
   
   import app.cash.sqldelight.db.SqlDriver

   interface DatabaseDriverFactory {
       fun createDriver(): SqlDriver
   }
   ```

3. Create the class implementing this interface for Android: in the `shared/src/androidMain/kotlin` directory,
   create the `com.jetbrains.spacetutorial.cache` package with the `DatabaseDriverFactory.kt` file inside it.
4. On Android, the SQLite driver is implemented by the `AndroidSqliteDriver` class. In the `DatabaseDriverFactory.kt` file,
   pass the database information and the context link to the `AndroidSqliteDriver` class constructor:

   ```kotlin
   import android.content.Context
   import app.cash.sqldelight.db.SqlDriver
   import app.cash.sqldelight.driver.android.AndroidSqliteDriver

   class AndroidDatabaseDriverFactory(private val context: Context) : DatabaseDriverFactory {
       override fun createDriver(): SqlDriver {
           return AndroidSqliteDriver(AppDatabase.Schema, context, "launch.db")
       }
   }
   ```

5. For iOS, in the `shared/src/iosMain/kotlin` directory, create the `com.jetbrains.spacetutorial.cache` directory.
6. Inside this directory, create the `DatabaseDriverFactory.kt` file and add this code:

   ```kotlin
   import app.cash.sqldelight.db.SqlDriver
   import app.cash.sqldelight.driver.native.NativeSqliteDriver

   class IOSDatabaseDriverFactory : DatabaseDriverFactory {
       override fun createDriver(): SqlDriver {
           return NativeSqliteDriver(AppDatabase.Schema, "launch.db")
       }
   }
   ```

You will program creating instances of these drivers later, in the platform-specific code of your project.

### Implement cache

So far, you have added factories for platform database drivers and an `AppDatabase` class to perform database operations.
Now create a `Database` class, which will wrap the `AppDatabase` class and contain the caching logic.

1. In the common source set `shared/src/commonMain/kotlin`, create a new `Database` class in
   the `com.jetbrains.tutorial.cache` directory. It will contain logic common to both platforms.

2. To provide a driver for `AppDatabase`, pass an abstract `DatabaseDriverFactory` to the `Database` class constructor:

   ```kotlin
   package com.jetbrains.spacetutorial.cache

   internal class Database(databaseDriverFactory: DatabaseDriverFactory) {
       private val database = AppDatabase(databaseDriverFactory.createDriver())
       private val dbQuery = database.appDatabaseQueries
   }
   ```

   This class's [visibility](https://kotlinlang.org/docs/visibility-modifiers.html#class-members) is set to internal, which means it is only
   accessible from within the multiplatform module.

3. Inside the `Database` class, implement some data handling operations.
   First, create the `getAllLaunches` function to return a list of all the rocket launches.
   The `mapLaunchSelecting` function is used to map the result of the database query to `RocketLaunch` objects:

   ```kotlin
   import com.jetbrains.handson.kmm.shared.entity.Links
   import com.jetbrains.handson.kmm.shared.entity.Patch
   import com.jetbrains.handson.kmm.shared.entity.RocketLaunch

   internal fun getAllLaunches(): List<RocketLaunch> {
       return dbQuery.selectAllLaunchesInfo(::mapLaunchSelecting).executeAsList()
   }

   private fun mapLaunchSelecting(
       flightNumber: Long,
       missionName: String,
       details: String?,
       launchSuccess: Boolean?,
       launchDateUTC: String,
       patchUrlSmall: String?,
       patchUrlLarge: String?,
       articleUrl: String?
   ): RocketLaunch {
       return RocketLaunch(
           flightNumber = flightNumber.toInt(),
           missionName = missionName,
           details = details,
           launchDateUTC = launchDateUTC,
           launchSuccess = launchSuccess,
           links = Links(
               patch = Patch(
                   small = patchUrlSmall,
                   large = patchUrlLarge
               ),
               article = articleUrl
           )
       )
   }
   ```

4. Add the `clearAndCreateLaunches` function to clear the database and insert the data:

    ```kotlin
    internal fun clearAndCreateLaunches(launches: List<RocketLaunch>) {
        dbQuery.transaction {
            dbQuery.removeAllLaunches()
            launches.forEach { launch ->
                dbQuery.insertLaunch(
                    flightNumber = launch.flightNumber.toLong(),
                    missionName = launch.missionName,
                    details = launch.details,
                    launchSuccess = launch.launchSuccess ?: false,
                    launchDateUTC = launch.launchDateUTC,
                    patchUrlSmall = launch.links.patch?.small,
                    patchUrlLarge = launch.links.patch?.large,
                    articleUrl = launch.links.article
                )
            }
        }
    }
    ```

## Implement an API service

To retrieve data over the internet, you'll need the [SpaceX public API](https://github.com/r-spacex/SpaceX-API/tree/master/docs#rspacex-api-docs)
and a single method to retrieve the list of all launches from the `v5/launches` endpoint.

Create a class that will connect the application to the API:

1. In the common source set `shared/src/commonMain/kotlin`, create the `com.jetbrains.spacetutorial.network`
   package and the `SpaceXApi` class inside it:

   ```kotlin
   import io.ktor.client.HttpClient
   import io.ktor.client.plugins.contentnegotiation.ContentNegotiation
   import io.ktor.serialization.kotlinx.json.json
   import kotlinx.serialization.json.Json

   class SpaceXApi {
       private val httpClient = HttpClient {
           install(ContentNegotiation) {
               json(Json {
                   ignoreUnknownKeys = true
                   useAlternativeNames = false
               })
           }
       }
   }
   ```

   * This class executes network requests and deserializes JSON responses into entities from the `com.jetbrains.spacetutorial.entity` package.
     The Ktor `HttpClient` instance initializes and stores the `httpClient` property.
   * This code uses the [Ktor `ContentNegotiation` plugin](https://ktor.io/docs/serialization-client.html)
     to deserialize the result of a `GET` request. The plugin processes the request and the response payload as JSON,
     serializing and deserializing them as needed.

2. Declare the data retrieval function that returns the list of rocket launches:

   ```kotlin
   suspend fun getAllLaunches(): List<RocketLaunch> {
       return httpClient.get("https://api.spacexdata.com/v5/launches").body()
   }
   ```

   * The `getAllLaunches` function has the `suspend` modifier because it contains a call of the suspend function `HttpClient.get()`,
     The `get()` function includes an asynchronous operation to retrieve data over the internet and can only be called from a
     coroutine or another suspend function. The network request will be executed in the HTTP client's thread pool.
   * The URL to send a GET request to is passed as an argument to the `get()` function.

## Build an SDK

Your iOS and Android applications will communicate with the SpaceX API through the shared module, which will provide a
public class.

1. In the `com.jetbrains.spacetutorial` directory of the common source set, create the `SpaceXSDK` class.
   This class will be the facade for the `Database` and `SpaceXApi` classes.

   To create a `Database` class instance, you'll need to provide a `DatabaseDriverFactory` instance:

   ```kotlin
   package com.jetbrains.spacetutorial
   
   import com.jetbrains.spacetutorial.cache.Database
   import com.jetbrains.spacetutorial.cache.DatabaseDriverFactory
   import com.jetbrains.spacetutorial.network.SpaceXApi

   class SpaceXSDK(databaseDriverFactory: DatabaseDriverFactory, val api: SpaceXApi) { 
       private val database = Database(databaseDriverFactory)
   }
   ```

   You will inject the correct database driver in the platform-specific code through the `SpaceXSDK` class constructor.

2. Add the `getLaunches` function, which makes use of the created database and the API to get the launches list:

   ```kotlin
    import com.jetbrains.spacetutorial.entity.RocketLaunch
    
    class SpaceXSDK(databaseDriverFactory: DatabaseDriverFactory, val api: SpaceXApi) {
        // ...
        @Throws(Exception::class)
        suspend fun getLaunches(forceReload: Boolean): List<RocketLaunch> {
            val cachedLaunches = database.getAllLaunches()
            return if (cachedLaunches.isNotEmpty() && !forceReload) {
                cachedLaunches
            } else {
                api.getAllLaunches().also {
                    database.clearAndCreateLaunches(it)
                }
            }
        }
   }
   ```

To sum up:

* The class contains one function for getting all launch information. Depending on the value of `forceReload`, it
  returns cached values or loads the data from the internet and then updates the cache with the results. If there is
  no cached data, it loads the data from the internet independently of the `forceReload` flag's value.
* Clients of your SDK could use a `forceReload` flag to load the latest information about the launches, which would
  allow the user to use the pull-to-refresh gesture.
* All Kotlin exceptions are unchecked, while Swift has only checked errors. Thus, to make your Swift code aware of expected
  exceptions, Kotlin functions (TODO should it be "called from Swift" here? clearly not ALL Kotlin functions are marked) should be marked with the `@Throws` annotation specifying a list of potential exception
  classes.

## Create the Android application

The Kotlin Multiplatform wizard has already handled the configuration for you, so the `shared` module is already connected
to your Android application.

Before implementing the UI and the presentation logic, add all the required dependencies to
the `composeApp/build.gradle.kts` file:

```kotlin
kotlin {
// ...
   sourceSets {
      androidMain.dependencies {
         implementation(libs.androidx.compose.material3)
         implementation(libs.koin.androidx.compose)
         implementation(libs.androidx.lifecycle.viewmodel.compose)
      }
      // ...
   }
}
```

Sync the Gradle project files when prompted.

### Add internet access permission

To access the internet, an Android application needs the appropriate permission. All network requests are made
from the shared module, so it makes sense to add the internet access permission to this module's manifest.

In the `composeApp/src/androidMain/AndroidManifest.xml` file, add the `<uses-permission>` line:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
   <uses-permission android:name="android.permission.INTERNET" />
   <!--...-->
</manifest>
```

### Add dependency injection

You need to declare a Koin module that will contain all of the components to be used for the Android app. Then you will
start Koin with that module from your instance of the `Application` class.

1. In the `composeApp/src/androidMain/kotlin` directory, create the `AppModule.kt` file in the `com.jetbrains.spacetutorial` package.

   In that file, declare the module as two singletons, for the `SpaceXApi` class and for the `SpaceXSDK` class:

   ```kotlin
   import com.jetbrains.spacetutorial.cache.AndroidDatabaseDriverFactory
   import com.jetbrains.spacetutorial.network.SpaceXApi
   import org.koin.android.ext.koin.androidContext
   import org.koin.dsl.module
   
   val appModule = module { 
       single<SpaceXApi> { SpaceXApi() }
       single<SpaceXSDK> {
           SpaceXSDK(
               databaseDriverFactory = AndroidDatabaseDriverFactory(
                   androidContext()
               ), api = get()
           )
       }
   }
   ```

   The `SpaceXSDK` class constructor is injected with the platform-specific `AndroidDatabaseDriverFactory` class.

2. Create a custom `Application` class which will start the Koin module. Next to the `AppModule.kt` file,
   create the `Application.kt` file with the following code, specifying the module you declared in the `modules()` function call:

   ```kotlin
   import android.app.Application
   import org.koin.android.ext.koin.androidContext
   import org.koin.core.context.GlobalContext.startKoin
    
   class MainApplication : Application() {
       override fun onCreate() {
           super.onCreate()
    
           startKoin {
               androidContext(this@MainApplication)
               modules(appModule)
           }
       }
   }
   ```

3. Specify the custom `Application` class you created in the `<application>` tag of you `AndroidManifest.xml` file:

   ```xml
   <!--...-->
   <application
       ...
       android:name="com.jetbrains.spacetutorial.MainApplication">
   ```

Now you are ready to implement the UI that will use information provided by the platform-specific database driver.


### Implement the UI: display the list of rocket launches

You will implement the Android UI using Jetpack Compose and Material 3. First, you'll create the view model that makes
use of the SDK to get the list of launches. Then you'll set up the Material theme, and finally, write the composable
function that brings it all together.

1. Create the `RocketLaunchViewModel.kt` file in the `com.jetbrains.spacetutorial` package of your `composeApp` module:

   ```kotlin
   import androidx.compose.runtime.State
   import androidx.compose.runtime.mutableStateOf
   import androidx.lifecycle.ViewModel
   import com.jetbrains.spacetutorial.entity.RocketLaunch
    
   class RocketLaunchViewModel(private val sdk: SpaceXSDK) : ViewModel() {
       private val _state = mutableStateOf(RocketLaunchScreenState())
       val state: State<RocketLaunchScreenState> = _state
    
   }
    
   data class RocketLaunchScreenState(
       val isLoading: Boolean = false,
       val launches: List<RocketLaunch> = emptyList()
   )
   ```

   A `RocketLaunchScreenState` instance will store data received from the SDK and the current state of the request.

2. Add the `loadLaunches` function that will call the `getLaunches` function of the SDK in a coroutine scope of this view model:

   ```kotlin
   import androidx.lifecycle.viewModelScope
   import kotlinx.coroutines.launch
   
   class RocketLaunchViewModel(private val sdk: SpaceXSDK) : ViewModel() {
   //...
       fun loadLaunches() {
           viewModelScope.launch { 
               _state.value = _state.value.copy(isLoading = true, launches = emptyList())
               try {
                   val launches = sdk.getLaunches(forceReload = true)
                   _state.value = _state.value.copy(isLoading = false, launches = launches)
               } catch (e: Exception) {
                   _state.value = _state.value.copy(isLoading = false, launches = emptyList())
               }
           }
       }
   }
   ```
   
3. Then add a `loadLaunches()` call to the `init{}` block of the class:

    ```kotlin
    init {
        loadLaunches()
    }
    ```
   
   Your view model is ready to go. Now, add a Material Theme.

4. You can generate a color theme for your Compose app
   using the [Material Theme Builder](https://m3.material.io/theme-builder#/custom).
   When you're done picking colors, click **Export** in the top right corner and select the **Jetpack Compose (Theme.kt)**
   option.
5. Unpack the archive and copy the `theme` folder into the `composeApp/src/androidMain/kotlin/com/jetbrains/spacetutorial`
   directory.
6. Finally, create the `App.kt` file in the `com.jetbrains.spacetutorial` package and add the `App()` composable function:

    ```kotlin
    import androidx.compose.material3.ExperimentalMaterial3Api
    import androidx.compose.material3.pulltorefresh.rememberPullToRefreshState
    import androidx.compose.runtime.Composable
    import androidx.compose.runtime.getValue
    import androidx.compose.runtime.remember
    import org.koin.androidx.compose.koinViewModel
    
    @OptIn(
        ExperimentalMaterial3Api::class
    )
    @Composable
    fun App() {
        val viewModel = koinViewModel<RocketLaunchViewModel>()
        val state by remember { viewModel.state }
        val pullToRefreshState = rememberPullToRefreshState()
        if (pullToRefreshState.isRefreshing) {
            viewModel.loadLaunches()
            pullToRefreshState.endRefresh()
        }
    }
    ```
   
   The `viewModel` field instantiates the Koin-based view model which you created earlier.
11. 
12. To implement the UI, create the `layout/activity_main.xml` file in `composeApp/src/androidMain/res`.

   The screen is based on the `ConstraintLayout` with the `SwipeRefreshLayout` inside it, which contains `RecyclerView`
   and `FrameLayout` with a background with a `ProgressBar` across its center:

8. In `composeApp/src/androidMain/kotlin/kmp.project.demo`, replace the implementation of the `MainActivity` class, adding the properties for the
   UI elements:

   ```kotlin
   class MainActivity : AppCompatActivity() {
       private lateinit var launchesRecyclerView: RecyclerView
       private lateinit var progressBarView: FrameLayout
       private lateinit var swipeRefreshLayout: SwipeRefreshLayout
   
       override fun onCreate(savedInstanceState: Bundle?) {
           super.onCreate(savedInstanceState)
   
           title = "SpaceX Launches"
           setContentView(R.layout.activity_main)
   
           launchesRecyclerView = findViewById(R.id.launchesListRv)
           progressBarView = findViewById(R.id.progressBar)
           swipeRefreshLayout = findViewById(R.id.swipeContainer)
       }
   }
   ```

9. For the `RecyclerView` element to work, you need to create an adapter (as a subclass of `RecyclerView.Adapter`) that
   will convert raw data into list item views. To do this, create a separate `LaunchesRvAdapter` class:

   ```kotlin
   class LaunchesRvAdapter(var launches: List<RocketLaunch>) : RecyclerView.Adapter<LaunchesRvAdapter.LaunchViewHolder>() {
   
       override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): LaunchViewHolder {
           return LayoutInflater.from(parent.context)
               .inflate(R.layout.item_launch, parent, false)
               .run(::LaunchViewHolder)
       }
   
       override fun getItemCount(): Int = launches.count()
   
       override fun onBindViewHolder(holder: LaunchViewHolder, position: Int) {
           holder.bindData(launches[position])
       }
   
       inner class LaunchViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
           // ...
           fun bindData(launch: RocketLaunch) {
               // ...
           }
       }
   }
   ```

10. Create an `item_launch.xml` resource file in `composeApp/src/androidMain/res/layout` with the items view layout:

    ```xml
    ```
    {src="multiplatform-tutorial/item_launch.xml" initial-collapse-state="collapsed" collapsed-title="androidx.cardview.widget.CardView xmlns:android" lines="1-28"}

11. In `composeApp/src/androidMain/res/values`, either create your appearance of the app or copy the following styles:

    <tabs>
    <tab title="colors.xml">

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <resources>
        <color name="colorPrimary">#37474f</color>
        <color name="colorPrimaryDark">#102027</color>
        <color name="colorAccent">#62727b</color>
   
        <color name="colorSuccessful">#4BB543</color>
        <color name="colorUnsuccessful">#FC100D</color>
        <color name="colorNoData">#615F5F</color>
    </resources>
    ```

    </tab>
    <tab title="strings.xml">

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <resources>
        <string name="app_name">SpaceLaunches</string>
   
        <string name="successful">Successful</string>
        <string name="unsuccessful">Unsuccessful</string>
        <string name="no_data">No data</string>
   
        <string name="launch_year_field">Launch year: %s</string>
        <string name="mission_name_field">Launch name: %s</string>
        <string name="launch_success_field">Launch success: %s</string>
        <string name="details_field">Launch details: %s</string>
    </resources>
    ```

    </tab>
    <tab title="styles.xml">

    ```xml
    <resources>
        <!-- Base application theme. -->
        <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
            <!-- Customize your theme here. -->
            <item name="colorPrimary">@color/colorPrimary</item>
            <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
            <item name="colorAccent">@color/colorAccent</item>
        </style>
    </resources>
    ```

    </tab>
    </tabs>

12. Complete the implementation of the `RecyclerView.Adapter`:

    ```kotlin
    class LaunchesRvAdapter(var launches: List<RocketLaunch>) : RecyclerView.Adapter<LaunchesRvAdapter.LaunchViewHolder>() {
        // ...
        inner class LaunchViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
            private val missionNameTextView = itemView.findViewById<TextView>(R.id.missionName)
            private val launchYearTextView = itemView.findViewById<TextView>(R.id.launchYear)
            private val launchSuccessTextView = itemView.findViewById<TextView>(R.id.launchSuccess)
            private val missionDetailsTextView = itemView.findViewById<TextView>(R.id.details)

            fun bindData(launch: RocketLaunch) {
                val ctx = itemView.context
                missionNameTextView.text = ctx.getString(R.string.mission_name_field, launch.missionName)
                launchYearTextView.text = ctx.getString(R.string.launch_year_field, launch.launchYear.toString())
                missionDetailsTextView.text = ctx.getString(R.string.details_field, launch.details ?: "")
                val launchSuccess = launch.launchSuccess
                if (launchSuccess != null ) {
                    if (launchSuccess) {
                        launchSuccessTextView.text = ctx.getString(R.string.successful)
                        launchSuccessTextView.setTextColor((ContextCompat.getColor(itemView.context, R.color.colorSuccessful)))
                    } else {
                        launchSuccessTextView.text = ctx.getString(R.string.unsuccessful)
                        launchSuccessTextView.setTextColor((ContextCompat.getColor(itemView.context, R.color.colorUnsuccessful)))
                    }
                } else {
                    launchSuccessTextView.text = ctx.getString(R.string.no_data)
                    launchSuccessTextView.setTextColor((ContextCompat.getColor(itemView.context, R.color.colorNoData)))
                }
            }
        }
    }
    ```

13. Update the `MainActivity` class as follows:

    ```kotlin
    class MainActivity : AppCompatActivity() {
        // ...
        private val launchesRvAdapter = LaunchesRvAdapter(listOf())
   
        override fun onCreate(savedInstanceState: Bundle?) {
            // ...
            launchesRecyclerView.adapter = launchesRvAdapter
            launchesRecyclerView.layoutManager = LinearLayoutManager(this)
   
            swipeRefreshLayout.setOnRefreshListener {
                swipeRefreshLayout.isRefreshing = false
                displayLaunches(true)
            }
   
            displayLaunches(false)
        }
   
        private fun displayLaunches(needReload: Boolean) {
            // TODO: Presentation logic
        }
    }
    ```

    Here you create an instance of `LaunchesRvAdapter`, configure the `RecyclerView` component, and implement all the
    `LaunchesListView` interface functions. To catch the screen refresh gesture, you add a listener to the `SwipeRefreshLayout`.

### Implement the presentation logic

1. Create an instance of the `SpaceXSDK` class from the shared module and inject an instance of `DatabaseDriverFactory` in
   it:

   ```kotlin
   class MainActivity : AppCompatActivity() {
       // ...
       private val sdk = SpaceXSDK(DatabaseDriverFactory(this))
   }
   ```

2. Implement the private function `displayLaunches(needReload: Boolean)`. It runs the `getLaunches()` function inside
   the coroutine launched in the main `CoroutineScope`, handles exceptions, and displays the error text in the toast message:

   ```kotlin
   class MainActivity : AppCompatActivity() {
       private val mainScope = MainScope()
       // ...
       override fun onDestroy() {
           super.onDestroy()
           mainScope.cancel()
       }
       // ...
       private fun displayLaunches(needReload: Boolean) {
           progressBarView.isVisible = true
           mainScope.launch {
               kotlin.runCatching {
                   sdk.getLaunches(needReload)
               }.onSuccess {
                   launchesRvAdapter.launches = it
                   launchesRvAdapter.notifyDataSetChanged()
               }.onFailure {
                   Toast.makeText(this@MainActivity, it.localizedMessage, Toast.LENGTH_SHORT).show()
               }
               progressBarView.isVisible = false
           }
       }
   }
   ```

3. Select **composeApp** from the run configurations menu, choose an emulator, and click the run button:

![Android application](android-application.png){width=350}

You've just created an Android application that has its business logic implemented in the Kotlin Multiplatform module.

## Create the iOS application

For the iOS part of the project, you'll make use of [SwiftUI](https://developer.apple.com/xcode/swiftui/) to build the user
interface and the "Model View View-Model" pattern to connect the UI to the shared module, which contains all the business logic.

The shared module is already connected to the iOS project because the Android Studio plugin wizard has done all the configuration.
You can import it the same way you would regular iOS dependencies: `import shared`.

### Implement the UI

First, you'll create a `RocketLaunchRow` SwiftUI view for displaying an item from the list. It will be based on the `HStack`
and `VStack` views. There will be extensions on the `RocketLaunchRow` structure with useful helpers for displaying the
data.

1. Launch your Xcode app and select **Open Existing Project**.
2. Navigate to your project and select the `iosApp` folder. Click **Open**.
3. In your Xcode project, create a new Swift file with the type **SwiftUI View**, name it `RocketLaunchRow`, and update
   it with the following code:

   ```swift
   import SwiftUI
   import Shared
   
   struct RocketLaunchRow: View {
       var rocketLaunch: RocketLaunch
   
       var body: some View {
           HStack() {
               VStack(alignment: .leading, spacing: 10.0) {
                   Text("Launch name: \(rocketLaunch.missionName)")
                   Text(launchText).foregroundColor(launchColor)
                   Text("Launch year: \(String(rocketLaunch.launchYear))")
                   Text("Launch details: \(rocketLaunch.details ?? "")")
               }
               Spacer()
           }
       }
   }
   
   extension RocketLaunchRow {
       private var launchText: String {
           if let isSuccess = rocketLaunch.launchSuccess {
               return isSuccess.boolValue ? "Successful" : "Unsuccessful"
           } else {
               return "No data"
           }
       }
   
       private var launchColor: Color {
           if let isSuccess = rocketLaunch.launchSuccess {
               return isSuccess.boolValue ? Color.green : Color.red
           } else {
               return Color.gray
           }
       }
   }
   ```

   The list of launches will be displayed in the `ContentView`, which the project wizard has already created.

4. Create a `ViewModel` class for the `ContentView`, which will prepare and manage the data. Declare it as an extension
   to the `ContentView`, as they are closely connected, and then add the following code to `ContentView.swift`:

   ```swift
   // ...
   extension ContentView {
       enum LoadableLaunches {
           case loading
           case result([RocketLaunch])
           case error(String)
       }
       
      @MainActor
      class ViewModel: ObservableObject {
          @Published var launches = LoadableLaunches.loading
      }
   }
   ```

   * The [Combine framework](https://developer.apple.com/documentation/combine) connects the view model (`ContentView.ViewModel`)
     with the view (`ContentView`).
   * `ContentView.ViewModel` is declared as an `ObservableObject` and `@Published` wrapper is used for the `launches`
     property, so the view model will emit signals whenever this property changes.

5. Implement the body of the `ContentView` file and display the list of launches:

   ```swift
   struct ContentView: View {
    @ObservedObject private(set) var viewModel: ViewModel
   
        var body: some View {
            NavigationView {
                listView()
                .navigationBarTitle("SpaceX Launches")
                .navigationBarItems(trailing:
                    Button("Reload") {
                        self.viewModel.loadLaunches(forceReload: true)
                })
            }
        }
   
        private func listView() -> AnyView {
            switch viewModel.launches {
            case .loading:
                return AnyView(Text("Loading...").multilineTextAlignment(.center))
            case .result(let launches):
                return AnyView(List(launches) { launch in
                    RocketLaunchRow(rocketLaunch: launch)
                })
            case .error(let description):
                return AnyView(Text(description).multilineTextAlignment(.center))
            }
        }
    }
   ```

   The `@ObservedObject` property wrapper is used to subscribe to the view model.

6. To make it compile, the `RocketLaunch` class needs to confirm the `Identifiable` protocol, as it is used as a
   parameter for initializing the `List` Swift UIView. The `RocketLaunch` class already has a property named `id`, so
   add the following to the bottom of `ContentView.swift`:

   ```Swift
   extension RocketLaunch: Identifiable { }
   ```

### Load the data

To retrieve the data about the rocket launches in the view model, you'll need an instance of `SpaceXSDK` from the Multiplatform
library.

1. In `ContentView.swift`, pass it in through the constructor:

   ```swift
   extension ContentView {
       // ...
       @MainActor
       class ViewModel: ObservableObject {
           let sdk: SpaceXSDK
           @Published var launches = LoadableLaunches.loading
   
           init(sdk: SpaceXSDK) {
               self.sdk = sdk
               self.loadLaunches(forceReload: false)
           }
   
           func loadLaunches(forceReload: Bool) {
               // TODO: retrieve data
           }
       }
   }
   ```

2. Call the `getLaunches()` function from the `SpaceXSDK` class and save the result in the `launches` property:

   ```Swift
   func loadLaunches(forceReload: Bool) {
       Task {
           do {
               self.launches = .loading
               let launches = try await sdk.getLaunches(forceReload: forceReload)
               self.launches = .result(launches)
           } catch {
               self.launches = .error(error.localizedDescription)
           }
       }
   }
   ```

   * When you compile a Kotlin module into an Apple framework, [suspending functions](https://kotlinlang.org/docs/whatsnew14.html#support-for-kotlin-s-suspending-functions-in-swift-and-objective-c)
     are available in it as Swift's `async`/`await` mechanism.
   * Since the `getLaunches` function is marked with the `@Throws(Exception::class)` annotation, any exceptions that are
     instances of the `Exception` class or its subclass will be propagated as `NSError`. Therefore, all such errors can
     be caught by the `loadLaunches()` function.

3. Go to the entry point of the app, `iOSApp.swift`, and initialize the SDK, view, and view model:

   ```swift
   import SwiftUI
   import Shared
   
   @main
   struct iOSApp: App {
       let sdk = SpaceXSDK(databaseDriverFactory: DatabaseDriverFactory())
       var body: some Scene {
           WindowGroup {
               ContentView(viewModel: .init(sdk: sdk))
           }
       }
   }
   ```

4. In Android Studio, switch to the **iosApp** configuration, choose an emulator, and run it to see the result:

![iOS Application](ios-application.png){width=350}

<!-- You can find the final version of the project [on the `final` branch](https://github.com/kotlin-hands-on/kmm-networking-and-data-storage/tree/final).
>
{type="note"} -->

## What's next?

This tutorial features some potentially resource-heavy operations, like parsing JSON and making requests to the database in
the main thread. To learn about how to write concurrent code and optimize your app,
see the [Coroutines guide](https://kotlinlang.org/docs/coroutines-guide.html).

You can also check out these additional learning materials:

* [Use the Ktor HTTP client in multiplatform projects](https://ktor.io/docs/http-client-engines.html#mpp-config)
* [Make your Android application work on iOS](multiplatform-integrate-in-existing-app.md)
* [Learn more about multiplatform project structure](https://kotlinlang.org/docs/multiplatform-discover-project.html).
