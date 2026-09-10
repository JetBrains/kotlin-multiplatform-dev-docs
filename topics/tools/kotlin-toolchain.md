[//]: # (title: Creating and building a Kotlin Multiplatform project with Kotlin Toolchain)

[Kotlin Toolchain](https://kotlin-toolchain.org/) is a tool created by JetBrains to help you configure projects
for building, packaging, publishing, and more. With Kotlin Toolchain, you can spend less time dealing with build systems and focus
on addressing real business challenges instead.

Kotlin Toolchain is designed to support Kotlin Multiplatform applications as well,
aware of all platforms supported by the framework and making cross-platform configuration simple.

> Kotlin Toolchain is in [Alpha](supported-platforms.md#general-kotlin-stability-levels).
> You're welcome to try it in your Kotlin Multiplatform projects.
> We would appreciate your feedback in [YouTrack](https://youtrack.jetbrains.com/issues/AMPER).
>
{style="note"}

Kotlin Toolchain provides IDE support with plugins for IntelliJ IDEA and Android Studio,
it is focused on providing a smooth CLI experience that helps AI agents to interact with the build system
predictably and transparently.

This page guides you through setting up a Kotlin Multiplatform project from scratch,
referencing the comprehensive [Kotlin Toolchain documentation](https://kotlin-toolchain.org/%kotlinToolchainVersion%/user-guide/)
for further information.

## Prerequisites

<!--<include from="kotlin-toolchain.md" element-id="toolchain-script-install"/>-->

### CLI {id="toolchain-script-install"}

You can install the Kotlin Toolchain CLI using [SDKMAN!](https://sdkman.io/):

```shell
sdk install kotlintoolchain
```

Or via the installer script:

<tabs>
<tab title="macOS or Linux">

```shell
curl -fsSL https://kotl.in/install.sh | sh
```

</tab>

<tab title="Windows">

```shell
powershell -ExecutionPolicy ByPass -c "irm 'https://kotl.in/install.ps1' | iex"
```

</tab>
</tabs>

This adds `kotlin` as a command-line tool.

### Simulators and emulators

If you intend to build and run iOS applications, make sure to install Xcode and run it at least once
to accept terms of use as well as select and download an iOS SDK.

To build and run Android applications, make sure to have [Android SDK Tools](https://developer.android.com/tools/releases/platform-tools)
installed.

### JDK

Kotlin Toolchain automatically provisions JDK 25 if it's not available in your `JAVA_HOME`,
so you don't need to do anything to run the tool.

You can, however, [customize the provisioning behavior when necessary](https://kotlin-toolchain.org/0.12/user-guide/advanced/jdk-provisioning/).

## Generate a new project

1. Create a new empty directory and open it in your terminal.
2. Run the following command to initialize a new Kotlin Toolchain project:

    ```shell
    kotlin init compose-multiplatform
    ```

The `compose-multiplatform` argument directs the generator to create a Kotlin Multiplatform project
with Compose Multiplatform UI shared across platforms.

The resulting project has several `*App` modules with application entry points for each platform and a `shared` module with common code.
The modules are listed in the `project.yaml` file and configured in their own `module.yaml` file.
Every application module explicitly depends on the shared module, for example:

```yaml
product: android/app

dependencies:
  # Shared module dependency
  - ../shared
  # Android-specific dependency
  - androidx.activity:activity-compose:1.7.2

settings:
  compose: enabled
  junit: junit-4
```

## Run the project

You can build a project with the `kotlin build` command and run it the `kotlin run` command.
For Kotlin Multiplatform applications, specify the exact application module you would like to run,
for example:

```shell
# Build and run the Android app
$ kotlin run -m android-app

# When no module is specified, the tool asks you to choose one
$ kotlin run
Multiple modules are available to run, please choose:
❯ androidApp
  desktopApp
  iosApp    
  webApp 
```

Kotlin Toolchain runs the appropriate build task and launches the application on the corresponding platform.

## Refine the project in the IDE

To fine-tune the project directly, you can open it in IntelliJ IDEA or Android Studio.
You can pre-install the necessary plugins:

* [Kotlin Toolchain plugin](https://plugins.jetbrains.com/plugin/31850-kotlin-toolchain)
* [Kotlin Multiplatform plugin](https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform)

They enable code completion and navigation for the Kotlin Toolchain configuration files
and automatically import declared modules as IDE run configurations.

### Create a project directly in the IDE

With the Kotlin Multiplatform and Kotlin Toolchain plugins installed,
you can also create a new project directly in the IDE:

1. Open IntelliJ IDEA or Android Studio.
2. Select **File** | **New** | **Project**.
3. Select **Kotlin Multiplatform** and choose **Kotlin Toolchain** in the **Build system** switch.
4. Fill in the rest of the project details and click **Create**.

The resulting project is similar to the one created with the [`kotlin init` command](#generate-a-new-project).

When the project is created and imported, the IDE automatically registers run configuration for all declared modules
so you can run corresponding applications from the IDE toolbar.

<!-- ## Publish the artifacts

When you're satisfied with how the app runs, you can publish the applications.

TODO link to the Toolchain page on multiplatform app publishing -->

## What's next

* For more on what Kotlin Toolchain is and what purpose it serves, check out the [product FAQ](https://kotlin-toolchain.org/dev/faq/).
* A [from-scratch tutorial](https://kotlin-toolchain.org/dev/getting-started/tutorial/)
  shows how to create a Kotlin Toolchain "Hello, World!" and gradually transform it into a multiplatform project
  with a complex templated configuration.
* For a deep dive into Kotlin Toolchain, check out the [user guide](https://kotlin-toolchain.org/%kotlinToolchainVersion%/getting-started/).