[//]: # (title: Create a Kotlin Multiplatform project with Kotlin Toolchain)

Kotlin Toolchain suggests organizing and configuring a Kotlin Multiplatform project in a way that is different from Gradle,
but the main concepts and practices are the same.

The [Kotlin Toolchain documentation](https://kotlin-toolchain.org/dev/user-guide/multiplatform/)
has an overview of the general approach to Kotlin Multiplatform projects and the configuration reference.

This page shows how to create a Kotlin Multiplatform project with Kotlin Toolchain from scratch.

> Using the latest IntelliJ IDEA EAP with the [Kotlin Toolchain plugin](https://plugins.jetbrains.com/plugin/31850-kotlin-toolchain),
> you can create a working Kotlin Multiplatform project with the Kotlin Toolchain using the **New Project** wizard.
> This page walks you through setting up a project from the ground up.
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
│  │  ╰─ iosApp.swift
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
│  │  ├─ Platform.ios.kt
│  │  ╰─ ViewController.kt
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

## Configure modules within the project

### List modules in `project.yaml`

The `project.yaml` file should hold the list of all modules (directories where a `module.yaml` is expected):

```yaml
modules:
  - androidApp
  - desktopApp
  - iosApp
  - shared
```

### Configure individual modules

Each module mentioned in the `project.yaml` file should be configured in its respective `module.yaml` file,
with application modules depending on the `shared` module.

As you start opening the `module.yaml` files, IntelliJ IDEA suggests downloading Kotlin wrappers since you haven't yet
set up Kotlin Toolchain for the project.
Click **Download wrappers**: this adds a `kotlin` and a `kotlin.bat` file to the project root and allows you to build the project.

For Kotlin CLI, run `kotlin update` in the project root: This adds the wrapper if it's not there and updates Kotlin Toolchain
to the latest version.

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

## Add source code

### Write common code

To the `shared/src/App.kt` file, add the `App()` composable that will be responsible for the UI across platforms:

```kotlin
import androidx.compose.foundation.background
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.safeContentPadding
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier

@Composable
fun App() {
    MaterialTheme {
        Column(
            modifier = Modifier
                .background(MaterialTheme.colorScheme.primaryContainer)
                .safeContentPadding()
                .fillMaxSize(),
            horizontalAlignment = Alignment.CenterHorizontally,
        ) {
            Text("Hello from ${getPlatform()}")
        }
    }
}
```
{collapsible="true" collapsed-title-line-number="21"}

The `getPlatform()` call doesn't resolve yet: This is the function that will have [expect / actual definitions](multiplatform-expect-actual.md)
to provide platform-specific system name for each application.

Fill in the `getPlatform()` definitions in their respective directories under `shared/`:

<tabs>
<tab title="Platform.kt">
<code-block language="kotlin">
expect fun getPlatform(): String
</code-block>
</tab>
<tab title="Platform.android.kt">
<code-block language="kotlin">
import android.os.Build

actual fun getPlatform(): String = "Android ${Build.VERSION.RELEASE} (API ${Build.VERSION.SDK_INT})"
</code-block>
</tab>
<tab title="Platform.ios.kt">
<code-block language="kotlin">
import platform.UIKit.UIDevice

actual fun getPlatform(): String =
"${UIDevice.currentDevice.systemName()} ${UIDevice.currentDevice.systemVersion}"
</code-block>
</tab>
<tab title="Platform.jvm.kt">
<code-block language="kotlin">
actual fun getPlatform(): String = "JVM Desktop (Java ${System.getProperty("java.version")})"
</code-block>
</tab>
</tabs>

> In IntelliJ IDEA, you can use a quick fix to generate missing `actual` declarations for a given `expect` declaration.
> 
{style="tip"}

### Connect common code to applications

Each application now has to call the common `App()` composable.

#### Android

Set the `App()` composable as the sole content source for the `MainActivity`:

```kotlin
// androidApp/src/MainActivity.kt

package org.hello.platform

import App
import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        enableEdgeToEdge()
        super.onCreate(savedInstanceState)

        setContent {
            // Calls the App() function from common code
            App()
        }
    }
}
```

Write the Android manifest file to declare the main activity for the application:

```xml
<!-- androidApp/src/AndroidManifest.xml -->

<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    <application>
        <activity android:name="org.hello.platform.MainActivity"
                  android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN"/>
                <category android:name="android.intent.category.LAUNCHER"/>
            </intent-filter>
        </activity>
    </application>
</manifest>
```

Now you can run the automatically recognized **Module androidApp** run configuration in IDEA,
or run the following CLI command:

```shell
./kotlin run -m androidApp
```

#### iOS

For the iOS application, wire the common `App()` composable through a Compose `ViewController`,
then consume it in SwiftUI.

On the Kotlin side:

```kotlin
// shared/src@ios/ViewController.kt
import androidx.compose.ui.window.ComposeUIViewController

// Sets the content of the ViewController to App()
fun ViewController() = ComposeUIViewController { App() }
```

On the Swift side:

```swift
// iosApp/src/iosApp.swift

import SwiftUI
// Generated name for the catch-all package with all imported Kotlin
import KotlinModules

struct ComposeView: UIViewControllerRepresentable {
    func makeUIViewController(context: Context) -> some UIViewController {
        // Calls the ViewController imported from Kotlin
        ViewControllerKt.ViewController()
    }
    func updateUIViewController(_ uiViewController: UIViewControllerType, context: Context) {}
}

struct ContentView: View {
    var body: some View {
        ComposeView().ignoresSafeArea()
    }
}

@main
struct iosApp: App {
    var body: some Scene {
        WindowGroup {
            ContentView()
        }
    }
}
```

Now you can run the automatically recognized **Module iosApp** run configuration in IDEA,
or run the following CLI command:

```shell
./kotlin run -m iosApp
```

#### Desktop

For desktop, you only have to create a window and set its content to the `App()` composable:

```kotlin
import androidx.compose.ui.window.Window
import androidx.compose.ui.window.application

fun main() = application {
    Window(onCloseRequest = ::exitApplication) {
        App()
    }
}
```

Now you can run the automatically recognized **Module iosApp** run configuration in IDEA,
or run the following CLI command:

```shell
./kotlin run -m desktopApp
```

## What's next

* If you're not sure what Kotlin Toolchain is and what purpose it serves, check out the [product FAQ](https://kotlin-toolchain.org/dev/faq/).
* To learn more about how Kotlin Toolchain works, see the general [Getting started guide](https://kotlin-toolchain.org/dev/getting-started/)
  and a thorough [User guide](https://kotlin-toolchain.org/dev/user-guide/).