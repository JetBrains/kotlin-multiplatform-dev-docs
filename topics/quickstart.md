[//]: # (title: Kotlin Multiplatform quickstart)

With this tutorial, you can get a simple Kotlin Multiplatform app up and running.

## Set up the environment

Kotlin Multiplatform projects need a specific environment,
but most of the requirements are made clear through preflight checks in the IDE.

Start with an IDE and necessary plugins:

1. Choose and install the IDE.
    KMP is supported in IntelliJ IDEA and Android Studio, so you can use the IDE you prefer.

    > The plugins necessary for Kotlin Multiplatform require IntelliJ IDEA 2025.1.1.1
    > or Android Studio Narwhal 2025.1.1 (TODO: this is a guess, check supported versions before publishing).
    >
    {style="note"}

2. Make sure you have all the necessary IDE plugins (TODO check if in IDEA dependencies are automated):

    * The [Kotlin Multiplatform](https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform) plugin (TODO: check the URL)

        > The Kotlin Multiplatform plugin is not yet available for IDEs on Windows or Linux.
        > TODO You can still follow the tutorial to learn about the alternatives. 

     * The [Native Debugging Support](https://plugins.jetbrains.com/plugin/12775-native-debugging-support) plugin
    * Android plugins need to be installed only if you use IDEA (they come bundled with Android Studio):
      * [Android](https://plugins.jetbrains.com/plugin/22989-android)
      * [Android Design Tools](https://plugins.jetbrains.com/plugin/22990-android-design-tools)
      * [Jetpack Compose](https://plugins.jetbrains.com/plugin/18409-jetpack-compose)

3. If you don't have the `ANDROID_HOME` environment variable set, configure your system to recognize it:

    <tabs>
    <tab title= "Bash or Zsh">
   
    Add the following command to your `.profile` or `.zprofile`:
        
    ```shell
    export ANDROID_HOME=~/Library/Android/sdk
    ```
   
    </tab>
    <tab title= "Windows Powershell or CMD">

    For Powershell, you can add a persistent environment variable with the following command (see [PowerShell docs](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_environment_variables)
    for details):

    ```shell
    [Environment]::SetEnvironmentVariable('ANDROID_HOME', '<path to the SDK>', 'Machine')
    ```

    For CMD, use the [`setx`](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/setx) command:
    
    ```shell
    setx ANDROID_HOME "<path to the SDK>"
    ```
    </tab>
    </tabs>

4. To create iOS applications, you also need a macOS host with [Xcode](https://apps.apple.com/us/app/xcode/id497799835) installed.
   Your IDE will run Xcode under the hood to build iOS frameworks.

    Make sure to launch Xcode at least once before starting to work with KMP projects so that it has a chance to
    go through the initial setup.

    > You will need to launch Xcode manually every time it is updated.
    > Preflight checks in the IDE you're using alert you whenever Xcode is not in the right state to work with.
    >
    {style="note"}

## Create a project 

For the quickstart we'll use the IDE wizard, but it's not available on Windows and Linux yet.
You can generate the same project as an IDE wizard using the [web KMP wizard](https://kmp.jetbrains.com/).
TODO: you can import and run the generated project on any host. Maybe link to running?

Both IntelliJ IDEA and Android Studio explicitly support creating a bare-bones Kotlin Multiplatform project:

<tabs>
<tab title= "IntelliJ IDEA">

1. Select **File** | **New** | **Project** in the main menu.
2. Choose **Kotlin Multiplatform** in the list on the left.
3. Set the name, location, and other base attributes of the project as needed.
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

When you're done choosing platforms, click the **Create** button and wait for the IDE to generate and import the project.

![IntelliJ IDEA Wizard with default settings and Android, iOS, desktop, and web platforms selected](idea-wizard-1step.png)

</tab>
<tab title= "Android Studio">

Before you start, make sure K2 mode is enabled: **Settings** | **Languages & Frameworks** | **Kotlin** | **Enable K2 mode**.
The Kotlin Multiplatform IDE plugin relies heavily on K2 functionality and is not going to work properly otherwise.

<!-- TODO check if K2 is actually required or strongly recommended -->

Use the wizard to create a new KMP project:

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

![Last step in the Android Studio wizard with Android, iOS, desktop, and web platforms selected](as-wizard-3step.png)

</tab>
</tabs>

## Run the sample apps

The project created by the IDE wizard includes pregenerated run configurations for iOS, Android,
desktop, and web applications, as well as Gradle tasks for running the server app.

### Run the Android app

TODO: align this with the CMP tutorial. There are a lot of details on running on a physical device, for example,
but that is not quick.

To run the Android app, start the composeApp run configuration:

![Dropdown with the Android run configuration highlighted](run-android-configuration.png)

TODO: collapsible Windows/Linux stuff about running the project

By default, it runs on the first available virtual device:

![Android app ran on a virtual device](run-android-app.png)

For details on running the Android app (adding virtual devices and setting up physical device connections) see
TODO link to the tutorial. 

### Run the iOS app

> You need a macOS host to build iOS apps.
>
{style="note"}

If you chose the iOS target for the project and set up a macOS machine with Xcode,
you can choose the **iosApp** run configuration and select a simulated device:

![Dropdown with the iOS run configuration highlighted](run-ios-configuration.png)

When you run the iOS app, it is built with Xcode under the hood and launched in the iOS Simulator.
The very first build collects native dependencies for compilation and warms up the build for subsequent runs:

![iOS app ran on a virtual device](run-ios-app.png)

### Run the desktop app

TODO: align this with the CMP tutorial.

The default run configuration for a desktop app is created as **composeApp [desktop]**:

![Dropdown with the default desktop run configuration highlighted](run-desktop-configuration.png)

With this configuration you can run the JVM desktop app:

![iOS app ran on a virtual device](run-desktop-app.png)

### Run the web app

TODO: align this with the CMP tutorial.

The default run configuration for a web app is created as **composeApp [wasmJs]**:

![Dropdown with the default Wasm run configuration highlighted](run-wasm-configuration.png)

When you run this configuration, the IDE builds the Kotlin/Wasm app and opens it in the default browser:

![iOS app ran on a virtual device](run-wasm-app.png)

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
* A series of tutorials about working with the shared UI code: [TODO current CMP tutorial chain]
* A series of tutorials about working with shared code along with native UI: [TODO current KMP tutorial chain]
* Take a deep dive into the Kotlin Multiplatform documentation TODO links:
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

