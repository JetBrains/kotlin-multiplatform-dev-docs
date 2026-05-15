[//]: # (title: Create your Compose Multiplatform app)

<secondary-label ref="IntelliJ IDEA"/>
<secondary-label ref="Android Studio"/>

<tldr>
    <p>This tutorial uses IntelliJ IDEA, but you can also follow it in Android Studio – both IDEs share the same core functionality and Kotlin Multiplatform support.</p>
    <br/>
    <p>This is the first part of the <strong>Create a Compose Multiplatform app with shared logic and UI</strong> tutorial.</p>
    <p><img src="icon-1.svg" width="20" alt="First step"/> <strong>Create your Compose Multiplatform app</strong><br/>
        <img src="icon-2-todo.svg" width="20" alt="Second step"/> Explore composable code <br/>
        <img src="icon-3-todo.svg" width="20" alt="Third step"/> Modify the project <br/>      
        <img src="icon-4-todo.svg" width="20" alt="Fourth step"/> Create your own application <br/>
    </p>
</tldr>

Here, you'll learn how to create and run your first Compose Multiplatform application using IntelliJ IDEA.

With the [Compose Multiplatform](https://www.jetbrains.com/lp/compose-multiplatform/) UI framework, you can push the
code-sharing capabilities of Kotlin Multiplatform beyond application logic. You can implement the user interface once
and then use it for all the platforms supported by Compose Multiplatform.

In this tutorial, you will build a sample application that runs on Android, iOS, desktop, and web. To create a user
interface, you will use the Compose Multiplatform framework and learn about its basics: composable functions, themes, layouts,
events, and modifiers.

Things to keep in mind for this tutorial:
* No previous experience with Compose Multiplatform, Android, or iOS is required. We do recommend that
  you become familiar with the [fundamentals of Kotlin](https://kotlinlang.org/docs/getting-started.html) before starting.
* To complete this tutorial, you'll only need IntelliJ IDEA. It allows you to try multiplatform development on Android
  and desktop. For iOS, you'll need a macOS machine with Xcode installed. This is a general limitation of iOS development.
* If you wish, you can limit your choice to the specific platforms you're interested in and omit the others.

## Create a project

1. In the [quickstart](quickstart.md), complete the instructions to [set up your environment for Kotlin Multiplatform development](quickstart.md#set-up-the-environment).
2. In IntelliJ IDEA, select **File** | **New** | **Project**.
3. In the panel on the left, select **Kotlin Multiplatform**.

    > If you're not using the Kotlin Multiplatform IDE plugin, you can generate the same project
    > using the [KMP web wizard](https://kmp.jetbrains.com/?android=true&ios=true&iosui=compose&desktop=true&web=true&includeTests=true).
    >
    {style="note"}

4. Specify the following fields in the **New Project** window:

    * **Name**: ComposeDemo
    * **Project ID** (used as the package name): compose.project.demo

5. Select the **Android**, **iOS**, **Desktop**, and **Web** targets.
    Make sure that the **Share UI** option is selected for iOS and web.
6. Once you've specified all the fields and targets, click **Create** (**Download** in the web wizard).

   ![Create Compose Multiplatform project](create-compose-multiplatform-project.png){width=800}

## Examine the project structure

In IntelliJ IDEA, navigate to the `ComposeDemo` folder.
If you didn't select iOS in the wizard, you won't have the folders whose names begin with "ios" or "apple".

> The IDE may automatically suggest upgrading the Android Gradle plugin in the project to the latest version.
> We don't recommend upgrading as Kotlin Multiplatform is not compatible with the latest AGP version
> (see the [compatibility table](https://kotlinlang.org/docs/multiplatform-compatibility-guide.html#version-compatibility)).
>
{style="note"}

The project contains the following modules:

* **shared** is a Kotlin Multiplatform module that contains the logic shared among the Android, desktop, iOS, and web applications
  – the code you use for all the platforms. It uses [Gradle](https://kotlinlang.org/docs/gradle.html) as the build system that helps
  you automate your build process.
* **androidApp** is the module that builds into an Android application.
* **iosApp** is an Xcode project that builds into an iOS application. It depends on and uses the shared module as an iOS
  framework.
* **desktopApp** is the module that builds into a desktop JVM application. It depends on the `shared` module.
* **webApp** is the module that builds into web applications, both Kotlin/JS and Kotlin/Wasm.

![Compose Multiplatform project structure](compose-project-structure.png){width=400}

The **shared** module contains the following source sets: `androidMain`, `commonMain`, `iosMain`, `jsMain`, 
`jvmMain`, and `wasmJsMain` (with `-Test` companion source sets if you chose to include tests).

A _source set_ is a Gradle concept for a number of files logically grouped together, where each group has its own
dependencies. In Kotlin Multiplatform, different source sets usually target different platforms.

The `commonMain` source set uses the common Kotlin code, and platform source sets use Kotlin code specific to each
target:

* `jvmMain` contains source files for the desktop target, which uses Kotlin/JVM.
* `androidMain` contains Android source files and targets Kotlin/JVM.
* `iosMain` contains Kotlin code for iOS and targets Kotlin/Native.
* `jsMain` contains JavaScript-specific Kotlin code and targets Kotlin/JS.
* `wasmJsMain` contains Wasm-specific Kotlin code and targets Kotlin/Wasm.

This way, when the `shared` module is built into an Android library, common Kotlin code gets treated as Kotlin/JVM, and
when it is built into an iOS framework, common Kotlin code gets treated as Kotlin/Native.
When the shared module is built into a web app, common Kotlin code can be treated as Kotlin/Wasm or Kotlin/JS
as necessary.

![Common Kotlin, Kotlin/JVM, and Kotlin/Native](module-structure.svg){width=700}

In general, write your implementation as common code whenever possible,
to take advantage of implementing only once the functionality that should work the same across platforms.
Platform-specific source sets ideally contain only platform-specific API calls and UX flows.

In the `shared/src/commonMain/kotlin` directory, open the `App.kt` file. It contains the `App()` function, which implements a
minimalistic but complete Compose Multiplatform UI:

```kotlin
@Composable
@Preview
fun App() {
    MaterialTheme {
        var showContent by remember { mutableStateOf(false) }
        Column(
            modifier = Modifier
                .background(MaterialTheme.colorScheme.primaryContainer)
                .safeContentPadding()
                .fillMaxSize(),
            horizontalAlignment = Alignment.CenterHorizontally,
        ) {
            Button(onClick = { showContent = !showContent }) {
                Text("Click me!")
            }
            AnimatedVisibility(showContent) {
                val greeting = remember { Greeting().greet() }
                Column(
                    modifier = Modifier.fillMaxWidth(),
                    horizontalAlignment = Alignment.CenterHorizontally,
                ) {
                    Image(painterResource(Res.drawable.compose_multiplatform), null)
                    Text("Compose: $greeting")
                }
            }
        }
    }
}
```
{id="common-app-composable"}

Let's run the application on all supported platforms.

## Run your application

You can run the application on Android, iOS, desktop, and web. You don't have to run the applications in any particular
order, so start with whichever platform you are most familiar with.

> The provided run configurations are more efficient than the general Gradle build task.
> Run configurations only trigger builds for corresponding targets while the default Gradle task
> builds debug and release versions of all targets.
>
{style="tip"}

### Run your application on Android

1. In the list of run configurations, select **androidApp**.
2. Choose your Android virtual device and then click **Run**: Your IDE starts the selected virtual device if it
   is powered down and runs the app.

![Run the Compose Multiplatform app on Android](compose-run-android.png){width=352}

![First Compose Multiplatform app on Android](first-compose-project-on-android-1.png){width=352}

<snippet id="run_android_other_devices">

#### Run on a different Android simulated device {initial-collapse-state="collapsed" collapsible="true"}

Learn how to [configure the Android Emulator and run your application on a different simulated device](https://developer.android.com/studio/run/emulator#runningapp).

#### Run on a real Android device {initial-collapse-state="collapsed" collapsible="true"}

Learn how to [configure and connect a hardware device and run your application on it](https://developer.android.com/studio/run/device).

</snippet>

### Run your application on iOS

If you haven't launched Xcode as part of the initial setup, do that before running the iOS app.

In IntelliJ IDEA, select **iosApp** in the list of run configurations, select a simulated device next to the run configuration,
and click **Run**.

![Run the Compose Multiplatform app on iOS](compose-run-ios.png){width=405}

![First Compose Multiplatform app on iOS](first-compose-project-on-ios-1.png){width=411}

<snippet id="run_ios_other_devices">

#### Run on a real iOS device {initial-collapse-state="collapsed" collapsible="true"}

You can run your multiplatform application on a real iOS device. Before you start,
you'll need to set the Team ID associated with your [Apple ID](https://support.apple.com/en-us/HT204316).

##### Set your Team ID

To set new Team ID for your project for the first time, open the project in Xcode
(**File | Open Project in Xcode**):

1. In the Project navigator on the left-hand side, select **iosApp**.
2. Select **iosApp** under **Targets** and switch to the **Signing & Capabilities** tab.
3. In the **Team** list, select your team.

   If you haven't set up your team yet, use the **Add an Account** option in the **Team** list and follow Xcode instructions.

4. Make sure that the Bundle Identifier is unique and a Signing Certificate is successfully assigned.

After you set up a team in Xcode, you can set or change the team in IntelliJ IDEA:

1. Edit the run configuration for **iosApp**:

   ![Edit iOS run configuration](ios-edit-configurations.png){width=450}

2. Switch to the **Options** tab and make the necessary changes in the **Development team** dropdown, then click **OK**.

##### Run the app

Connect your iPhone with a cable. If you already have the device registered in Xcode, IntelliJ IDEA should show it
in the list of run configurations. Run the corresponding `iosApp` configuration.

If you haven't registered your iPhone in Xcode yet, follow [Apple recommendations](https://developer.apple.com/documentation/xcode/running-your-app-in-simulator-or-on-a-device/).
In short, you should:

1. Connect your iPhone with a cable.
2. On your iPhone, enable the developer mode in **Settings** | **Privacy & Security**.
3. In Xcode, go to the top menu and choose **Window** | **Devices and Simulators**.
4. If your iPhone is not shown as connected, click the plus sign at the bottom left and select it.
5. Follow the on-screen instructions to complete the pairing process.

Once you've registered your iPhone in Xcode, it will become available in the list of available devices in IntelliJ IDEA
when you select the **iosApp** run configuration.

</snippet>

### Run your application on desktop

Select **desktopApp [hot] 🔥** in the list of run configurations and click **Run**.
By default, the run configuration starts a desktop app in its own OS window with [Compose Hot Reload](compose-hot-reload.md) running:

![Run the Compose Multiplatform app on desktop](compose-run-desktop.png){width=350}

![First Compose Multiplatform app on desktop](first-compose-project-on-desktop-1.png){width=500}

### Run your web application

1. In the list of run configurations, select:

   * **webApp[js]**: To run your Kotlin/JS application.
   * **webApp[wasmJs]**: To run your Kotlin/Wasm application.

2. Click **Run**.

The web application opens automatically in your default browser
and is available by default at `http://localhost:8080/`. 

> The port number can vary because port 8080 may be unavailable.
> You can find the actual port number in the Gradle build console by searching for the phrase "Project is running at".
>
{style="tip"}

![Compose web application](first-compose-project-on-web.png){width=600}

#### Compatibility mode for web targets

You can enable compatibility mode for your web application to ensure it works on all browsers out of the box.
In this mode, modern browsers use the Wasm version, while older ones fall back to the JS version.
This mode is achieved through cross-compilation for both the `js` and `wasmJs` targets.

To enable compatibility mode for your web application:

1. Open the Gradle tool window by selecting **View | Tool Windows | Gradle**.
2. In **ComposeDemo | Tasks | compose**, select and run the **composeCompatibilityBrowserDistribution** task.

   > You need at least Java 11 as your Gradle JVM for the tasks to load successfully, and we recommend at least
   > JetBrains Runtime 17 for Compose Multiplatform projects in general.
   >
   {style="note"}

   ![Run compatibility task](web-compatibility-gradle-task.png){width=500}

   Alternatively, you can run the following command in the terminal from the `ComposeDemo` root directory:

    ```bash
    ./gradlew composeCompatibilityBrowserDistribution
    ```

Once the Gradle task completes, compatible artifacts are generated in the
`composeApp/build/dist/composeWebCompatibility/productionExecutable` directory.
You can use these artifacts to [publish your application](https://kotlinlang.org/docs/wasm-get-started.html#publish-the-application) 
working on both the `js` and `wasmJs` targets.

## Next step

In the next part of the tutorial, you'll learn how to implement composable functions and launch your application on each
platform.

**[Proceed to the next part](compose-multiplatform-explore-composables.md)**

## Get help

* **Kotlin Slack**. Get an [invite](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up) and join
  the [#multiplatform](https://kotlinlang.slack.com/archives/C3PQML5NU) channel.
* **Kotlin issue tracker**. [Report a new issue](https://youtrack.jetbrains.com/newIssue?project=KT).
