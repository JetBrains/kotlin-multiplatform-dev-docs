[//]: # (title: Create a Kotlin Multiplatform project with Kotlin Toolchain)

Kotlin Toolchain suggests organizing and configuring a Kotlin Multiplatform project in a way that is different from Gradle,
but the main concepts and practices are the same.

The [Kotlin Toolchain documentation](https://kotlin-toolchain.org/dev/user-guide/multiplatform/)
has an overview of the general approach to Kotlin Multiplatform projects and the configuration reference.

This page shows how to create a Kotlin Multiplatform project with Kotlin Toolchain from scratch.

> To get a working example straightaway, see the [`compose-multiplatform` project in the Kotlin Toolchain repository](https://github.com/JetBrains/kotlin-toolchain/tree/main/examples/compose-multiplatform)  
>
{style="note"}

## Prerequisites

Before you start:

* To follow the instructions, [install Kotlin Toolchain](https://kotlin-toolchain.org/dev/) as an IntelliJ IDEA plugin or as a CLI.
* To build apps for Apple targets, you need an Apple machine with Xcode installed.
* To properly support multiplatform projects, you should also install the [Kotlin Multiplatform IDE plugin](https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform).

## Basic file setup

The overall configuration of the project is set up in the root `project.yaml` file,
individual modules – in their respective `module.yaml` files.

For a "Hello, world!" project that works on Android, iOS, and desktop, create the following files:

```text
├─ androidApp/
│  ├─ src/
│  │  ├─ main.kt
│  │  ╰─ AndroidManifest.xml
│  ╰─ module.yaml
├─ iosApp/
│  ├─ src/
│  │  ├─ iosApp.swift
│  │  ╰─ ViewController.kt
│  ╰─ module.yaml
├─ jvmApp/
│  ├─ src/
│  │  ╰─ main.kt
│  ╰─ module.yaml
├─ shared/
│  ├─ src/
│  │  ├─ App.kt
│  │  ╰─ Platform.kt
│  ├─ src@android/
│  │  ╰─ Platform.android.kt
│  ├─ src@ios/
│  │  ╰─ Platform.ios.kt
│  ├─ src@jvm/
│  │  ╰─ Platform.jvm.kt
╰─ project.yaml
```

Here the `android-app`, `ios-app`, and `jvm-app` directories are named that for clarity,
but the names of `shared/src@...` directories contain preset [platform qualifiers](https://kotlin-toolchain.org/dev/user-guide/multiplatform/#platform-qualifier)
that Kotlin Toolchain uses to mark platform-specific source files and configurations.

### Open the directory in IntelliJ IDEA

In IntelliJ IDEA, select **File | Open** and choose the directory with the files.
Since configuration files are empty, the project is not recognized fully as a Kotlin Toolchain project,
but the IDE can already help set it up.`

## Configure modules

The `project.yaml` file should hold the list of all modules:

```yaml
modules:
  - androidApp
  - desktopApp
  - iosApp
  - shared
```

Each mentioned module should be configured in its respective `module.yaml` file,
with application modules depending on the `shared` module.

As you start opening the `module.yaml` files, IntelliJ IDEA suggests downloading Kotlin wrappers since you haven't yet
set up Kotlin Toolchain for the project.
Click **Download wrappers**: this adds a `kotlin` and a `kotlin.bat` file to the project root and allows you to build the project.

Copy the following configurations to respective `module.yaml` files:
<tabs>
<tab title="Android">
<code-block language="yaml">
# android-app/module.yaml
product: android/app

dependencies:
- ../shared
# Set up integration of common UI code with the Android activity
- androidx.activity:activity-compose:1.13.0

settings:
compose: enabled
android:
namespace: org.example.project
applicationId: org.example.project
</code-block>
</tab>
<tab title="Desktop">
<code-block language="yaml">
# desktopApp/module.yaml
product: jvm/app

dependencies:
- ../shared
# Desktop dependency necessary for windowed applications
- $compose.desktop.currentOs

settings:
compose: enabled
jvm:
mainClass: org.example.project.MainKt
</code-block>
</tab>
<tab title="iOS">
<code-block language="yaml">
# iosApp/module.yaml
product:
    platforms: [iosArm64, iosSimulatorArm64]
    type: ios/app

dependencies:
    - ../shared
    - $compose.ui

settings:
    compose: enabled
</code-block>

<p>When the IDE recognizes an iOS module, it generates an Xcode project that can run be built and run on iOS devices.</p>
</tab>
<tab title="Shared">
<code-block language="yaml">
# shared/module.yaml
product:
    type: kmp/lib
    platforms: [ jvm, android, iosArm64, iosSimulatorArm64]

dependencies:
    - $compose.foundation
    - $compose.material3

settings:
    compose:
        enabled: true
</code-block>
</tab>
</tabs>

