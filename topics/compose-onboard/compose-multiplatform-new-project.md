[//]: # (title: Simple timezone picker app)

<secondary-label ref="IntelliJ IDEA"/>
<secondary-label ref="Android Studio"/>

Let's build an application that implements a relatively complex shared UI using external dependencies
and images as multiplatform resources.

You'll create an application where users can select a country to see the time in the capital city of that country.
All the functionality of your Compose Multiplatform app will be implemented in common code using multiplatform libraries.
It'll load and display images within a dropdown menu and will use Compose events, styles, themes, modifiers, and layouts.

In this tutorial, you will:

1. Add and use a multiplatform library dependency.
2. Implement a common Compose UI for a country picker for all supported platforms: iOS, Android, desktop, and web.
3. Practice using Compose Hot Reload, a tool for quickly iterating on your UI without rebuilding the entire app.
4. Import and use images in the common UI as multiplatform resources.

At each stage, you can run the application on all supported platforms (iOS, Android, desktop, and web), or you can focus on the
specific platforms that best suit your needs.

> You can find the final state of the project in our [GitHub repository](https://github.com/kotlin-hands-on/get-started-with-cm/).
>
{style="note"}

## Create a project

Create a fresh project using the Kotlin Multiplatform project template in your IDE
(it is available if you have the [KMP IDE plugin installed](https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform)):

1. Select **File** | **New** | **Project**, then **Kotlin Multiplatform** in the list of template options.
2. To follow the tutorial exactly, name the project "ComposeDemo" and set **Project ID** to "compose.project.demo".
3. The tutorial covers all platforms, but you can select only the ones you're interested in.
   Select the option to **Share UI** where it is present.

The first import takes a couple of minutes.
After it is done, make sure that preflight checks are all green (**View | Tool Windows | Projects Environment Preflight Checks**).

## Add the time dependency

To work with timezones and time calculation, you'll use the [kotlin.time](https://kotlinlang.org/docs/time-measurement.html)
classes together with the [kotlinx-datetime](https://github.com/Kotlin/kotlinx-datetime)
library.

While `kotlin.time` is always available as part of the standard library,
`kotlinx-datetime` needs to be configured as an explicit dependency.
It is a multiplatform library, so you will use it only in common code and need to specify the dependency only once:

1. Open the `gradle/libs.versions.toml` file and add the `kotlinx-datetime` dependency to the [version catalog](https://docs.gradle.org/current/userguide/version_catalogs.html):

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

3. For the web target, timezone support requires the `js-joda` library.
   Add a reference to the `js-joda` npm package to the `webApp/build.gradle.kts file`:

    ```kotlin
    kotlin {
        // ...
        sourceSets {
            // ...
            webMain.dependencies {
                // ...
                implementation(npm("@js-joda/timezone", "%js-joda-timezone%"))
            }
        }
    }
    
    ```

   Adding the dependency to the `webMain` source set makes the library available both to the `wasmJs` and `js` targets.

4. Once the dependency is added, accept the IDE suggestion to sync the Gradle configuration
   or press double **Shift** and execute the **Sync Project with Gradle Files** command.

5. In the **Terminal** tool window, run the following command to update the `yarn.lock` file with the latest dependency versions:

    ```shell
    ./gradlew kotlinUpgradeYarnLock kotlinWasmUpgradeYarnLock
    ```

6. In the `webApp/src/webMain/kotlin/.../main.kt` file, use the `@JsModule` annotation to import the `js-joda` npm package:

    ```kotlin
    import androidx.compose.ui.ExperimentalComposeUiApi
    import androidx.compose.ui.window.ComposeViewport
    import kotlin.js.ExperimentalWasmJsInterop
    import kotlin.js.JsModule

    @OptIn(ExperimentalWasmJsInterop::class)
    @JsModule("@js-joda/timezone")
    external object JsJodaTimeZoneModule
    
    private val jsJodaTz = JsJodaTimeZoneModule
    
    @OptIn(ExperimentalComposeUiApi::class)
    fun main() {
        ComposeViewport {
            App()
        }
    }
    ```
   {initial-collapse-state="collapsed" collapsible="true" collapsed-title='@JsModule("@js-joda/timezone")'}

> When commiting your project to version control, include the `yarn.lock` files generated in the `kotlin-js-store` directory.
> This helps ensure that the same versions of JavaScript dependencies are used wherever the project is built.
>
{style="note"}

Now you can use `kotlinx-datetime` APIs in your common code.

> For more general information on how to manage multiplatform dependencies,
> see [](multiplatform-add-dependencies.md).
>
{style="tip"}

## Lay the foundation

UI in a template Compose Multiplatform project is organized in several app modules
and a shared UI module, feeding the main `App()` composable into the entry points defined in app modules.
In this tutorial, you are not doing anything that requires updating the platform-specific code:
all changes in the common UI code are seamlessly propagated across the apps.

To get started, implement the basic layout in the common `App()` composable:

1. In `shared/src/commonMain/kotlin`, open the `compose.project.demo/App.kt` file and replace the code
   with the following `App()` composable:

    ```kotlin
    @Composable
    @Preview
    fun App() {
        MaterialTheme {
            var timeAtLocation by remember { mutableStateOf("No location selected") }
   
            // The UI is a column that holds a Text and a Button
            Column(
                // Basic layout improvements that make sure that the Column()
                // fills all available space without overlapping the system bars
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
   
    > The `remember` API implements Compose-specific state management.
    > The state object is wrapped in a `remember()` call, meaning that it's built once and then
    > retained by the framework.
    > When the value of the state changes, any composables that observe it are re-invoked and redrawn.
    > This is called a _recomposition_. 
    >
    > For an in-depth introduction, see [Managing state](https://developer.android.com/develop/ui/compose/state)
    > in Jetpack Compose documentation.  

2. Follow the IDE's instructions to import the missing dependencies.

3. Run the application on Android and iOS:

   ![New Compose Multiplatform app on Android and iOS](first-compose-project-on-android-ios-3.png){width=500}

   When you run your application and click the button, the app displays the hardcoded time — 13:30.

4. Run the application on the desktop using [Compose Hot Reload](compose-hot-reload.md) by starting the "desktopApp [hot] 🔥"
   run configuration.
   The app works, but the window looks mismatched with the UI:

   ![New Compose Multiplatform app on desktop](first-compose-project-on-desktop-3.png){width=400}

## Use Compose Hot Reload to quickly iterate on UI

You can fix the desktop UI and verify the fix without rerunning the build:

1. Update the `main.kt` file under the `desktopApp/src/` directory as follows:

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
            // Makes sure that the window is always on top
            // to make debug and UI iteration easier
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

For simplicity, you won't implement a complicated logic of specifying and validating time zones.
The app will offer several countries to choose from and display the time in the capital of the country:

1. In `shared/src/commonMain/kotlin`, open the `compose.project.demo/App.kt` file and add the supporting code,
   a data class to hold country information and a `currentTimeAt()` function that calculates local time for a given zone:

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
    fun defaultCountries() = listOf(
      Country("Japan", TimeZone.of("Asia/Tokyo")),
      Country("France", TimeZone.of("Europe/Paris")),
      Country("Mexico", TimeZone.of("America/Mexico_City")),
      Country("Indonesia", TimeZone.of("Asia/Jakarta")),
      Country("Egypt", TimeZone.of("Africa/Cairo")),
    )
    ```

2. Replace the `App()` composable to account for this change.
   The list of countries is presented as a dropdown and time is calculated instead of hardcoded:

    ```kotlin
    // Now requires a list of countries to display in the dropdown menu
    @Composable
    @Preview
    fun App(countries: List<Country> = defaultCountries()) {
      MaterialTheme {
          var showCountries by remember { mutableStateOf(false) }
          var timeAtLocation by remember { mutableStateOf("No location selected") }
    
    
          // Composables receive .padding() modifiers to add some space
          // between controls and around them
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
                      defaultCountries.forEach { (name, zone) ->
                          DropdownMenuItem(
                              text = { Text(name) },
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
    {initial-collapse-state="collapsed" collapsible="true" collapsed-title="defaultCountries.forEach { (name, zone) ->"}
   
3. Follow the IDE's instructions to import the missing dependencies. When importing `Row()`, pick the `@Composable` version.

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

Compose Multiplatform provides a library for accessing resources through common code across all platforms.
The library is automatically added and configured along with Compose Multiplatform itself,
so you can start loading resources right away.

To support images in your project, download image files, store them in the correct directory,
and add code to load and display them:

1. Download flag images from [Flag CDN](https://flagcdn.com/) to match the list of countries
   you have already created. In this case, these
   are [Japan](https://flagcdn.com/w320/jp.png), [France](https://flagcdn.com/w320/fr.png), [Mexico](https://flagcdn.com/w320/mx.png), [Indonesia](https://flagcdn.com/w320/id.png),
   and [Egypt](https://flagcdn.com/w320/eg.png).

2. Move the images to the `shared/src/commonMain/composeResources/drawable` directory so that the same flags are available on all platforms:

   ![Compose Multiplatform resources project structure](compose-resources-project-structure.png){width=300}

3. Build or run the application to generate the `Res` class with accessors for the added resources.

4. Update the code in the `commonMain/kotlin/.../App.kt` file to support images:

    ```kotlin
    import composedemo.shared.generated.resources.Res
    import demo.shared.generated.resources.eg
    import demo.shared.generated.resources.fr
    import demo.shared.generated.resources.id
    import demo.shared.generated.resources.jp
    import demo.shared.generated.resources.mx
    
    // The type now also holds a reference to the flag image
    data class Country(val name: String, val zone: TimeZone, val image: DrawableResource)

    fun currentTimeAt(location: String, zone: TimeZone): String {
        fun LocalTime.formatted() = "$hour:$minute:$second"

        val time = Clock.System.now()
        val localTime = time.toLocalDateTime(zone).time

        return "The time in $location is ${localTime.formatted()}"
    }

    // The list is initialized with imported Compose Multiplatform resources
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
                        defaultCountries.forEach { (name, zone, image) ->
                            // Each country is displayed in a 'DropdownMenuItem'
                            // as a flag ('Image()') and a name ('Text()')
                            DropdownMenuItem(
                                text = { Row(verticalAlignment = Alignment.CenterVertically) {
                                    Image(
                                        // 'painterResource()' supplies the Painter object
                                        // required by 'Image()'
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

This tutorial covers the basic building blocks of a multiplatform project.
To dive deeper into specifics:
* Learn about the [fundamentals of Compose layouts](compose-layout.md) and [working with Compose modifiers](compose-layout-modifiers.md).
* Learn about [possibilities and challenges of multiplatform resources in Compose](compose-multiplatform-resources.md).
* For more information on how to manage multiplatform dependencies, see [](multiplatform-add-dependencies.md).
* Follow more complicated tutorials to take a look at samples which are closer to production applications:
  * [Share data and network logic using Ktor and SQLDelight](multiplatform-ktor-sqldelight.md)
  * [Migrate an advanced Android app to KMP](migrate-from-android.md)
* [See the curated list of sample multiplatform projects](multiplatform-samples.md)

Join the community:

* ![Slack](slack.svg){width=25}{type="joined"} **Kotlin Slack**: Get help and participate in discussions about KMP and Compose Multiplatform.
  Request an [invitation](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up) and join
  [#multiplatform](https://kotlinlang.slack.com/archives/C3PQML5NU)
  and [#compose](https://kotlinlang.slack.com/archives/CJLTWPH7S) channels.
* ![GitHub](git-hub.svg){width=25}{type="joined"} **Compose Multiplatform GitHub**: star [the repository](https://github.com/JetBrains/compose-multiplatform) and contribute
* ![Stack Overflow](stackoverflow.svg){width=25}{type="joined"} **Stack Overflow**: Subscribe to
  the ["kotlin-multiplatform" tag](https://stackoverflow.com/questions/tagged/kotlin-multiplatform)
* ![YouTube](youtube.svg){width=25}{type="joined"} **Kotlin YouTube channel**: Subscribe and watch videos
  about [Kotlin Multiplatform](https://www.youtube.com/playlist?list=PLlFc5cFwUnmy_oVc9YQzjasSNoAk4hk_C)