[//]: # (title: Create your multiplatform project)

<microformat>
   <p>This is the second part of the <strong>Getting started with Compose Multiplatform</strong> tutorial. Before proceeding, make sure you've completed the previous step.</p>
   <p><img src="icon-1-done.svg" width="20" alt="First step"/> <a href="compose-multiplatform-setup.md">Set up an environment</a><br/>
      <img src="icon-2.svg" width="20" alt="Second step"/> <strong>Create your multiplatform project</strong><br/>
      <img src="icon-3-todo.svg" width="20" alt="Third step"/> Explore composable code<br/>      
      <img src="icon-4-todo.svg" width="20" alt="Fourth step"/> Modify the project<br/>
      <img src="icon-5-todo.svg" width="20" alt="Fifth step"/> Create your own application<br/>
  </p>
</microformat>

Here, you'll learn how to create and run your first Compose Multiplatform application using a web wizard and Android
Studio.

## Create the project with a wizard

To begin, create a sample project. This is best achieved with the Kotlin Multiplatform web wizard:

1. Open the [Kotlin Multiplatform wizard](https://kmp.jetbrains.com).
2. On the **New project** tab, change the project name to "ComposeDemo".
3. Select the **Android** and **Desktop** options.
4. If you use Mac, add **iOS** as well. Make sure that the **Share UI** option is selected.
5. Click the download button and unpack the resulting archive.

![Kotlin Multiplatform wizard](multiplatform-web-wizard.png){width=450}

## Examine the project structure

1. Launch Android Studio.
2. On the welcome screen, click **Open** or select **File | Open** in the editor.
3. Navigate to the unpacked "ComposeDemo" folder and click **Open**.

   Android Studio detects that the folder contains a Gradle build file and opens the folder as a new project.

   If you didn't select iOS in the wizard, you won't have the folders whose names begin with "ios" or "apple".

4. The default view in Android Studio is optimized for Android development. To see the full file structure of the project,
   which is more convenient for multiplatform development, switch the view from **Android** to **Project**:

   ![Select the Project view](select-project-view.png){width=200}

The project contains two modules:

* _composeApp_ is a Kotlin module that contains the logic common for Android, desktop, and iOS applications – the code
  you share between platforms. It uses [Gradle](https://kotlinlang.org/docs/gradle.html) as the build system that helps
  you automate your build process.
* _iosApp_ is an Xcode project that builds into an iOS application. It depends on and uses the shared module as an iOS
  framework.

  ![Compose Multiplatform project structure](compose-project-structure.png){width=350}

The **composeApp** module consists of four source sets: `androidMain`, `commonMain`, `desktopMain`, and `iosMain`.
A _source set_ is a Gradle concept for a number of files logically grouped together where each group has its own
dependencies. In Kotlin Multiplatform, different source sets can target different platforms.

The `commonMain` source set uses the common Kotlin code, and platform source sets use Kotlin code specific to each
target. Kotlin/JVM is used for `androidMain` and `desktopMain`. Kotlin/Native is used for `iosMain`.

When the shared module is built into an Android library, common Kotlin code gets treated as Kotlin/JVM. When it is built
into an iOS framework, common Kotlin code gets treated as Kotlin/Native:

![Common Kotlin, Kotlin/JVM, and Kotlin/Native](modules-structure.png){width=700}

In general, write your implementation as common code whenever possible instead of duplicating functionality
in platform-specific source sets.

In `composeApp/src/commonMain/kotlin`, open the `App.kt` file. It contains the `App()` function, which implements a
minimalist but complete Compose Multiplatform UI:

![Compose Multiplatform UI app](compose-multiplatform-ui-app.png){width=700}

Let's run the application and then examine the code in depth.

## Run your application

You can run the application on Android, iOS, and desktop. You don't have to run the applications in any particular
order, so start with whichever platform you are most familiar with.

> You don't need to use the Gradle build task. In a multiplatform application, this will build debug and release versions
> of all the supported targets. Depending on the platforms selected in the Multiplatform wizard, this can take some time.
> Using a run configuration is much faster; only the selected target is built in this case.
>
{type="tip"}

### Run your application on Android

1. Create an [Android virtual device](https://developer.android.com/studio/run/managing-avds#createavd).
2. In the list of run configurations, select **composeApp**.
3. Choose your Android virtual device and click **Run**.

![Run Compose Multiplatform app on Android](compose-run-android.png){width=350}

![First Compose Multiplatform app on Android](first-compose-project-on-android-1.png){width=300}

### Run your application on iOS

1. Launch Xcode in a separate window to complete the initial setup. If it is the first time you ever launch Xcode, you 
may also need to accept its license terms and allow it to perform some necessary initial tasks.
2. Return to Android Studio.
3. In the list of run configurations, select **iosApp** and click **Run**.

If you don't have an available iOS configuration in the list, add
a [new iOS simulated device](#run-on-a-new-ios-simulated-device).

![Run Compose Multiplatform app on iOS](compose-run-ios.png){width=350}

![First Compose Multiplatform app on iOS](first-compose-project-on-ios-1.png){width=300}

#### Run on a new iOS simulated device {initial-collapse-state="collapsed"}

If you want to run your application on a simulated device, you can add a new run configuration.

1. In the list of run configurations, click **Edit Configurations**.

   ![Edit run configurations](ios-edit-configurations.png){width=450}

2. Click the **+** button above the list of configurations and select **iOS Application**.

   ![New run configuration for iOS application](ios-new-configuration.png)

3. Name your configuration.
4. Select the **Xcode project file**. To do so, navigate to your project, for example **KotlinMultiplatformSandbox**,
   open the`iosApp` folder, and select the `.xcodeproj` file.

5. In the **Execution target** list, select a simulated device and click **OK**.

   ![New run configuration with iOS simulator](ios-new-simulator.png)

6. Click **Run** to run your application on the new simulated device.

### Run your application on desktop

You can run the application on the desktop as follows:

1. Go to `composeApp/src/desktopMain/kotlin`.
2. Open the `main.kt` file and find the `main()` function:

   ![Compose Multiplatform desktop app](first-compose-project-on-desktop-main.png){width=350}

3. Click the green run icon in the gutter next to the `main()` function:

   ![First Compose Multiplatform app on desktop](first-compose-project-on-desktop-1.png){width=400}

Once you run your desktop application, a new **Run Configuration** is created. You can use it from now on to run the
desktop application:

![Run Compose Multiplatform app on iOS](compose-run-desktop.png){width=350}

## Next step

In the next part of the tutorial, you'll learn how to implement composable functions and launch your application on each
platform.

**[Proceed to the next part](compose-multiplatform-explore-composables.md)**

## Get help

* **Kotlin Slack**. Get an [invite](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up) and join
  the [#multiplatform](https://kotlinlang.slack.com/archives/C3PQML5NU) channel.
* **Kotlin issue tracker**. [Report a new issue](https://youtrack.jetbrains.com/newIssue?project=KT).