[//]: # (title: Simple timezone picker app)

<secondary-label ref="IntelliJ IDEA"/>
<secondary-label ref="Android Studio"/>

Let's build an application that implements a more complex shared UI using additional dependencies and images as multiplatform resources.
In this tutorial, you will:

1. Add and use the common `kotlinx-datetime` dependency.
2. Implement a common UI for a timezone picker for all supported platforms: iOS, Android, desktop, and web.
3. Import and use images in the common UI as multiplatform resources.

You'll create an application where users can enter their country and city, and the app will display the time
in the capital city of that country. All the functionality of your Compose Multiplatform app will be implemented in common
code using multiplatform libraries. It'll load and display images within a dropdown menu and will use events, styles, themes,
modifiers, and layouts.

At each stage, you can run the application on all supported platforms (iOS, Android, desktop, and web), or you can focus on the
specific platforms that best suit your needs.

> You can find the final state of the project in our [GitHub repository](https://github.com/kotlin-hands-on/get-started-with-cm/).
>
{style="note"}

## Create a project

Create the same project as in the [basic Compose Multiplatform tutorial](compose-multiplatform-create-first-app.md):
Targeting all platforms you're interested in, with the **Share UI** option selected for iOS and web.

## Add the time dependency

To work with timezones and time calculation, you'll use the [kotlin.time](https://kotlinlang.org/docs/time-measurement.html)
classes together with the [kotlinx-datetime](https://github.com/Kotlin/kotlinx-datetime)
library.

While `kotlin.time` is always available as part of the standard library,
`kotlinx-datetime` needs to be configured as an explicit dependency.
It is a multiplatform library, so you will use it only in common code and need to specify the dependency only once:

1. Open the `gradle/libs.versions.toml` file and add the `kotlinx-datetime` dependency to the version catalog:

    ```toml
    [versions]
    kotlinx-datetime = "%dateTimeVersion%"
    
    [libraries]
    kotlinx-datetime = { module = "org.jetbrains.kotlinx:kotlinx-datetime", version.ref = "kotlinx-datetime" }
    ```

2. Open the `shared/build.gradle.kts` file and add a reference to the version catalog entry
   to the section that configures the `commonMain` source set:

    ```kotlin
    kotlin {
        // ... 
        sourceSets {
            commonMain.dependencies {
                // ...
                implementation(libs.kotlinx.datetime)
            } 
        }
    }
    ```

3. Select the **Build | Sync Project with Gradle Files** menu item
   or click the **Sync Gradle Changes** ![Synchronize Gradle files](gradle-sync.png){width=50} button in the build script editor to synchronize Gradle files.

Now you can refer to `kotlinx-datetime` APIs in your common code.

## Lay the foundation

To get started, implement a new `App()` composable with the basic layout:

1. In `shared/src/commonMain/kotlin`, open the `App.kt` file and replace the code
   with the following `App()` composable:

    ```kotlin
    @Composable
    @Preview
    fun App() {
        MaterialTheme {
            var timeAtLocation by remember { mutableStateOf("No location selected") }
   
            // The UI is a column that holds a Text and a Button
            Column(
                modifier = Modifier
                    .safeContentPadding()
                    .fillMaxSize(),
            ) {
                // The Text composable observes the timeAtLocation state
                Text(timeAtLocation)
                // The Button composable also uses the timeAtLocation state
                // but shows a hardcoded time for now
                Button(onClick = { timeAtLocation = "13:30" }) {
                    Text("Show Time At Location")
                }
            }
        }
    }
    ```

2. Run the application on Android and iOS:

   ![New Compose Multiplatform app on Android and iOS](first-compose-project-on-android-ios-3.png){width=500}

   When you run your application and click the button, the app displays the hardcoded time — 13:30.

3. Run the application on the desktop using [Compose Hot Reload](compose-hot-reload.md) by starting the "🔥desktopApp"
   run configuration.
   The app works, but the window looks mismatched with the UI:

   ![New Compose Multiplatform app on desktop](first-compose-project-on-desktop-3.png){width=400}

## Use Compose Hot Reload to quickly iterate on UI

You can fix the desktop UI and verify the fix without rerunning the build:

1. Update the `main.kt` file in the `desktopApp/src/kotlin` directory as follows:

    ```kotlin
    fun main() = application {
        // Sets the initial size and position
        // of the window on screen
        val state = rememberWindowState(
            size = DpSize(400.dp, 350.dp),
            position = WindowPosition(300.dp, 300.dp)
        )
        // Sets the title of the application window
        // and uses the window state initialized above
        Window(
            title = "Local Time App", 
            onCloseRequest = ::exitApplication, 
            state = state,
            alwaysOnTop = true
        ) {
            App()
        }
    }
    ```

2. Follow the IDE's instructions to import the missing dependencies.

3. To see the app automatically update, save the modified files (<shortcut>⌘ S</shortcut> / <shortcut>Ctrl+S</shortcut>).
   The window should adjust:

   <!--![Smaller window of the Compose Multiplatform app on desktop](first-compose-project-on-desktop-4.png){width=350}-->

   ![Compose Hot Reload](compose-hot-reload-resize.gif)

## Support user input

Now let users enter the name of a city to see the time in the corresponding timezone.
A simple way to achieve this is to add a `TextField()` composable:

1. Replace the current implementation of `App()` in `shared/src/commonMain/kotlin/compose.project.demo/App.kt` with the one below:

    ```kotlin
    @Composable
    @Preview
    fun App() {
        MaterialTheme {
            // New property to hold the user input
            // with an example value
            var location by remember { mutableStateOf("Europe/Paris") }
            var timeAtLocation by remember { mutableStateOf("No location selected") }
    
            Column(
                modifier = Modifier
                    .safeContentPadding()
                    .fillMaxSize(),
            ) {
                Text(timeAtLocation)
                // The TextField composable displays the location property
                // and updates it as user continues to type
                TextField(value = location, onValueChange = { location = it })
                Button(onClick = { timeAtLocation = "13:30" }) {
                    Text("Show Time At Location")
                }
            }
        }
    }
    ```

2. Follow the IDE's suggestions to import the missing dependencies.
3. Run the application on each platform you're targeting.
   The time displayed is still hardcoded, but now you can enter a timezone in the text field.

<tabs>
    <tab id="mobile-user-input" title="Android and iOS">
        <img src="first-compose-project-on-android-ios-4.png" alt="User input in the Compose Multiplatform app on Android and iOS" width="500"/>
    </tab>
    <tab id="desktop-user-input" title="Desktop">
        <img src="first-compose-project-on-desktop-5.png" alt="User input in the Compose Multiplatform app on desktop" width="350"/>
    </tab>
    <tab id="web-user-input" title="Web">
        <img src="first-compose-project-on-web-3.png" alt="User input in the Compose Multiplatform app on the web" width="500"/>
    </tab>
</tabs>

## Calculate time

The next step is to match the user input with a timezone and calculate time.
To do this, create a `currentTimeAt()` function:

1. Return to the `shared/src/commonMain/kotlin/compose.project.demo/App.kt` file and add the following function:

    ```kotlin
   fun currentTimeAt(location: String): String? {
        fun LocalTime.formatted() = "$hour:$minute:$second"

        return try {
            val time = Clock.System.now()
            val zone = TimeZone.of(location)
            val localTime = time.toLocalDateTime(zone).time
            "The time in $location is ${localTime.formatted()}"
        } catch (ex: IllegalTimeZoneException) {
            null
        }
    }
    ```

2. Follow the IDE's instructions to import the missing dependencies.
   Make sure to import the `Clock` class from `kotlin.time`, not `kotlinx.datetime`.

3. Call the `currentTimeAt()` function from your `App()` composable
   instead of using the hardcoded value:

   <compare type="top-bottom">
   <code-block lang="kotlin">
   Button(onClick = { timeAtLocation = "13:30" }) {
       Text("Show Time At Location")
   }
   </code-block>
   <code-block lang="kotlin">
   Button(onClick = { timeAtLocation = currentTimeAt(location) ?: "Invalid Location" }) {
       Text("Show Time At Location")
   }
   </code-block>
   </compare>

4. Run the application again and enter a [valid timezone identifier](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones#List)
   (from the "TZ identifier" column).
5. Click the button. You should see the correct time:

<tabs>
    <tab id="mobile-time-display" title="Android and iOS">
        <img src="first-compose-project-on-android-ios-5.png" alt="Time display in the Compose Multiplatform app on Android and iOS" width="500"/>
    </tab>
    <tab id="desktop-time-display" title="Desktop">
        <img src="first-compose-project-on-desktop-6.png" alt="Time display in the Compose Multiplatform app on desktop" width="350"/>
    </tab>
    <tab id="web-time-display" title="Web">
        <img src="first-compose-project-on-web-4.png" alt="Time display in the Compose Multiplatform app on the web" width="500"/>
    </tab>
</tabs>

## Improve the style

The application is working, but the composables could be spaced better,
and the time message could be rendered more prominently.

1. To address these issues, use the following version of the `App` composable
   (see comments for specific adjustments):

    ```kotlin
    @Composable
    @Preview
    fun App() {
        MaterialTheme {
            var location by remember { mutableStateOf("Europe/Paris") }
            var timeAtLocation by remember { mutableStateOf("No location selected") }
   
            Column(
                modifier = Modifier
                    // Adds padding all around the Column
                    .padding(20.dp)
                    .safeContentPadding()
                    .fillMaxSize(),
            ) {
                Text(
                    timeAtLocation,
                    // Renders text with a larger font size
                    style = TextStyle(fontSize = 20.sp),
                )
                TextField(
                    value = location,
                    onValueChange = { location = it },
                    // Adds padding before the input field
                    modifier = Modifier.padding(top = 10.dp)
                )
                Button(
                    onClick = { timeAtLocation = currentTimeAt(location) ?: "Invalid Location" },
                    // Adds padding before the button
                    modifier = Modifier.padding(top = 10.dp)
                ) {
                    Text("Show Time")
                }
            }
        }
    }
    ```

2. Follow the IDE's instructions to import the missing dependencies.

3. Run each application to see how the appearance has changed simultaneously on each platform:

<tabs>
    <tab id="mobile-improved-style" title="Android and iOS">
        <img src="first-compose-project-on-android-ios-6.png" alt="Improved style of the Compose Multiplatform app on Android and iOS" width="500"/>
    </tab>
    <tab id="desktop-improved-style" title="Desktop">
        <img src="first-compose-project-on-desktop-7.png" alt="Improved style of the Compose Multiplatform app on desktop" width="350"/>
    </tab>
    <tab id="web-improved-style" title="Web">
        <img src="first-compose-project-on-web-5.png" alt="Improved style of the Compose Multiplatform app on the web" width="500"/>
    </tab>
</tabs>

## Refactor the input processing

The application works, but it's susceptible to typos.
For example, if a user enters "Franse" instead of "France",
the app won't be able to process that input.
It would be easier for users to pick countries from a predefined list.

To achieve this, update the `App()` composable and the `currentTimeAt()` function, adding an auxiliary data class:

 ```kotlin
// Simplified representation of timezones for this example 
data class Country(val name: String, val zone: TimeZone)
 
// Takes TimeZone as a parameter to calculate time with
fun currentTimeAt(location: String, zone: TimeZone): String {
  fun LocalTime.formatted() = "$hour:$minute:$second"

  val time = Clock.System.now()
  val localTime = time.toLocalDateTime(zone).time

  return "The time in $location is ${localTime.formatted()}"
}

// Defines a list of supported countries
// with specific associated timezones
fun countries() = listOf(
  Country("Japan", TimeZone.of("Asia/Tokyo")),
  Country("France", TimeZone.of("Europe/Paris")),
  Country("Mexico", TimeZone.of("America/Mexico_City")),
  Country("Indonesia", TimeZone.of("Asia/Jakarta")),
  Country("Egypt", TimeZone.of("Africa/Cairo")),
)

// Now requires a list of countries to display in the dropdown menu
@Composable
@Preview
fun App(countries: List<Country> = countries()) {
  MaterialTheme {
      var showCountries by remember { mutableStateOf(false) }
      var timeAtLocation by remember { mutableStateOf("No location selected") }

      Column(
          modifier = Modifier
              .padding(20.dp)
              .safeContentPadding()
              .fillMaxSize(),
      ) {
          Text(
              timeAtLocation,
              style = TextStyle(fontSize = 20.sp),
          )
          Row(modifier = Modifier.padding(start = 20.dp, top = 10.dp)) {
              DropdownMenu(
                  // Uses a remembered value to control
                  // the visibility of the  dropdown menu
                  expanded = showCountries,
                  onDismissRequest = { showCountries = false }
              ) {
                  // Creates a dropdown menu item for each country
                  countries().forEach { (name, zone) ->
                      DropdownMenuItem(
                          text = {   Text(name)},
                          onClick = {
                              timeAtLocation = currentTimeAt(name, zone)
                              showCountries = false
                          }
                      )
                  }
              }
          }

          Button(modifier = Modifier.padding(start = 20.dp, top = 10.dp),
              onClick = { showCountries = !showCountries }) {
              Text("Select Location")
          }
      }
  }
}
```

Follow the IDE's instructions to import the missing dependencies. When importing `Row()`, pick the `@Composable` version.

Run the application to see the redesigned version:

<tabs>
    <tab id="mobile-country-list" title="Android and iOS">
        <img src="first-compose-project-on-android-ios-7.png" alt="The country list in the Compose Multiplatform app on Android and iOS" width="500"/>
    </tab>
    <tab id="desktop-country-list" title="Desktop">
        <img src="first-compose-project-on-desktop-8.png" alt="The country list in the Compose Multiplatform app on desktop" width="350"/>
    </tab>
   <tab id="web-country-list" title="Web">
        <img src="first-compose-project-on-web-6.png" alt="The country list in the Compose Multiplatform app on the web" width="500"/>
    </tab>
</tabs>

> You can further improve the design using a dependency injection framework, such as [Koin](https://insert-koin.io/),
> to build and inject the table of locations. If the data is stored externally,
> you can use the [Ktor](https://ktor.io/docs/create-client.html) library to fetch it over the network or
> the [SQLDelight](https://github.com/cashapp/sqldelight) library to fetch it from a database.
>
{style="note"}

## Introduce images

The list of country names works, but it's not a great user experience.
You can improve the list by adding images of national flags next to country names.

Compose Multiplatform provides a library for accessing resources through common code across all platforms. The Kotlin
Multiplatform wizard has already added and configured this library, so you can start loading resources right away.

To support images in your project, you'll need to download image files, store them in the correct directory, and add
code to load and display them:

1. Download flag images from [Flag CDN](https://flagcdn.com/) to match the list of countries
   you have already created. In this case, these
   are [Japan](https://flagcdn.com/w320/jp.png), [France](https://flagcdn.com/w320/fr.png), [Mexico](https://flagcdn.com/w320/mx.png), [Indonesia](https://flagcdn.com/w320/id.png),
   and [Egypt](https://flagcdn.com/w320/eg.png).

2. Move the images to the `shared/src/commonMain/composeResources/drawable` directory so that the same flags are available on all platforms:

   ![Compose Multiplatform resources project structure](compose-resources-project-structure.png){width=300}

3. Build or run the application to generate the `Res` class with accessors for the added resources.

4. Update the code in the `commonMain/kotlin/.../App.kt` file to support images:

    ```kotlin
    import demo.shared.generated.resources.jp
    import demo.shared.generated.resources.mx
    import demo.shared.generated.resources.eg
    import demo.shared.generated.resources.fr
    import demo.shared.generated.resources.id
   
    data class Country(val name: String, val zone: TimeZone, val image: DrawableResource)

    fun currentTimeAt(location: String, zone: TimeZone): String {
        fun LocalTime.formatted() = "$hour:$minute:$second"

        val time = Clock.System.now()
        val localTime = time.toLocalDateTime(zone).time

        return "The time in $location is ${localTime.formatted()}"
    }

    val defaultCountries = listOf(
        Country("Japan", TimeZone.of("Asia/Tokyo"), Res.drawable.jp),
        Country("France", TimeZone.of("Europe/Paris"), Res.drawable.fr),
        Country("Mexico", TimeZone.of("America/Mexico_City"), Res.drawable.mx),
        Country("Indonesia", TimeZone.of("Asia/Jakarta"), Res.drawable.id),
        Country("Egypt", TimeZone.of("Africa/Cairo"), Res.drawable.eg)
    )

    @Composable
    @Preview
    fun App(countries: List<Country> = defaultCountries) {
        MaterialTheme {
            var showCountries by remember { mutableStateOf(false) }
            var timeAtLocation by remember { mutableStateOf("No location selected") }

            Column(
                modifier = Modifier
                    .padding(20.dp)
                    .safeContentPadding()
                    .fillMaxSize(),
            ) {
                Text(
                    timeAtLocation,
                    style = TextStyle(fontSize = 20.sp),
                )
                Row(modifier = Modifier.padding(start = 20.dp, top = 10.dp)) {
                    DropdownMenu(
                        expanded = showCountries,
                        onDismissRequest = { showCountries = false }
                    ) {
                        countries.forEach { (name, zone, image) ->
                            DropdownMenuItem(
                                text = { Row(verticalAlignment = Alignment.CenterVertically) {
                                    Image(
                                        painterResource(image),
                                        modifier = Modifier.size(50.dp).padding(end = 10.dp),
                                        contentDescription = "$name flag"
                                    )
                                    Text(name)
                                } },
                                onClick = {
                                    timeAtLocation = currentTimeAt(name, zone)
                                    showCountries = false
                                }
                            )
                        }
                    }
                }

                Button(modifier = Modifier.padding(start = 20.dp, top = 10.dp),
                    onClick = { showCountries = !showCountries }) {
                    Text("Select Location")
                }
            }
        }
    }
    ```
    {initial-collapse-state="collapsed" collapsible="true" collapsed-title="data class Country(val name: String, val zone: TimeZone, val image: DrawableResource)"}

    * The `Country` type stores the path to the associated image.
    * The list of countries passed to the `App` includes these paths.
    * The `App` displays an `Image` in each `DropdownMenuItem`, followed by a `Text` composable with the name of a country.
    * Each `Image` requires a `Painter` object to fetch the data.

5. Follow the IDE's instructions to import the missing dependencies.
6. Run the application to see the new behavior:

<tabs>
    <tab id="mobile-flags" title="Android and iOS">
        <img src="first-compose-project-on-android-ios-8.png" alt="The country flags in the Compose Multiplatform app on Android and iOS" width="500"/>
    </tab>
    <tab id="desktop-flags" title="Desktop">
        <img src="first-compose-project-on-desktop-9.png" alt="The country flags in the Compose Multiplatform app on desktop" width="350"/>
    </tab>
   <tab id="web-flags" title="Web">
        <img src="first-compose-project-on-web-7.png" alt="The country flags in the Compose Multiplatform app on the web" width="500"/>
    </tab>
</tabs>

> You can find the final state of the project in our [GitHub repository](https://github.com/kotlin-hands-on/get-started-with-cm/).
>
{style="note"}

## What's next

We encourage you to explore multiplatform development further and try out more projects:

* [Make your Android app cross-platform](multiplatform-integrate-in-existing-app.md)
* [Create a multiplatform app using Ktor and SQLDelight](multiplatform-ktor-sqldelight.md)
* [Share business logic between iOS and Android while keeping the UI native](multiplatform-create-first-app.md)
* [Create a Compose Multiplatform app with Kotlin/Wasm](https://kotlinlang.org/docs/wasm-get-started.html)
* [See the curated list of sample projects](multiplatform-samples.md)

Join the community:

* ![GitHub](git-hub.svg){width=25}{type="joined"} **Compose Multiplatform GitHub**: star [the repository](https://github.com/JetBrains/compose-multiplatform) and contribute
* ![Slack](slack.svg){width=25}{type="joined"} **Kotlin Slack**: Get
  an [invitation](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up) and join
  the [#multiplatform](https://kotlinlang.slack.com/archives/C3PQML5NU) channel
* ![Stack Overflow](stackoverflow.svg){width=25}{type="joined"} **Stack Overflow**: Subscribe to
  the ["kotlin-multiplatform" tag](https://stackoverflow.com/questions/tagged/kotlin-multiplatform)
* ![YouTube](youtube.svg){width=25}{type="joined"} **Kotlin YouTube channel**: Subscribe and watch videos
  about [Kotlin Multiplatform](https://www.youtube.com/playlist?list=PLlFc5cFwUnmy_oVc9YQzjasSNoAk4hk_C)