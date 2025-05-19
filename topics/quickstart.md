[//]: # (title: Kotlin Multiplatform quickstart)

With this tutorial, you can get a simple Kotlin Multiplatform app up and running.

## Set up the environment

Kotlin Multiplatform projects need a specific environment,
but most of the requirements are made clear through preflight checks in the IDE.

Start with an IDE and necessary plugins:

1. Choose and install the IDE.
    KMP is supported in IntelliJ IDEA and Android Studio, so you can use the IDE you prefer.
    
    The plugins necessary for Kotlin Multiplatform require **IntelliJ IDEA 2025.1.1.1**
    or **Android Studio Narwhal 2025.1.1 Canary 10**.

2. Install the [Kotlin Multiplatform IDE plugin](https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform)
    (not to be confused with the Kotlin Multiplatform Gradle plugin).
   
    > The Kotlin Multiplatform plugin is not yet available for IDEs on Windows or Linux.
    > But it is also not strictly necessary on those platforms:
    > you can still follow the tutorial to generate and run a KMP project.
    >
    {style="note"}
    
3. Installing the Kotlin Multiplatform IDE plugin for IntelliJ IDEA also installs all necessary dependencies if you don't have them yet
    (Android Studio has all necessary plugins bundled).
    
    If you're using IntelliJ IDEA for Windows or Linux, make sure to install all necessary plugins manually:
    * [Android](https://plugins.jetbrains.com/plugin/22989-android)
    * [Android Design Tools](https://plugins.jetbrains.com/plugin/22990-android-design-tools)
    * [Jetpack Compose](https://plugins.jetbrains.com/plugin/18409-jetpack-compose)
    * [Native Debugging Support](https://plugins.jetbrains.com/plugin/12775-native-debugging-support)
    * [Compose Multiplatform IDE Support](https://plugins.jetbrains.com/plugin/16541-compose-multiplatform-ide-support)
      (only needed if you don't have the Kotlin Multiplatform plugin).

4. If you don't have the `ANDROID_HOME` environment variable set, configure your system to recognize it:

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

5. To create iOS applications, you need a macOS host with [Xcode](https://apps.apple.com/us/app/xcode/id497799835) installed.
    Your IDE will run Xcode under the hood to build iOS frameworks.

    Make sure to launch Xcode at least once before starting to work with KMP projects so that it goes
    through the initial setup.

    > You will need to launch Xcode manually every time it is updated and download the updated tooling.
    > The Kotlin Multiplatform IDE plugin makes preflight checks that alert you whenever Xcode is not in the right state to work with.
    >
    {style="note"}

## Create a project 

### On macOS

On macOS, the Kotlin Multiplatform plugin provides a project generation wizard inside the IDE:

<tabs>
<tab title= "IntelliJ IDEA">

Use the IDE wizard to create a new KMP project:

1. Select **File** | **New** | **Project** in the main menu.
2. Choose **Kotlin Multiplatform** in the list on the left.
3. Set the name, location, and other base attributes of the project as needed.
4. Choose platforms that you would like to see as part of the project:
    * All target platforms can be set up for using Compose Multiplatform to share UI code from the start
      (except for the server module that doesn't have UI code).
    * For iOS, you can choose one of two implementations:
        * shared UI code, with Compose Multiplatform,
        * fully native UI, made with SwiftUI and connected to the Kotlin module with shared logic.
    * The desktop target includes an alpha version of hot reload (TODO link?) functionality that allows you to see UI changes
      as soon as you alter corresponding code.
      Even if you're not planning on making desktop apps, you may want to use the desktop version to speed up
      writing UI code.

When you're done choosing platforms, click the **Create** button and wait for the IDE to generate and import the project.

![IntelliJ IDEA Wizard with default settings and Android, iOS, desktop, and web platforms selected](idea-wizard-1step.png){width=800}

</tab>
<tab title= "Android Studio">

The Kotlin Multiplatform IDE plugin relies heavily on K2 functionality and is not going to work as described without it.
So, before you start, make sure K2 mode is enabled:
**Settings** | **Languages & Frameworks** | **Kotlin** | **Enable K2 mode**.

Use the IDE wizard to create a new KMP project:

1. Select **File** | **New** | **New project** in the main menu.
2. Choose **Kotlin Multiplatform** in the default **Phone and Tablet** template category.
3. Set the name, location, and other base attributes of the project as needed, then click **Next**.
4. Choose platforms that you would like to see as part of the project:
    * All target platforms can be set up for using Compose Multiplatform to share UI code from the start
      (except for the server module that doesn't have UI code).
    * For iOS, you can choose one of two implementations: 
      * shared UI code, with Compose Multiplatform,
      * fully native UI, made with SwiftUI and connected to the Kotlin module with shared logic.  
    * The desktop target includes an alpha version of hot reload functionality that allows you to see UI changes
      as soon as you alter corresponding code.
      Even if you're not planning on making desktop apps, you may want to use the desktop version to speed up
      writing UI code.

When you're done choosing platforms, click the **Finish** button and wait for the IDE to generate and import the project.

![Last step in the Android Studio wizard with Android, iOS, desktop, and web platforms selected](as-wizard-3step.png){width=800}

</tab>
</tabs>

### On Windows or Linux

If you're on Windows or Linux:

1. Generate a project using the [web KMP wizard](https://kmp.jetbrains.com/).
2. Extract the archive and open the resulting folder in your IDE.
3. Wait for the import to finish, then go to the [](#run-the-sample-apps) section to learn how to build and run the apps.

Since the Kotlin Multiplatform IDE plugin does not support Windows or Linux yet

## Consult the preflight checks

You can make sure there are no environment issues with the project setup by opening
the **Project Environment Preflight Checks** tool window:
click the preflight checks icon on the right sidebar or the bottom bar ![Preflight checks icon with a plane](ide-preflight-checks.png){width="20"}

In this tool window, you can see the messages related to these checks, rerun them, or change their settings. 

Preflight checks commands are also available in the **Search Everywhere** dialog.
Press double **Shift** and search for commands containing the word "preflight":

![The Search Everywhere menu with the word "preflight" entered](double-shift-preflight-checks.png)

## Run the sample apps

The project created by the IDE wizard includes generated run configurations for iOS, Android,
desktop, and web applications, as well as Gradle tasks for running the server app.
On Windows and Linux, see Gradle commands for each platform below.

<tabs>
<tab title="Android">

To run the Android app, start the **composeApp** run configuration:

![Dropdown with the Android run configuration highlighted](run-android-configuration.png){width=250}

To run the Android app on Windows or Linux, create an **Android App** run configuration
and choose the module **[project name].composeApp**.

By default, it runs on the first available virtual device:

![Android app ran on a virtual device](run-android-app.png){width=350}

For details on running the Android app (adding virtual devices and setting up physical device connections) see
[](compose-multiplatform-create-first-app.md).

</tab>
<tab title="iOS">

> You need a macOS host to build iOS apps.
>
{style="note"}

If you chose the iOS target for the project and set up a macOS machine with Xcode,
you can choose the **iosApp** run configuration and select a simulated device:

![Dropdown with the iOS run configuration highlighted](run-ios-configuration.png){width=250}

When you run the iOS app, it is built with Xcode under the hood and launched in the iOS Simulator.
The very first build collects native dependencies for compilation and warms up the build for subsequent runs:

![iOS app ran on a virtual device](run-ios-app.png){width=350}

</tab>
<tab title="Desktop">

The default run configuration for a desktop app is created as **composeApp [desktop]**:

![Dropdown with the default desktop run configuration highlighted](run-desktop-configuration.png){width=250}

To run the desktop app on Windows or Linux, create a **Gradle** run configuration pointing
to the **[app name]:composeApp** Gradle project with the following command:

```shell
desktopRun -DmainClass=com.example.myapplication.MainKt --quiet
```

With this configuration you can run the JVM desktop app:

![JVM app ran on a virtual device](run-desktop-app.png){width=600}

</tab>
<tab title="Web">

The default run configuration for a web app is created as **composeApp [wasmJs]**:

![Dropdown with the default Wasm run configuration highlighted](run-wasm-configuration.png){width=250}

To run the web app on Windows or Linux, create a **Gradle** run configuration pointing
to the **[app name]:composeApp** Gradle project with the following command:

```shell
wasmJsBrowserDevelopmentRun
```

When you run this configuration, the IDE builds the Kotlin/Wasm app and opens it in the default browser:

![Web app ran on a virtual device](run-wasm-app.png){width=600}

</tab>
</tabs>

## Troubleshooting

### Java and JDK

Common issues with Java:

* Some tools may not find a Java version to run, or use a wrong version.
  To solve this:
    * Set the `JAVA_HOME` environment variable to the directory where the appropriate JDK is installed.
    * Append the path to the `bin` folder inside your `JAVA_HOME` to the `PATH` variable,
      so that the tools included in JDK are available in the terminal.
* If you encounter issues with Gradle JDK in Android Studio, make sure it's configured correctly:
  select **Settings** | **Build, Execution, Deployment** | **Build Tools** | **Gradle**.

### Android tools

Same as for JDK: if you have trouble launching Android tools like `adb`,
make sure paths to `ANDROID_HOME/tools`, `ANDROID_HOME/tools/bin`, and
`ANDROID_HOME/platform-tools` are added to your `PATH` environment variable.

### Xcode

If your iOS run configuration reports that there is no virtual device to run on, make sure to launch Xcode
and see if there are any updates for the iOS simulator.

### Get help

* **Kotlin Slack**. Get an [invite](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up) and join the [#multiplatform](https://kotlinlang.slack.com/archives/C3PQML5NU) channel.
* **Kotlin Multiplatform Tooling issue tracker**. [Report a new issue](https://youtrack.jetbrains.com/newIssue?project=KMT).

## What's next

Learn more about the structure of a KMP project and writing shared code:
* A series of tutorials about working with the shared UI code: [](compose-multiplatform-create-first-app.md)
* A series of tutorials about working with shared code along with a native UI: [](multiplatform-create-first-app.md)
* Take a deep dive into the Kotlin Multiplatform documentation:
  * [Project configuration](multiplatform-project-configuration.md)
  * [Working with multiplatform dependencies](https://kotlinlang.org/docs/multiplatform-add-dependencies.html)
* Learn about the Compose Multiplatform UI framework, its fundamentals, and platform-specific features:
    [](compose-multiplatform-and-jetpack-compose.md).

Discover code already written for KMP:
* Our [Samples](multiplatform-samples.md) page, with official JetBrains samples as well as a curated list of
    projects showcasing KMP capabilities.
* The GitHub topics:
  * [kotlin-multiplatform](https://github.com/topics/kotlin-multiplatform), projects implemented with Kotlin Multiplatform.
  * [kotlin-multiplatform-sample](https://github.com/topics/kotlin-multiplatform-sample),
      a list of sample projects written with KMP.
* [klibs.io](https://klibs.io) – search platform for KMP libraries, with more than 2000 libraries indexed so far,
    including OkHttp, Ktor, Coil, Koin, SQLDelight, and others.

