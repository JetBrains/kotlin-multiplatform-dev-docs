[//]: # (title: Your first Kotlin Multiplatform app)

<secondary-label ref="IntelliJ IDEA"/>
<secondary-label ref="Android Studio"/>

<tldr>
    <p>This tutorial uses IntelliJ IDEA, but you can also follow it in Android Studio – both IDEs share the same core functionality and Kotlin Multiplatform support.</p>
</tldr>

With the Kotlin Multiplatform,
you can implement business logic once in common code and keep the UI for each app native.
If you'd like to share the UI code as well, see our [Compose Multiplatform tutorial](compose-multiplatform-create-first-app.md).

On this page, you'll learn:
* To create a basic Kotlin Multiplatform app that runs on Android and iOS.
* To build and run the app on various platforms.
* To update the shared code to see changes reflected in both apps.

## Requirements

To complete this tutorial, you'll need IntelliJ IDEA or Android Studio and the [Kotlin Multiplatform IDE plugin](https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform).
To build and run iOS apps, you'll also need a macOS machine with Xcode installed (an Apple requirement).

No previous experience with Kotlin Multiplatform is required,
although we do recommend that you become familiar with the [fundamentals of Kotlin](https://kotlinlang.org/docs/getting-started.html) before starting.

## Create a project

With the IDE and the Kotlin Multiplatform IDE plugin installed, you can create a new Compose Multiplatform project:

1. In IntelliJ IDEA, select **File** | **New** | **Project**.
2. In the panel on the left, select **Kotlin Multiplatform**.
3. Specify the following fields in the **New Project** window:

    * **Name**: GreetingKMP
    * **Project ID** (used as the package name): com.jetbrains.greetingkmp

4. Select the **Android** and **iOS** targets.
   For iOS, select the **Do not share UI** option to keep the UI native.
5. Click **Create**.

   ![Create Kotlin Multiplatform project](create-first-multiplatform-app.png){width=700}

## Examine the project structure

In IntelliJ IDEA, expand the `GreetingKMP` folder.

The Kotlin Multiplatform project includes the following modules:

* **androidApp** is a Kotlin module that builds into an Android application. It uses Gradle as the build system.
  The **androidApp** module depends on and uses the **sharedLogic** module as a regular Android library.
* **iosApp** is the Xcode project that builds into an iOS application.
  It depends on the **sharedLogic** module, which is exported as an iOS framework.
  Kotlin Multiplatform projects created using the IDE wizard use the regular framework dependency through
  [direct integration](multiplatform-direct-integration.md).
* **sharedLogic** is the multiplatform module that contains the logic common for both Android and iOS applications.
* **sharedUI** is the module with Compose Multiplatform UI code: in this project, it is used only by the Android app,
  but it is a multiplatform module that can be used by other targets whenever you need that.

Every module except for **iosApp** uses Gradle as the build system.

![Basic Multiplatform project structure](basic-project-structure.svg){width=700}
<!-- TODO need to redo the diagram: ios depends on sharedLogic while android on both sharedLogic and sharedUI -->

_Source set_ is a Gradle concept for a set of files logically grouped together where each set has its own dependencies.
In Kotlin Multiplatform, source sets in a shared module help separate code that targets different platforms.

The `sharedLogic` module contains the `androidMain`, `commonMain`, and `iosMain` source sets.
The `commonMain` source set contains shared Kotlin code, and platform source sets use Kotlin code specific to each target.
Kotlin/JVM is used for `androidMain` and Kotlin/Native for `iosMain`:

![Source sets and modules structure](basic-project-structure-2.png){width=350}

When the shared module is built into an Android library, common Kotlin code is treated as Kotlin/JVM.
When it is built into an iOS framework, common Kotlin is treated as Kotlin/Native:

![Common Kotlin, Kotlin/JVM, and Kotlin/Native](modules-structure.png)

## Run your application

The KMP IDE plugin provides ready-to-go run configurations for selected targets:

* Run **androidApp** to launch the Anroid app on an emulator.
  If you don't have a virtual Android device yet, open the **Device Manager** (**View | Tool Windows | Device Manager**)
  and create one.
* Run **iosApp** to launch the iOS app on the iPhone Simulator.
  To run the iOS app, you need Xcode installed and the latest iOS SDK downloaded through Xcode.

<!-- TODO needs a new screenshot -->

<!-- TODO For more details on possible app running use cases (like using real mobile devices),
see [](build-and-run-kmp.md).-->

## Update shared code

The **sharedLogic** code generates a greeting, which is then displayed in the UI of both apps.
You will alter the common code and then run the apps again to see how native UIs use the updated functionality:

1. Open the `sharedLogic/src/commonMain/.../Greeting.kt` file where you can find the generated `Greeting` class with a `greet()` function.
2. Change the greeting a little:
   Navigate to the definition of the `sayHello()` function in the `GreetingUtil.kt` file.
3. Randomize the first word and reverse the name of the platform:

    ```kotlin
    fun sayHello(to: String): String {
        val firstWord = if (Random.nextBoolean()) "Hi!" else "Hello!"

        return "$firstWord Guess what this is! > ${to.reversed()}!"
    }
    ```
4. Import the `kotlin.random.Random` class following the IDE's suggestion.
5. Run your apps again to see the new greeting produced both in iOS and Android:

<!-- TODO needs screenshots in tabs, like for CMP -->

## Get help

* **Kotlin Slack**. Get an [invite](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up) and join
  the [#multiplatform](https://kotlinlang.slack.com/archives/C3PQML5NU) channel.
* **Kotlin issue tracker**. [Report a new issue](https://youtrack.jetbrains.com/newIssue?project=KT).
