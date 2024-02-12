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

Here, you'll learn how to create and run your first Compose Multiplatform application using the Kotlin Multiplatform
web wizard and Android Studio.

## Create a project using the wizard

To begin, create a sample project. This is best achieved with the Kotlin Multiplatform web wizard:

1. Open the [Kotlin Multiplatform wizard](https://kmp.jetbrains.com).
2. On the **New project** tab, change the project name to "ComposeDemo" and the project ID to "kmp.project.demo".
3. Select the **Android** and **Desktop** options.
4. If you're using a Mac, select **iOS** as well. Make sure that the **Share UI** option is selected.
5. Click the **Download** button and unpack the resulting archive.

![Kotlin Multiplatform wizard](multiplatform-web-wizard.png){width=450}

## Examine the project structure

1. Launch Android Studio.
2. On the Welcome screen, click **Open**, or select **File | Open** in the editor.
3. Navigate to the unpacked "ComposeDemo" folder and then click **Open**.

   Android Studio detects that the folder contains a Gradle build file and opens the folder as a new project.

   If you didn't select iOS in the wizard, you won't have the folders whose names begin with "ios" or "apple".

4. The default view in Android Studio is optimized for Android development. To see the full file structure of the project,
   which is more convenient for multiplatform development, switch the view from **Android** to **Project**:

   ![Select the Project view](select-project-view.png){width=200}

The project contains two modules:

* _composeApp_ is a Kotlin module that contains the logic shared among the Android, desktop, and iOS applications – the code
  you use for all the platforms. It uses [Gradle](https://kotlinlang.org/docs/gradle.html) as the build system that helps
  you automate your build process.
* _iosApp_ is an Xcode project that builds into an iOS application. It depends on and uses the shared module as an iOS
  framework.

  ![Compose Multiplatform project structure](compose-project-structure.png){width=350}

The **composeApp** module consists of four source sets: `androidMain`, `commonMain`, `desktopMain`, and `iosMain`.
A _source set_ is a Gradle concept for a number of files logically grouped together, where each group has its own
dependencies. In Kotlin Multiplatform, different source sets can target different platforms.

The `commonMain` source set uses the common Kotlin code, and platform source sets use Kotlin code specific to each
target. Kotlin/JVM is used for `androidMain` and `desktopMain`. Kotlin/Native is used for `iosMain`.

When the shared module is built into an Android library, common Kotlin code gets treated as Kotlin/JVM. When it is built
into an iOS framework, common Kotlin code gets treated as Kotlin/Native:

![Common Kotlin, Kotlin/JVM, and Kotlin/Native](modules-structure.png){width=700}

In general, write your implementation as common code whenever possible instead of duplicating functionality
in platform-specific source sets.

In `composeApp/src/commonMain/kotlin`, open the `App.kt` file. It contains the `App()` function, which implements a
minimalistic but complete Compose Multiplatform UI:

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
3. Choose your Android virtual device and then click **Run**.

![Run the Compose Multiplatform app on Android](compose-run-android.png){width=350}

![First Compose Multiplatform app on Android](first-compose-project-on-android-1.png){width=300}

<chunk id="run_android_other_devices">

#### Run on a different Android simulated device {initial-collapse-state="collapsed"}

Learn how to [configure the Android Emulator and run your application on a different simulated device](https://developer.android.com/studio/run/emulator#runningapp).

#### Run on a real Android device {initial-collapse-state="collapsed"}

Learn how to [configure and connect a hardware device and run your application on it](https://developer.android.com/studio/run/device).

</chunk>

### Run your application on iOS

1. Launch Xcode in a separate window to complete the initial setup. If it's the first time you launch Xcode, you
   may also need to accept its license terms and allow it to perform some necessary initial tasks.
2. In Android Studio, select **iosApp** in the list of run configurations and click **Run**.

If you don't have an available iOS configuration in the list, add a [new run configuration](#run-on-a-new-ios-simulated-device).

![Run the Compose Multiplatform app on iOS](compose-run-ios.png){width=350}

![First Compose Multiplatform app on iOS](first-compose-project-on-ios-1.png){width=300}

<chunk id="run_ios_other_devices">

#### Run on a new iOS simulated device {initial-collapse-state="collapsed"}

If you want to run your application on a simulated device, you can add a new run configuration.

1. In the list of run configurations, click **Edit Configurations**.

   ![Edit run configurations](ios-edit-configurations.png){width=450}

2. Click the **+** button above the list of configurations and then select **iOS Application**.

   ![New run configuration for iOS application](ios-new-configuration.png)

3. Name your configuration.
4. Select the **Xcode project file**. To do so, navigate to your project, for example, **KotlinMultiplatformSandbox**,
   open the `iosApp` folder, and then select the `.xcodeproj` file.

5. In the **Execution target** list, select a simulated device and then click **OK**.

   ![New run configuration with iOS simulator](ios-new-simulator.png)

6. Click **Run** to run your application on the new simulated device.

#### Run on a real iOS device {initial-collapse-state="collapsed"}

You can run your multiplatform application on a real iOS device. Before you start,
you'll need to set the Team ID associated with your [Apple ID](https://support.apple.com/en-us/HT204316).

##### Set your Team ID

To set the Team ID in your project, you can either use the KDoctor tool in Android Studio or choose your team in Xcode.

For KDoctor:

1. In Android Studio, run the following command in the terminal:

   ```none
   kdoctor --team-ids 
   ```

   KDoctor will list all Team IDs currently configured on your system, for example:

   ```text
   3ABC246XYZ (Max Sample)
   ZABCW6SXYZ (SampleTech Inc.)
   ```

2. In Android Studio, open the `iosApp/Configuration/Config.xcconfig` and specify your Team ID.

Alternatively, choose the team in Xcode:

1. Go to Xcode and select **Open a project or file**.
2. Navigate to the `iosApp/iosApp.xcworkspace` file of your project.
3. In the left-hand menu, select `iosApp`.
4. Navigate to **Signing & Capabilities**.
5. In the **Team** list, select your team.

   If you haven't set up your team yet, use the **Add an Account** option in the **Team** list and follow Xcode instructions.

6. Make sure that the Bundle Identifier is unique and the Signing Certificate is successfully assigned.

##### Run the app

Connect your iPhone with a cable. If you already have the device registered in Xcode, Android Studio should show it
in the list of run configurations. Run the corresponding `iosApp` configuration.

If you haven't registered your iPhone in Xcode yet, follow [Apple recommendations](https://developer.apple.com/documentation/xcode/running-your-app-in-simulator-or-on-a-device/).
In short, you should:

1. Connect your iPhone with a cable.
2. On your iPhone, enable the developer mode in **Settings** | **Developer**.
3. In Xcode, go to the top menu and choose **Window** | **Devices and Simulators**.
4. Click on the plus sign. Select your connected iPhone and click **Add**.
5. Sign in with your Apple ID to enable development capabilities on the device.
6. Follow the on-screen instructions to complete the pairing process.

Once you've registered your iPhone in Xcode, [create a new run configuration](#run-on-a-new-ios-simulated-device)
in Android Studio and select your device in the **Execution target** list. Run the corresponding `iosApp` configuration.

</chunk>

### Run your application on desktop

You can create a run configuration for running the desktop application as follows:

1. Select the **Run | Edit Configurations** menu item.
2. Click the plus button and choose **Gradle** from the dropdown list.
3. In the **Tasks and arguments** field, paste this command:
   ```shell
   desktopRun -DmainClass=MainKt --quiet
   ```
4. Click **OK**.

Now, you can use this configuration to run the desktop app:

![Run the Compose Multiplatform app on desktop](compose-run-desktop-temp.png){width=350}

## Next step

In the next part of the tutorial, you'll learn how to implement composable functions and launch your application on each
platform.

**[Proceed to the next part](compose-multiplatform-explore-composables.md)**

## Get help

* **Kotlin Slack**. Get an [invite](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up) and join
  the [#multiplatform](https://kotlinlang.slack.com/archives/C3PQML5NU) channel.
* **Kotlin issue tracker**. [Report a new issue](https://youtrack.jetbrains.com/newIssue?project=KT).