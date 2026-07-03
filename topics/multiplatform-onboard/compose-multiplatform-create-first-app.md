[//]: # (title: Your first Compose Multiplatform app)

<secondary-label ref="IntelliJ IDEA"/>
<secondary-label ref="Android Studio"/>

<tldr>
    <p>This tutorial uses IntelliJ IDEA, but you can also follow it in Android Studio – both IDEs share the same core functionality and Kotlin Multiplatform support.</p>
</tldr>

With the [Compose Multiplatform](https://www.jetbrains.com/lp/compose-multiplatform/) UI framework,
you can implement the user interface once and then use it for all [supported platforms](compose-compatibility-and-versioning.md#supported-platforms).

On this page, you'll learn:
* To create a basic Compose Multiplatform app and run it on Android, iOS, desktop, and web.
* To build and run the app on various platforms.
* To change the UI on all platforms at once by leveraging common code.

## Requirements

To complete this tutorial, you'll need IntelliJ IDEA or Android Studio and the [Kotlin Multiplatform IDE plugin](https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform).
To build and run iOS apps, you'll also need a macOS machine with Xcode installed (an Apple requirement).

No previous experience with Compose Multiplatform is required,
although we do recommend that you familiarize yourself with the [fundamentals of Kotlin](https://kotlinlang.org/docs/getting-started.html) before starting.

## Create a project

With the IDE and the Kotlin Multiplatform IDE plugin installed, you can create a new Compose Multiplatform project:

1. In IntelliJ IDEA, select **File** | **New** | **Project**.
2. In the panel on the left, select **Kotlin Multiplatform**.
3. Specify the following fields in the **New Project** window:

    * **Name**: ComposeDemo
    * **Project ID** (used as the package name): compose.project.demo

4. Select the **Android**, **iOS**, **Desktop**, and **Web** targets.
    Make sure that the **Share UI** option is selected for iOS and web.
5. Once you've specified all the fields and targets, click **Create** (**Download** in the web wizard).

   ![Create Compose Multiplatform project](create-compose-multiplatform-project.png){width=800}

## Examine the project structure

If you pick all client targets, you'll see the following directories in the **Project** view:

![Compose Multiplatform project structure](compose-project-structure.png){width=400}

There's a module for each target application and the shared module with multiplatform code:

* **androidApp** is the module that builds into an Android application.
* **desktopApp** is the module that builds into a desktop JVM application.
* **iosApp** is an Xcode project that builds into an iOS application. It depends on and uses the shared module as an iOS
  framework.
* **shared** is a Kotlin Multiplatform module that contains the code common for the Android, desktop, iOS, and web applications.
* **webApp** is the module that builds into web applications, both Kotlin/JS and Kotlin/Wasm.

When using Gradle, Kotlin Multiplatform makes use of Gradle _source sets_ for organizing shared code.
In short, every target can have a corresponding source set that contains platform-specific code intended for that target.
Source sets also have a default hierarchy to make it possible to share code between related targets (Wasm and JS, for example).

> Learn more about source set organization in Kotlin Multiplatform in [](multiplatform-discover-project.md).
> 
{style="note"}

During a build, the `shared` module is treated appropriately for each target.
For example, it's treated as Kotlin/JVM for building an Android app and as Kotlin/Native for building an iOS app.

## Run your application

The Kotlin Multiplatform IDE plugin provides ready-to-go run configurations for all selected targets:

* Run **androidApp** to launch the Anroid app on an emulator.
  If you don't have a virtual Android device yet, open the **Device Manager** (**View | Tool Windows | Device Manager**)
  and create one.
* Run **iosApp** to launch the iOS app on the iPhone Simulator.
  To run an iOS app, you need Xcode installed and the latest iOS SDK downloaded through Xcode.
* Run **desktopApp [hot]** to launch the desktop app in the [](compose-hot-reload.md) mode.
* Run **webApp** to launch the web app.
  By default, the web app is launched at `http://localhost:8080/` a successful build opens this address in your default browser.
  If that port is in use, the build automatically selects another available port and communicates that in the Gradle build output. 

<!-- TODO For more details on possible app running use cases (like using real mobile devices),
see [](build-and-run-kmp.md). -->

## Update the UI

The default project is working.
Now check out how changes in common UI code update the UI in every app.

Add a new text line to the greeting that is displayed after the button press:

1. In the `shared/src/commonMain/kotlin/App.kt` file, update the `App()` composable
   to include a new `Text()`:

    ```kotlin
    @Composable
    @Preview
    fun App() {
        MaterialTheme {
            var showContent by remember { mutableStateOf(false) }
            val greeting = remember { Greeting().greet() }
            Column(
                modifier = Modifier
                    .safeContentPadding()
                    .fillMaxSize(),
                horizontalAlignment = Alignment.CenterHorizontally
            ) {
                Button(onClick = { showContent = !showContent }) {
                    Text("Click me!")
                    // TODO: this code is likely wrong
                    Text(
                        text = "Today's date is ...",
                        modifier = Modifier.padding(20.dp),
                        fontSize = 24.sp,
                        textAlign = TextAlign.Center
                    )
                }
                AnimatedVisibility(showContent) {
                    Column(Modifier.fillMaxWidth(), horizontalAlignment = Alignment.CenterHorizontally) {
                        Image(painterResource(Res.drawable.compose_multiplatform), null)
                        Text("Compose: $greeting")
                    }
                }
            }
        }
    }
    ```

2. Follow the IDE's suggestions to import the missing dependencies.

   <!-- TODO: screenshot needs updating -->
   ![Unresolved references](compose-unresolved-references.png)

## Rerun the application

<!-- TODO all screenshots will be outdated as the sample UI update is different (not using the datetime API) -->

You can now [rerun the application](compose-multiplatform-create-first-app.md#run-your-application)
using the same run configurations for Android, iOS, desktop, and web:

<tabs>
    <tab id="mobile-app" title="Android and iOS">
        <img src="first-compose-project-on-android-ios-2.png" alt="First Compose Multiplatform app on Android and iOS" width="500"/>
    </tab>
    <tab id="desktop-app" title="Desktop">
        <img src="first-compose-project-on-desktop-2.png" alt="First Compose Multiplatform app on desktop" width="600"/>
    </tab>
    <tab id="web-app" title="Web">
        <img src="first-compose-project-on-web-2.png" alt="First Compose Multiplatform app on web" width="600"/>
    </tab>
</tabs>

## Get help

* **Kotlin Slack**. Get an [invite](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up) and join
  the [#multiplatform](https://kotlinlang.slack.com/archives/C3PQML5NU) channel.
* **Kotlin issue tracker**. [Report a new issue](https://youtrack.jetbrains.com/newIssue?project=KT).
