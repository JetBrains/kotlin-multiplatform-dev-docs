[//]: # (title: Create your own application)
<microformat>
   <p>This is the fourth part of the <strong>Getting started with Compose Multiplatform</strong> tutorial. Before proceeding, make sure you've completed previous steps.</p>
   <p><img src="icon-1-done.svg" width="20" alt="First step"/> <a href="compose-multiplatform-setup.md">Set up an environment</a><br/>
       <img src="icon-2-done.svg" width="20" alt="Second step"/> <a href="compose-multiplatform-create-first-app.md">Create your multiplatform project</a><br/>
       <img src="icon-3-done.svg" width="20" alt="Third step"/> <a href="compose-multiplatform-explore-composables.md">Explore composable code</a><br/>
       <img src="icon-4-done.svg" width="20" alt="Fourth step"/> <a href="compose-multiplatform-modify-project.md">Modify the project</a><br/>
       <img src="icon-5.svg" width="20" alt="Fifth step"/> <strong>Create your own application</strong><br/>
  </p>
</microformat>

Now that you've explored and enhanced the sample project created by the wizard, you can create your own application from
scratch, using concepts you already know and introducing some new ones.

You'll create a "Local time application" where users can enter their country and city, and the app will display the time
in the capital city of that country. All the functionality of your Compose Multiplatform app will be implemented in common
code using multiplatform libraries. It'll load and display images within a dropdown menu and will use events, styles, themes,
modifiers, and layouts.

At each stage, you can run the application on all three platforms (iOS, Android, and desktop), or you can focus on the
specific platforms that best suit your needs.

## Lay the foundation

To get started, implement a new `App` composable:

1. In `composeApp/src/commonMain/kotlin`, open the `App.kt` file and replace the code with the following `App`
   composable:

   ```kotlin
   @Composable
   @Preview
   fun App() {
       MaterialTheme {
           var timeAtLocation by remember { mutableStateOf("No location selected") }
           Column {
               Text(timeAtLocation)
               Button(onClick = { timeAtLocation = "13:30" }) { 
                   Text("Show Time At Location")
               }
           }
       }
   }
   ```

   * The layout is a column containing two composables. The first is a `Text` composable, and the second is a `Button`.
   * The two composables are linked by a single shared state, namely the `timeAtLocation` property. The `Text`
     composable is an observer of this state.
   * The `Button` composable changes the state using the `onClick` event handler.

2. Run the application on Android and iOS:

   ![New Compose Multiplatform app on Android and iOS](first-compose-project-on-android-ios-3.png){width=500}

   When you run your application and click the button, the hardcoded time is displayed.

3. Run the application on the desktop. It works, but the window is clearly too large for the UI:

   ![New Compose Multiplatform app on desktop](first-compose-project-on-desktop-3.png){width=400}

4. To fix this, in `composeApp/src/desktopMain/kotlin`, update the `main.kt` file as follows:

    ```kotlin
    fun main() = application {
        val state = rememberWindowState(
            size = DpSize(400.dp, 250.dp),
            position = WindowPosition(300.dp, 300.dp)
        )
        Window(title = "Local Time App", onCloseRequest = ::exitApplication, state = state) {
            App()
        }
    }
    ```

    Here, you set the title of the window and use the `WindowState` type to give the window an initial size and position on
    the screen.

5. Follow the IDE's instructions to import the missing dependencies.
6. Run the desktop application again. Its appearance should improve:

   ![Improved appearance of the Compose Multiplatform app on desktop](first-compose-project-on-desktop-4.png){width=350}

## Support user input

Now let users enter the name of a city to see the time at that location. The simplest way to achieve this is by adding
a `TextField` composable:

1. Replace the current implementation of `App` with the one below:

    ```kotlin
    @Composable
    @Preview
    fun App() {
        MaterialTheme {
            var location by remember { mutableStateOf("Europe/Paris") }
            var timeAtLocation by remember { mutableStateOf("No location selected") }
    
            Column {
                Text(timeAtLocation)
                TextField(value = location, onValueChange = { location = it })
                Button(onClick = { timeAtLocation = "13:30" }) {
                    Text("Show Time At Location")
                }
            }
        }
    }
    ```

    The new code adds both the `TextField` and a `location` property. As the user types into the text field, the value of
    the property is incrementally updated using the `onValueChange` event handler.

2. Follow the IDE's instructions to import the missing dependencies.
3. Run the application on each platform you're targeting:

   ![User input in the Compose Multiplatform app on Android and iOS](first-compose-project-on-android-ios-4.png){width=500}

   ![User input in the Compose Multiplatform app on desktop](first-compose-project-on-desktop-5.png){width=350}

## Calculate time

The next step is to use the given input to calculate time. To do this, create a `currentTimeAt()` function:

1. Return to the `App.kt` file and add the following function:

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

    This function is similar to `todaysDate()`, which you created earlier and which is no longer required.

2. Follow the IDE's instructions to import the missing dependencies.
3. Adjust your `App` composable to invoke `currentTimeAt()`:

    ```kotlin
    @Composable
    @Preview
    fun App() {
        MaterialTheme {
            var location by remember { mutableStateOf("Europe/Paris") }
            var timeAtLocation by remember { mutableStateOf("No location selected") }
    
            Column {
                Text(timeAtLocation)
                TextField(value = location, onValueChange = { location = it })
                Button(onClick = { timeAtLocation = currentTimeAt(location) ?: "Invalid Location" }) {
                    Text("Show Time At Location")
                }
            }
        }
    }
    ```

4. Run the application again and enter a valid timezone.
5. Click the button. You should see the correct time:

   ![Time display in the Compose Multiplatform app on Android and iOS](first-compose-project-on-android-ios-5.png){width=500}

   ![Time display in the Compose Multiplatform app on desktop](first-compose-project-on-desktop-6.png){width=350}

## Improve the style

The application is working, but there are issues with its appearance. The composables could be spaced better, and the
time message could be rendered more prominently.

1. To address these issues, use the following version of the `App` composable:

   ```kotlin
   @Composable
   @Preview
   fun App() {
       MaterialTheme {
           var location by remember { mutableStateOf("Europe/Paris") }
           var timeAtLocation by remember { mutableStateOf("No location selected") }
   
           Column(modifier = Modifier.padding(20.dp)) {
               Text(
                   timeAtLocation,
                   style = TextStyle(fontSize = 20.sp),
                   textAlign = TextAlign.Center,
                   modifier = Modifier.fillMaxWidth().align(Alignment.CenterHorizontally)
               )
               TextField(value = location,
                   modifier = Modifier.padding(top = 10.dp),
                   onValueChange = { location = it })
               Button(modifier = Modifier.padding(top = 10.dp),
                   onClick = { timeAtLocation = currentTimeAt(location) ?: "Invalid Location" }) {
                   Text("Show Time")
               }
           }
       }
   }
   ```

    * The `modifier` parameter adds padding all around the `Column`, as well as at the top of the `Button` and the `TextField`.
    * The `Text` composable fills the available horizontal space and centers its content.
    * The `style` parameter customizes the appearance of the `Text`.

2. Follow the IDE's instructions to import the missing dependencies.
3. Run the application to see how the appearance has improved:

   ![Improved style of the Compose Multiplatform app on Android and iOS](first-compose-project-on-android-ios-6.png){width=500}

   ![Improved style of the Compose Multiplatform app on desktop](first-compose-project-on-desktop-7.png){width=350}

<!--
> You can find this state of the project in our [GitHub repository](https://github.com/kotlin-hands-on/get-started-with-cm/tree/main/ComposeDemoStage2).
>
{type="tip"}-->

## Refactor the design

The application works, but it's susceptible to users' typos. For example, if the user enters "Franse" instead of "France",
the app won't be able to process that input. It would be preferable to ask users to select the country from a predefined
list.

1. To achieve this, change the design in the `App` composable:

    ```kotlin
    data class Country(val name: String, val zone: TimeZone)
    
    fun currentTimeAt(location: String, zone: TimeZone): String {
        fun LocalTime.formatted() = "$hour:$minute:$second"
    
        val time = Clock.System.now()
        val localTime = time.toLocalDateTime(zone).time
    
        return "The time in $location is ${localTime.formatted()}"
    }
    
    fun countries() = listOf(
        Country("Japan", TimeZone.of("Asia/Tokyo")),
        Country("France", TimeZone.of("Europe/Paris")),
        Country("Mexico", TimeZone.of("America/Mexico_City")),
        Country("Indonesia", TimeZone.of("Asia/Jakarta")),
        Country("Egypt", TimeZone.of("Africa/Cairo")),
    )
    
    @Composable
    @Preview
    fun App(countries: List<Country> = countries()) {
        MaterialTheme {
            var showCountries by remember { mutableStateOf(false) }
            var timeAtLocation by remember { mutableStateOf("No location selected") }
    
            Column(modifier = Modifier.padding(20.dp)) {
                Text(
                    timeAtLocation,
                    style = TextStyle(fontSize = 20.sp),
                    textAlign = TextAlign.Center,
                    modifier = Modifier.fillMaxWidth().align(Alignment.CenterHorizontally)
                )
                Row(modifier = Modifier.padding(start = 20.dp, top = 10.dp)) {
                    DropdownMenu(
                        expanded = showCountries,
                        onDismissRequest = { showCountries = false }
                    ) {
                        countries().forEach { (name, zone) ->
                            DropdownMenuItem(
                                onClick = {
                                    timeAtLocation = currentTimeAt(name, zone)
                                    showCountries = false
                                }
                            ) {
                                Text(name)
                            }
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

   * There is a `Country` type, consisting of a name and a timezone.
   * The `currentTimeAt()` function takes a `TimeZone` as its second parameter.
   * The `App` now requires a list of countries as a parameter. The `countries()` function provides the list.
   * `DropdownMenu` has replaced the `TextField`. The value of the `showCountries` property determines the visibility
     of the `DropdownMenu`. There is a `DropdownMenuItem` for each country.

2. Follow the IDE's instructions to import the missing dependencies.
3. Run the application to see the redesigned version:

   ![The country list in the Compose Multiplatform app on Android and iOS](first-compose-project-on-android-ios-7.png){width=500}

   ![The country list in the Compose Multiplatform app on desktop](first-compose-project-on-desktop-8.png){width=350}

<!--> You can find this state of the project in our [GitHub repository](https://github.com/kotlin-hands-on/get-started-with-cm/tree/main/ComposeDemoStage3).
>
{type="tip"}-->

> You can further improve the design using a dependency injection framework, such as [Koin](https://insert-koin.io/),
> to build and inject the table of locations. If the data is stored externally,
> you can use the [Ktor](https://ktor.io/docs/create-client.html) library to fetch it over the network or
> the [SQLDelight](https://github.com/cashapp/sqldelight) library to fetch it from a database.
>
{type="note"}

## Introduce images

The list of country names works, but it's not visually appealing. You can improve it by replacing the names with images
of national flags.

Compose Multiplatform provides a library for accessing resources through common code across all platforms. The Kotlin
Multiplatform wizard has already added and configured this library, so you can start loading resources without having to
modify the build file.

To support images in your project, you'll need to download image files, store them in the correct directory, and add
code to load and display them:

1. Using an external resource, such as [Flag CDN](https://flagcdn.com/), download flags to match the list of countries
   you have already created. In this case, these
   are [Japan](https://flagcdn.com/w320/jp.png), [France](https://flagcdn.com/w320/fr.png), [Mexico](https://flagcdn.com/w320/mx.png), [Indonesia](https://flagcdn.com/w320/id.png),
   and [Egypt](https://flagcdn.com/w320/eg.png).

2. Move the images to the `composeApp/src/commonMain/composeResources/drawable` directory so that the same flags are available on all platforms:

   ![Compose Multiplatform resources project structure](compose-resources-project-structure.png){width=300}

3. Change the codebase to support images:

    ```kotlin
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

            Column(modifier = Modifier.padding(20.dp)) {
                Text(
                    timeAtLocation,
                    style = TextStyle(fontSize = 20.sp),
                    textAlign = TextAlign.Center,
                    modifier = Modifier.fillMaxWidth().align(Alignment.CenterHorizontally)
                )
                Row(modifier = Modifier.padding(start = 20.dp, top = 10.dp)) {
                    DropdownMenu(
                        expanded = showCountries,
                        onDismissRequest = { showCountries = false }
                    ) {
                        countries.forEach { (name, zone, image) ->
                            DropdownMenuItem(
                                onClick = {
                                    timeAtLocation = currentTimeAt(name, zone)
                                    showCountries = false
                                }
                            ) {
                                Row(verticalAlignment = Alignment.CenterVertically) {
                                    Image(
                                        painterResource(image),
                                        modifier = Modifier.size(50.dp).padding(end = 10.dp),
                                        contentDescription = "$name flag"
                                    )
                                    Text(name)
                                }
                            }
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

    * The `Country` type stores the path to the associated image.
    * The list of countries passed to the `App` includes these paths.
    * The `App` displays an `Image` in each `DropdownMenuItem`, followed by a `Text` composable that contains country names.
    * Each `Image` requires a `Painter` object to fetch the data.

4. Follow the IDE's instructions to import the missing dependencies and annotations.
5. Run the application to see the new behavior:

   ![The country flags in the Compose Multiplatform app on Android and iOS](first-compose-project-on-android-ios-8.png){width=500}

   ![The country flags in the Compose Multiplatform app on desktop](first-compose-project-on-desktop-9.png){width=350}

<!-- > You can find this state of the project in our [GitHub repository](https://github.com/kotlin-hands-on/get-started-with-cm/tree/main/ComposeDemoStage4).
>
{type="tip"} -->

## What's next

We encourage you to explore multiplatform development further and try out more projects:

* [Make your Android app cross-platform](multiplatform-integrate-in-existing-app.md)
* [Create a multiplatform app using Ktor and SQLDelight](multiplatform-ktor-sqldelight.md)
* [Share business logic between iOS and Android while keeping the UI native](multiplatform-getting-started.md)
* [See the curated list of sample projects](multiplatform-samples.md)
