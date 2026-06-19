[//]: # (title: Create your first Compose Multiplatform app)

<secondary-label ref="IntelliJ IDEA"/>
<secondary-label ref="Android Studio"/>

<tldr>
    <p>This tutorial uses IntelliJ IDEA, but you can also follow it in Android Studio – both IDEs share the same core functionality and Kotlin Multiplatform support.</p>
</tldr>

With the [Compose Multiplatform](https://www.jetbrains.com/lp/compose-multiplatform/) UI framework,
you can implement the user interface once and then use it for all [supported platforms](compose-compatibility-and-versioning.md#supported-platforms).

On this page, you'll learn:
* To create a basic Compose Multiplatform app and run it on Android, iOS, desktop, and web.
* To run the app on various platforms.
* To change the UI on all platforms at once by leveraging common code.

## Requirements

To complete this tutorial, you'll need IntelliJ IDEA or Android Studio and the [Kotlin Multiplatform IDE plugin](https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform).
To build and run iOS apps, you'll also need a macOS machine with Xcode installed (an Apple requirement).

No previous experience with Compose Multiplatform, Android, or iOS is required,
although we do recommend that you become familiar with the [fundamentals of Kotlin](https://kotlinlang.org/docs/getting-started.html) before starting.

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

* **shared** is a Kotlin Multiplatform module that contains the code common for the Android, desktop, iOS, and web applications.
* **androidApp** is the module that builds into an Android application.
* **iosApp** is an Xcode project that builds into an iOS application. It depends on and uses the shared module as an iOS
  framework.
* **desktopApp** is the module that builds into a desktop JVM application.
* **webApp** is the module that builds into web applications, both Kotlin/JS and Kotlin/Wasm.

As Kotlin Multiplatform uses Gradle, it makes use of Gradle _source sets_ for organizing code. <Here's a link to details.>

The `shared` module in your default project contains the following source sets:

* `commonMain` with the common Kotlin code.
* `jvmMain` with source files for the Kotlin/JVM target which are built into a desktop app.
* `androidMain` with source files for the Kotlin/JVM target which are built into an Android app.
* `iosMain` with source files for the Kotlin/Native target which are built into an iOS app.
* `jsMain` with source files for the Kotlin/JS target which are built into a JavaScript app.
* `wasmJsMain` with source files for the Kotlin/Wasm target which are built into a Wasm app.

When the Android app is built, the `shared` module is treated as Kotlin/JVM and is built into an Android library.
When an iOS app is built, the `shared` module is treated as Kotlin/Native and is built into an iOS framework.
When a web app is built, common Kotlin code can be treated as Kotlin/Wasm or Kotlin/JS as necessary.

![Common Kotlin, Kotlin/JVM, and Kotlin/Native](module-structure.svg){width=700}

This is why you should write your code as common whenever possible:
to take advantage of writing platform-independent implementations only once and automatically propagating them
to all applications.

## Run your application

The KMP IDE plugin provides ready-to-go run configurations for all selected targets:

* Run **androidApp** to launch the Anroid app on an emulator.
  If you don't have a virtual Android device yet, open the **Device Manager** (**View | Tool Windows | Device Manager**)
  and create one.
* Run **iosApp** to launch the iOS app on the iPhone Simulator.
  To run the iOS app, you need Xcode installed and the latest iOS SDK downloaded through Xcode.
* Run **desktopApp [hot]** to launch the desktop app in the [](compose-hot-reload.md) mode.
* Run **webApp** to launch the web app.
  By default, the web app is launched at `http://localhost:8080/`.
  If that port is in use, the build automatically selects another available port.
  You can find the selected port in the Gradle build output. 

<!-- TODO For more details on possible app running use cases (like using real mobile devices),
see [](build-and-run-kmp.md). -->

## Updating the UI

<!-- TODO: tweak the UI and run the app to see the changes in all targets -->

## Get help

* **Kotlin Slack**. Get an [invite](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up) and join
  the [#multiplatform](https://kotlinlang.slack.com/archives/C3PQML5NU) channel.
* **Kotlin issue tracker**. [Report a new issue](https://youtrack.jetbrains.com/newIssue?project=KT).
