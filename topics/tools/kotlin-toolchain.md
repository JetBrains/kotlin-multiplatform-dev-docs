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

### Install the Kotlin Toolchain CLI {id="toolchain-script-install"}

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

This adds `kotlin` as a command-line tool system-wide.

### Building iOS apps

If you intend to build and run iOS applications, you will need to install [Xcode](https://apps.apple.com/us/app/xcode/id497799835),
accept its license agreement, and install the necessary SDK.

Kotlin Toolchain can guide you (or your agent) through that process when it is actually required. 

## Generate a new project

With the `kotlin new` command, Kotlin Toolchain CLI can do the work for you, only asking for necessary input:

1. Run the command where you would like your project directory to be created:

   ```shell
   kotlin new
   ```

2. Enter the directory name when prompted for project path.
3. Select **Compose Multiplatform application** when presented with a choice of templates.
4. Confirm or alter the default choice of target platforms.
5. Provide a project ID that will be used across the project to identify the app (a default is generated based on the directory name).

Kotlin Toolchain generates the final project and initializes a Git repository.

The resulting project has several `*App` modules with application entry points for each platform and a `shared` module with common code.
Each module is listed in the overall `project.yaml` file and configured with its own `module.yaml` file.
Every application module explicitly depends on the shared module, for example:

```yaml
# androidApp/module.yaml
product: android/app

dependencies:
  # Shared module dependency
  - //shared
  # Android-specific dependency
  - $libs.androidx.activity.compose

settings:
  compose: enabled
  android:
    namespace: org.example.toolchainfirst
    applicationId: org.example.toolchainfirst
```

## Run the project

When the project is generated, you can switch to its directory and run it the `kotlin run` command (or [open it in the IDE](#work-on-a-project-in-intellij-idea-or-android-studio)).

For Kotlin Multiplatform applications, specify the exact application module you would like to run,
for example:

```shell
# Build and run the Android app
kotlin run -m android-app
```

When no module is specified,
the CLI lists all available application modules:

```shell
$ kotlin run
Multiple modules are available to run, please choose:
❯ desktopApp (with Hot Reload 🔥) 
androidApp
iosApp    
webApp
```

Kotlin Toolchain runs the appropriate build task and launches the application on the corresponding platform.

## Work on a project in IntelliJ IDEA or Android Studio

To work on the project directly, you can open it in IntelliJ IDEA or Android Studio.
You can pre-install the necessary plugins:

* [Kotlin Multiplatform plugin](https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform)
  is necessary for a KMP project to be supported properly 
* [Kotlin Toolchain plugin](https://plugins.jetbrains.com/plugin/31850-kotlin-toolchain)
  helps IDE recognize the Kotlin Toolchain project structure, generate run configurations, and so on.

### Create a project directly in the IDE

With the Kotlin Multiplatform and Kotlin Toolchain plugins installed,
you can also create a new project directly in the IDE:

1. Open IntelliJ IDEA or Android Studio.
2. Select **File** | **New** | **Project**.
3. Select **Kotlin Multiplatform** and choose **Kotlin Toolchain** in the **Build system** switch.
4. Fill in the rest of the project details and click **Create**.

When the project is created and imported, the IDE automatically registers run configuration for all declared modules
so you can run corresponding applications from the IDE toolbar.

## Publish the applications

When you're satisfied with how the app runs, you can publish the applications.

See the full instructions on producing artifacts in Kotlin Toolchain documentation:

* [Publishing an Android app](https://kotlin-toolchain.org/latest/user-guide/product-types/ios-app/#publishing)
* [Publishing an iOS app](https://kotlin-toolchain.org/latest/user-guide/product-types/android-app/#publishing)

You can also package a JVM app or a Wasm app, but publishing for these targets is not fully supported yet. 

## What's next

* For more on what Kotlin Toolchain is and what purpose it serves, check out the [product FAQ](https://kotlin-toolchain.org/dev/faq/).
* A [from-scratch tutorial](https://kotlin-toolchain.org/dev/getting-started/tutorial/)
  shows how to create a Kotlin Toolchain "Hello, World!" and gradually transform it into a multiplatform project
  with a complex templated configuration.
* For a deep dive into Kotlin Toolchain, check out the [user guide](https://kotlin-toolchain.org/%kotlinToolchainVersion%/getting-started/).
