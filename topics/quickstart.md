[//]: # (title: Kotlin Multiplatform quickstart)

<web-summary>JetBrains provides official Kotlin IDE support for IntelliJ IDEA and Android Studio.</web-summary>

In this tutorial, you'll learn how to build and run a simple Kotlin Multiplatform app with a Compose Multiplatform UI.

## Set up the environment

Start with an IDE and necessary plugins:

1. Choose and install the IDE: Kotlin Multiplatform is fully supported in IntelliJ IDEA and Android Studio.
    
    The [JetBrains Toolbox App](https://www.jetbrains.com/toolbox/app/) is the recommended tool to install IDEs.
    It allows you to manage multiple products or versions, including
    [Early Access Program](https://www.jetbrains.com/resources/eap/) (EAP) and Nightly releases.

    For standalone installations, download the installer for [IntelliJ IDEA](https://www.jetbrains.com/idea/download/) 
    or [Android Studio](https://developer.android.com/studio).

    The plugins necessary for Kotlin Multiplatform require at least
    **IntelliJ IDEA 2025.2.2** or **Android Studio Otter 2025.2.1**.

2. Install the [Kotlin Multiplatform IDE plugin](https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform).
    
   The IDE plugin also installs any necessary dependencies your IDE does not have yet.
    
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

4. To create iOS applications, you need a macOS host with [Xcode](https://apps.apple.com/us/app/xcode/id497799835) installed.
    Your IDE will run Xcode under the hood to build iOS frameworks.

    Make sure to launch Xcode at least once before starting to work with KMP projects so that it goes
    through the initial setup.

    > You will need to launch Xcode manually every time it is updated and download the updated tooling.
    > The Kotlin Multiplatform IDE plugin makes preflight checks that alert you whenever Xcode is not in the right state to work with.
    >
    {style="note"}

## Create a project 

<tabs>
<tab title= "IntelliJ IDEA">

Use the IDE wizard to create a new KMP project:

1. Select **File** | **New** | **Project** in the main menu.
2. Choose **Kotlin Multiplatform** in the list on the left.
3. Set the name, location, and other base attributes of the project as needed.
4. We recommend selecting a version of [JetBrains Runtime](https://github.com/JetBrains/JetBrainsRuntime)
   (JBR) as the JDK for your project, as it provides important fixes, particularly for improving the compatibility
   of desktop KMP apps.
   Relevant versions of JBR are included in every IntelliJ IDEA distribution, so no additional setup is required.
5. To create a full demo, choose all available platforms: Android, iOS, Desktop, Web, and Server.
   Leave **Share UI** options selected where they are available to use Compose Multiplatform as the UI framework
   for the corresponding target.

   > The desktop target automatically includes [](compose-hot-reload.md) functionality that allows you to see UI changes
   > as soon as you save changes in your code.
   > Even if you're not planning on making a desktop app, you may want to add the desktop target to your project to speed up
   > iterating on UI code.
   > 
   {style="note"}

6. When you've chosen your platforms, click the **Create** button and wait for the IDE to generate and import the project.

![IntelliJ IDEA Wizard with default settings and Android, iOS, desktop, and web platforms selected](idea-wizard-1step.png){width=600}

</tab>
<tab title= "Android Studio">

Use the IDE wizard to create a new KMP project:

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

5. When you've chosen your platforms, click the **Finish** button and wait for the IDE to generate and import the project.

![Last step in the Android Studio wizard with Android, iOS, desktop, and web platforms selected](as-wizard-3step.png){width=600}

</tab>
</tabs>

## Consult the preflight checks

You can make sure there are no environment issues with the project setup by opening
the **Project Environment Preflight Checks** tool window:
click the preflight checks icon on the right sidebar or the bottom bar ![Project Environment Preflight Checks icon with a plane](ide-preflight-checks.png){width="20"}

In this tool window, you can see messages related to these checks, rerun them, or change their settings. 

Preflight checks commands are also available in the **Search Everywhere** dialog.
Press double <shortcut>Shift</shortcut> and search for commands containing the word "preflight":

![The Search Everywhere menu with the word "preflight" entered](double-shift-preflight-checks.png){width=600}

## Run the sample apps

The project created by the IDE wizard includes generated run configurations for iOS, Android,
desktop, and web applications, as well as Gradle tasks for running the server app.
The specific Gradle commands for each platform are listed below.

<tabs>
<tab title="Android">

To run the Android app, start the **composeApp** run configuration:

![Dropdown with the Android run configuration highlighted](run-android-configuration.png){width=250}

To create an Android run configuration manually, choose **Android App** as the run configuration template
and select the module **[project name].composeApp**.

By default, it runs on the first available virtual device:

![Android app ran on a virtual device](run-android-app.png){width=300}

</tab>
<tab title="iOS">

> You need a macOS host and Xcode installed to build iOS apps.
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

To create a desktop run configuration manually, choose a **Gradle** run configuration template and point to
the **[app name]:composeApp** Gradle project with the following command:

```shell
desktopRun -DmainClass=com.example.myapplication.MainKt --quiet
```

With this configuration you can run the JVM desktop app:

![JVM app ran on a virtual device](run-desktop-app.png){width=600}

</tab>
<tab title="Web">

The default run configuration for a web app is created as **composeApp [wasmJs]**:

![Dropdown with the default Wasm run configuration highlighted](run-wasm-configuration.png){width=250}

To create a web run configuration manually, choose a **Gradle** run configuration template and point to
the **[app name]:composeApp** Gradle project with the following command:

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

* Some tools may not find a Java version to run or use a wrong version.
  To solve this:
    * Set the `JAVA_HOME` environment variable to the directory where the appropriate JDK is installed.
  
      > We recommend using [JetBrains Runtime](https://github.com/JetBrains/JetBrainsRuntime),
      > an OpenJDK fork that supports class redefinition.
      >
      {style="note"}
  
    * Append the path to the `bin` folder inside your `JAVA_HOME` to the `PATH` variable,
      so that the tools included in JDK are available in the terminal.
* If you encounter issues with Gradle JDK in Android Studio, make sure it's configured correctly:
  select **Settings** | **Build, Execution, Deployment** | **Build Tools** | **Gradle**.

### Android tools

Same as for JDK, if you have trouble launching Android tools like `adb`,
make sure paths to `ANDROID_HOME/tools`, `ANDROID_HOME/tools/bin`, and
`ANDROID_HOME/platform-tools` are added to your `PATH` environment variable.

### Xcode

If your iOS run configuration reports that there is no virtual device to run on, or the preflight check fails, make sure to launch Xcode
and see if there are any updates for the iOS simulator.

### Get help

* **Kotlin Slack**. Get an [invite](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up) and join the [#multiplatform](https://kotlinlang.slack.com/archives/C3PQML5NU) channel.
* **Kotlin Multiplatform Tooling issue tracker**. [Report a new issue](https://youtrack.jetbrains.com/newIssue?project=KMT).

## What's next

Learn more about the structure of a KMP project and writing shared code:
* A series of tutorials about working with shared UI code using Compose Multiplatform: [](compose-multiplatform-create-first-app.md)
* A series of tutorials about working with shared code in a project with a native UI: [](multiplatform-create-first-app.md)
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

