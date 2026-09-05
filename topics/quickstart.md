[//]: # (title: Kotlin Multiplatform quickstart)

<web-summary>JetBrains provides official Kotlin IDE support for IntelliJ IDEA and Android Studio.</web-summary>

<secondary-label ref="IntelliJ IDEA"/>
<secondary-label ref="Android Studio"/>

In this tutorial, you'll learn how to build and run a simple Kotlin Multiplatform (KMP) app with a Compose Multiplatform UI.

## Set up the environment

Set up the IDE, `ANDROID_HOME`, and Xcode:

1. Choose and install the IDE: KMP is fully supported in IntelliJ IDEA and Android Studio.
    
    The [JetBrains Toolbox App](https://www.jetbrains.com/toolbox/app/) is the recommended tool to install IDEs.
    For standalone installations, download the installer for [IntelliJ IDEA](https://www.jetbrains.com/idea/download/) 
    or [Android Studio](https://developer.android.com/studio).

    For best results, use the latest stable version of the IDE.

2. Install the Kotlin Multiplatform IDE plugin.
   You can find it in the plugin marketplace (**Settings | Plugins | Marketplace**)
   or install it from the [plugin web page](https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform).
   
   You can find the plugin  
    
3. If you don't have the `ANDROID_HOME` environment variable set, configure your system to recognize it:

    <tabs>
    <tab title= "Bash or Zsh">
   
    Add the following command to your `.profile` or `.zprofile`:
        
    ```shell
    export ANDROID_HOME=~/Library/Android/sdk
    ```
   
    </tab>
    <tab title= "Windows PowerShell or CMD">

    For PowerShell, you can add a persistent environment variable with the following command
    (see [PowerShell docs](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_environment_variables) for details):

    ```shell
    [Environment]::SetEnvironmentVariable('ANDROID_HOME', '<path to the SDK>', 'Machine')
    ```

    For CMD, use the [`setx`](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/setx) command:
    
    ```shell
    setx ANDROID_HOME "<path to the SDK>"
    ```
    </tab>
    </tabs>

4. To create iOS applications, you need a macOS machine with [Xcode](https://apps.apple.com/us/app/xcode/id497799835) installed.
    Your IDE runs Xcode under the hood to build iOS frameworks.

    Make sure to launch Xcode at least once before starting to work with KMP projects so that it goes
    through the initial setup.

    > You have to launch Xcode manually every time it's updated and download the updated tooling.
    > The Kotlin Multiplatform IDE plugin makes preflight checks that alert you whenever Xcode is not in the right state to work with.
    >
    {style="note"}

## Create a project 

<tabs>
<tab title= "IntelliJ IDEA">

Use the Kotlin Multiplatform generator to create a project:

1. Select **File** | **New** | **Project** in the main menu.
2. Choose **Kotlin Multiplatform** in the list on the left.
3. Set **Name** and **Location** as you see fit.
   **Project ID** is generated based on the name.
4. To create a full demo, select all available platforms: Android, iOS, Desktop, Web, and Server.
   In **UI implementation** options, leave **Share UI** selected to use Compose Multiplatform as the UI framework
   for the corresponding target.

   > The desktop target automatically includes [](compose-hot-reload.md) functionality which allows you to see UI changes
   > as soon as you save changes in your code.
   > Even if you're not planning on making a desktop app, you may want to add the desktop target to your project to speed up
   > iterating on UI code.
   > 
   {style="note"}

5. Click the **Create** button and wait for the IDE to generate and import the project.

![IntelliJ IDEA Wizard with default settings and Android, iOS, desktop, and web platforms selected](idea-wizard-1step.png){width=600}

</tab>
<tab title= "Android Studio">

Use the wizard to create a new project:

1. Select **File** | **New** | **New project** in the main menu.
2. Choose **Kotlin Multiplatform** in the default **Phone and Tablet** template category.

    ![First new project step in Android Studio](as-wizard-1.png){width="400"}

3. Set the name, location, and other base attributes of the project as needed, then click **Next**.
4. To create a full demo, choose all available platforms: Android, iOS, Desktop, Web, and Server.
   Leave **Share UI** options selected where they are available to use Compose Multiplatform as the UI framework
   for the corresponding target.

   > The desktop target automatically includes [](compose-hot-reload.md) functionality that allows you to see UI changes
   > as soon as you save changes in your code.
   > Even if you're not planning on making a desktop app, you may want to add the desktop target to your project to speed up
   > iterating on UI code.
   >
   {style="note"}

5. Click the **Finish** button and wait for the IDE to generate and import the project.

![Last step in the Android Studio wizard with Android, iOS, desktop, and web platforms selected](as-wizard-3step.png){width=600}

</tab>
</tabs>

You can find the code being shared between platforms in the `shared` module.
The `Platform.kt` file contains an [`expect`](multiplatform-expect-actual.md) declaration
that sets up requesting the platform name in the native code.

When you run the various apps, you can see the same UI layout with different platform names supplied by the native calls:

## Consult the preflight checks

To make sure there are no environment issues with the project setup,
open the opening the **Project Environment Preflight Checks** tool window:
click the preflight checks icon on the right sidebar or the bottom bar ![Project Environment Preflight Checks icon with a plane](ide-preflight-checks.png){width="20"}

In this tool window, you can see which checks passed, rerun them, or change their settings.
Normally, the window opens automatically when a problem is detected and stays hidden otherwise.

Preflight checks commands are also available in the **Search Everywhere** dialog.
Press double <shortcut>Shift</shortcut> and search for commands containing the word "preflight":

![The Search Everywhere menu with the word "preflight" entered](double-shift-preflight-checks.png){width=600}

## Modules in the generated project

Depending on the set of platforms you select in the Kotlin Multiplatform wizard,
you can see the following modules after the project you created is imported:

* **androidApp** is the module that builds the Android application.
* **desktopApp** is the module that builds the desktop JVM application.
* **iosApp** is an Xcode project that builds the iOS application. It depends on and uses the **shared** module as an iOS
  framework.
* **shared** is a Kotlin Multiplatform module that contains the code common for the Android, desktop, iOS, and web applications.
* **webApp** is the module that builds web applications, both Kotlin/JS and Kotlin/Wasm.
* **server** and **core** modules are created only for the server platform:
  **core** holds code shared between the server and client apps,
  **server** configures an endpoint.

  > When the server platform is selected, the IDE groups the application entry points under the `app` directory.
  > Otherwise, the app modules are created in the project root.
  >
  {style="tip"} 

The `shared` module is built appropriately for each target.
For example, it's treated as a Kotlin/JVM module when building the Android app and as Kotlin/Native when building an iOS app.

## Run the sample apps

The project created by the IDE wizard includes generated run configurations for iOS, Android,
desktop, and web applications, as well as Gradle tasks for running the server app.
The specific Gradle commands for each platform are listed below.

To start a run configuration, find the dropdown menu on the top right of the IDE
and click the **Run** button:

<tabs>
<tab title="Android">

To run the Android app, start the **androidApp** run configuration:

![Dropdown with the Android run configuration highlighted](run-android-configuration.png){width=250}

By default, it runs on the first available virtual device:

![Android app ran on a virtual device](run-android-app.png){width=300}

To create an Android run configuration manually (**Run | Edit Configurations**),
choose **Android App** as the run configuration template and select the module **[project name].androidApp**.

</tab>
<tab title="iOS">

> You need a macOS machine with Xcode installed to build iOS apps.
>
{style="note"}

Select the **iosApp** run configuration and a simulated device:

![Dropdown with the iOS run configuration highlighted](run-ios-configuration.png){width=250}

The run configuration builds the iOS app with Xcode under the hood and launches it using the iOS Simulator.
The first build collects native dependencies and caches build data to make subsequent runs faster:

![iOS app run on a virtual device](run-ios-app.png){width=350}

</tab>
<tab title="Desktop">

The default run configuration for a desktop app is created as **desktopApp [hot] 🔥**:

![Dropdown with the default desktop run configuration highlighted](run-desktop-configuration.png){width=250}

With this configuration, you can run the JVM desktop app:

![JVM app ran on a virtual device](run-desktop-app.png){width=600}

To create a desktop run configuration with Hot Reload manually (**Run | Edit Configurations**),
choose the **Gradle** run configuration template and point to the **[app name]:desktopApp** Gradle project with the following command:

```shell
hotRun --mainClass "com.example.demo.MainKt"
```

</tab>
<tab title="Web">

By default, two run configurations are created for the web: **webApp [wasmJs]** and **webApp [js]**.
Both run the same app, built with Kotlin/Wasm or Kotlin/JS respectively:

![Dropdown with the default Wasm run configuration highlighted](run-wasm-configuration.png){width=250}

When you run this configuration, the IDE builds the Kotlin/Wasm app and opens it in the default browser:

![Web app ran on a virtual device](run-wasm-app.png){width=600}

To create a web run configuration manually, choose a **Gradle** run configuration template and point to
the **[app name]:webApp** Gradle project with the `wasmJsBrowserDevelopmentRun` task,
or `jsBrowserDevelopmentRun` for the Kotlin/JS version.
</tab>
</tabs>

## Troubleshooting

Issues with Kotlin Multiplatform setup usually occur when Java, Android SDK, or Xcode are not configured correctly.  

### Java and JDK

Most common issues related to Java configuration:

* Some tools may not find a Java version to run or use a wrong version.
  To solve this, set the `JAVA_HOME` environment variable to the directory where the appropriate JDK is installed
  (we recommend using [JetBrains Runtime](https://github.com/JetBrains/JetBrainsRuntime)),
  then append the path to the `bin` folder inside your `JAVA_HOME` to the `PATH` variable.
* If you encounter issues with Gradle JDK in Android Studio, make sure it's configured correctly:
  select **Settings** | **Build, Execution, Deployment** | **Build Tools** | **Gradle**.

### Android tools

If you have trouble launching Android tools like `adb`,
make sure paths to `ANDROID_HOME/tools`, `ANDROID_HOME/tools/bin`, and
`ANDROID_HOME/platform-tools` are added to your `PATH` environment variable.

### Xcode

If your iOS run configuration reports that there is no virtual device to run on, or the preflight check fails, make sure to launch Xcode
and see if there are any updates for the iOS simulator.

### Get help

* **Kotlin Slack**: Get an [invite](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up) and join the [#multiplatform](https://kotlinlang.slack.com/archives/C3PQML5NU) channel.
* **Kotlin Multiplatform Tooling issue tracker**: [Report a new issue](https://youtrack.jetbrains.com/newIssue?project=KMT).

## What's next

Learn more about the structure of a KMP project and writing shared code:
* [](compose-multiplatform-new-project.md): A beginner-level tutorial that teaches to work with shared UI code using Compose Multiplatform.
* [](multiplatform-upgrade-app.md)A beginner-level tutorial that teaches to work with shared code in a multiplatform project
  with native UI code. 

Take a deep dive into specific Kotlin Multiplatform use cases:
* [Working with multiplatform dependencies](multiplatform-add-dependencies.md)
* [Organizing code and artifacts around multiplatform artifacts](multiplatform-project-configuration.md)
* Learn about the Compose Multiplatform UI framework and its place in the Compose ecosystem: [](compose-multiplatform-and-jetpack-compose.md)

Discover code already written for KMP:
* [Samples](multiplatform-samples.md): official JetBrains samples along with a curated list of projects showcasing KMP capabilities.
* The GitHub topics:
  * [kotlin-multiplatform](https://github.com/topics/kotlin-multiplatform): Projects implemented with Kotlin Multiplatform.
  * [kotlin-multiplatform-sample](https://github.com/topics/kotlin-multiplatform-sample): A list of sample projects written with KMP.
* [klibs.io](https://klibs.io): Search platform for KMP libraries.
  It indexes projects from GitHub and artifacts from Maven Central, allows for fine filtering of search results,
  and provides [support for AI workflows](https://klibs.io/ai).
