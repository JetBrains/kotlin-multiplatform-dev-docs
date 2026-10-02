[//]: # (title: Create and build a Kotlin Multiplatform application with Kotlin Toolchain)


[Kotlin Toolchain](https://kotlin-toolchain.org/) is a tool from JetBrains for creating, building, testing,
and running Kotlin projects.
It provides a CLI and declarative configuration, so you can work from a terminal, an IDE,
or with AI-assisted development tools.

This page guides you through setting up a Kotlin Multiplatform project from scratch using Kotlin Toolchain.

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

# Restart your terminal or run this command
# to make 'kotlin' available
exec $SHELL
```

</tab>

<tab title="Windows">

```shell
powershell -ExecutionPolicy ByPass -c "irm 'https://kotl.in/install.ps1' | iex"
```

</tab>
</tabs>

Check that the CLI is available by running `kotlin --version`.

### Building iOS apps

To build and run iOS applications, install [Xcode](https://apps.apple.com/us/app/xcode/id497799835)
and the necessary SDK.

When it is actually required to build or run a module,
Kotlin Toolchain CLI displays instructions on how to set Xcode up.

## Create a project

To generate a new project using Kotlin Toolchain:

1. Navigate to the directory where you would like your project directory to be created.
2. Run the following command:

   ```shell
   kotlin new
   ```

3. Enter the directory name when prompted for the project path, for example, `ktc-kmp`.
4. Select **Compose Multiplatform application** when presented with a choice of templates.
5. Press **Enter** to confirm the default selection of targets.
6. Provide a project ID that will be used across the project to identify the app (a default is generated based on the directory name).
   This ID is used for Kotlin package names, the Android namespace and application ID, and the iOS bundle ID.

Kotlin Toolchain generates the project, including configuration files, source code, and wrapper scripts.
By default, it also initializes a Git repository.

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

The basic structure of each module follows the [KMP source set model](multiplatform-discover-project.md#source-sets),
only instead of `androidMain`, for example, it's `src@android`.

## Run the project

To run the project:

1. Navigate to the project directory (`ktc-kmp` in the example above).
2. Run `kotlin run` to bring up the list of available applications to run.
   The available modules correspond to the targets you selected when generating the project:

    ```shell
    $ kotlin run
    
    Multiple modules are available to run, please choose:
    ❯ desktopApp (with Hot Reload 🔥)
    androidApp
    iosApp    
    webApp
    ```

3. You can also run a module directly by using the `-m` (`--module`) option, for example:

    ```shell
    # Build and run the desktop JVM app
    kotlin run -m desktopApp
    ```

## Work on a project in IntelliJ IDEA or Android Studio

You can work on the project and run it in IntelliJ IDEA or Android Studio.
Install the following plugins to make your IDE recognize Kotlin Toolchain and Kotlin Multiplatform projects:

* [Kotlin Multiplatform plugin](https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform)
  is necessary for a KMP project to be supported properly.
* [Kotlin Toolchain plugin](https://plugins.jetbrains.com/plugin/31850-kotlin-toolchain)
  helps the IDE recognize the Kotlin Toolchain project structure, generate run configurations, and so on.

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
* For a deep dive into Kotlin Toolchain, check out the [user guide](https://kotlin-toolchain.org/latest/user-guide/).
