[//]: # (title: Creating and building a Kotlin Multiplatform application with Kotlin Toolchain)

[Kotlin Toolchain](https://kotlin-toolchain.org/) supports the full project lifecycle out of the box, from creation to publication, so you can focus on real business challenges. It is optimized for the agentic and CLI workflows, which helps AI agents to interact with the toolchain reliably. Kotlin Toolchain also provides traditional IDE support with plugins for IntelliJ IDEA and Android Studio.

This page guides you through setting up a Kotlin Multiplatform project from scratch,
referencing the comprehensive [Kotlin Toolchain documentation](https://kotlin-toolchain.org/latest/user-guide/)
for further information.

> Kotlin Toolchain is in [Alpha](supported-platforms.md#general-kotlin-stability-levels).
> You're welcome to try it in your Kotlin Multiplatform projects.
> We would appreciate your feedback in [YouTrack](https://youtrack.jetbrains.com/issues/KTC).
>
{style="note"}

## Prerequisites

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

### Building iOS apps

If you intend to build and run iOS applications, make sure to install [Xcode](https://apps.apple.com/us/app/xcode/id497799835)
and run it at least once to accept terms of use as well as select and download an iOS SDK.

<!-- May becoma more straightforward after https://youtrack.jetbrains.com/issue/KTC-679 -->

## Generate a new project

1. Create a new empty directory and open it in your terminal.
   <!-- Should change after https://youtrack.jetbrains.com/issue/KTC-5478 -->
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

# When no module is specified,
# the tool lists the available application modules
$ kotlin run

ERROR: There are several matching application modules in the project. Please specify one with the '--platform' or '--module' option.

Runnable application modules:
  android-app: android
  ios-app: iosArm64 iosSimulatorArm64 iosX64
  jvm-app: jvm
```

Kotlin Toolchain runs the appropriate build task and launches the application on the corresponding platform.

## Work on a project in IntelliJ IDEA or Android Studio

To work on the project directly, you can open it in IntelliJ IDEA or Android Studio.
You can pre-install the necessary plugins:

* [Kotlin Toolchain plugin](https://plugins.jetbrains.com/plugin/31850-kotlin-toolchain)
* [Kotlin Multiplatform plugin](https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform)

They enable code completion and navigation for the Kotlin Toolchain configuration files
and automatically import declared modules as IDE run configurations.

<!-- TODO clarify the plugin features before the next release --> 

### Create a project directly in the IDE

With the Kotlin Multiplatform and Kotlin Toolchain plugins installed,
you can also create a new project directly in the IDE:

1. Open IntelliJ IDEA or Android Studio.
2. Select **File** | **New** | **Project**.
3. Select **Kotlin Multiplatform** and choose **Kotlin Toolchain** in the **Build system** switch.
4. Fill in the rest of the project details and click **Create**.

The resulting project is a little richer than a simple "Hello, World!":
it has a base Compose layout and an example of integrating resources.

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
