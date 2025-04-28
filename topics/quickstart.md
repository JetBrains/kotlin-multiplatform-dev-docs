[//]: # (title: Kotlin Multiplatform quickstart)

With this tutorial, you can get a simple Kotlin Multiplatform app up and running.

## Set up the environment

Kotlin Multiplatform projects need a specific environment,
but most of the requirements are made clear through preflight checks in the IDE.
Start with an IDE and necessary plugins:

1. Choose and install the IDE.
    KMP is supported in IntelliJ IDEA and Android Studio, so you can use the IDE you prefer.

    > The plugins necessary for Kotlin Multiplatform require IntelliJ IDEA 2025.1
    > or Android Studio Narwhal 2025.1 (TODO: this is a guess, check supported versions before publishing).
    >
    {style="note"}

2. Make sure you have all the necessary IDE plugins:

    * The [Kotlin Multiplatform](https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform) plugin (TODO: check the URL)
    * Android plugins need to be installed only if you use IDEA (they come bundled with Android Studio):
      * [Android](https://plugins.jetbrains.com/plugin/22989-android)
      * [Android Design Tools](https://plugins.jetbrains.com/plugin/22990-android-design-tools)
      * [Jetpack Compose](https://plugins.jetbrains.com/plugin/18409-jetpack-compose)
      *  (TODO: check that the Android plugins are ) 

3. To create iOS applications, you also need a macOS host with [Xcode](https://apps.apple.com/us/app/xcode/id497799835) installed.
   Your IntelliJ IDE will run Xcode under the hood to build iOS frameworks.

    Make sure to launch Xcode at least once before starting to work with KMP projects so that it has a chance to
    go through the initial setup.

    > You will need to launch Xcode manually every time it is updated.
    > Preflight checks in the IntelliJ IDE you're using alert you whenever Xcode is not in the right state to work with.

### Setup troubleshooting

TODO common problems
* From existing: `sudo xcode-select`
* JAVA_HOME something? Can this be popular?
* YouTrack link for when stumped.

## Create a project 

Both IntelliJ IDEA and Android Studio explicitly support creating a bare-bones Kotlin Multiplatform project:

1. Select **File | New | Project** in the main menu.
2. Choose **Kotlin Multiplatform** in the list on the left.
3. After setting the name, location, and other usual attributes of the project, choose platforms that you
    would like to see as part of the project.
    * All target platforms can be set up for using Compose Multiplatform to share UI code from the start
      (except for the server module that doesn't have UI code).
    * For iOS, you can choose one of two implementations: 
      * shared UI code, with Compose Multiplatform,
      * or fully native UI, made with SwiftUI and connected to the Kotlin module with shared logic.  
    * The desktop target includes an alpha version of hot reload functionality that allows you to see UI changes
      as soon as you alter corresponding code.
      Even if you're not planning on making desktop apps, you may want to use the desktop version to speed up
      writing UI code.

When you're done choosing platforms, click the **Create** button and wait for the IDE to generate and import the project.

## Run the sample apps

The project created by the IDE wizard includes pregenerated run configurations for iOS and Android
as well as Gradle tasks for running desktop, web, and server applications.

### Run the Android app

TODO: align this with the CMP tutorial. There are a lot of details on running on a physical device, for example,
but that is not quick.

To run the Android app, start the composeApp run configuration:

[screenshot]

By default, it runs on the first available virtual device:

[screenshot]

### Run the iOS app

TODO: align this with the CMP tutorial. There are a lot of details on running on a physical device, for example,
but that is not quick.
* iOS app does not run without Apple Development Team ID?

If you chose the iOS target for the project and set up a macOS machine with Xcode,
you can start the iosApp run configuration:

[screenshot]

By default, it runs on the first available virtual device:

[screenshot]

### Run the desktop app

TODO: align this with the CMP tutorial.

You can create a run configuration for running the desktop application as follows:

1. Select **Run | Edit Configurations** from the main menu.
2. Click the plus button and choose **Gradle** from the dropdown list.
3. In the **Tasks and arguments** field, paste this command:
   ```shell
   run
   ```
4. Click **OK**.

With this configuration you can run the JVM desktop app:

[screenshot]

### Run the web app

TODO: align this with the CMP tutorial.

Create a run configuration to run the web application:

1. Select **Run | Edit Configurations** from the main menu.
2. Click the plus button and choose **Gradle** from the dropdown list.
3. In the **Tasks and arguments** field, paste this command:

   ```shell
   wasmJsBrowserRun -t --quiet
   ```

4. Click **OK**.

With this configuration you can run the web app:

[screenshot]

## What's next

Learn more about the structure of a KMP project and writing shared code:
* A series of tutorials about working with the shared UI code: [TODO current CMP tutorial chain]
* A series of tutorials about working with shared code along with native UI: [TODO current KMP tutorial chain]
* Take a deep dive into the Kotlin Multiplatform documentation:
  * Project structure
  * Working with dependencies
  * Publishing artifacts
* Learn about the Compose Multiplatform UI framework, its fundamentals, and platform-specific features: [CMP overview]

Discover code already written for KMP:
* Our [Samples](multiplatform-samples.md) page, with official JetBrains samples as well as a curated list of
    projects showcasing KMP capabilities.
* The GitHub topics:
  * [kotlin-multiplatform](https://github.com/topics/kotlin-multiplatform), projects implemented with Kotlin Multiplatform.
  * [kotlin-multiplatform-sample](https://github.com/topics/kotlin-multiplatform-sample),
      a list of sample projects written with KMP.
* [klibs.io](https://klibs.io) — search platform for KMP libraries, with more than 2000 libraries indexed so far,
    including OkHttp, Ktor, Coil, Koin, SQLDelight, and others.

## Get help

* **Kotlin Slack**. Get an [invite](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up) and join the [#multiplatform](https://kotlinlang.slack.com/archives/C3PQML5NU) channel.
* **Kotlin issue tracker**. [Report a new issue](https://youtrack.jetbrains.com/newIssue?project=KT).
