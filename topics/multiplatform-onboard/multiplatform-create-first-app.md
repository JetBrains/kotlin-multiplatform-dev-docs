[//]: # (title: Create your Kotlin Multiplatform app)

<secondary-label ref="IntelliJ IDEA"/>
<secondary-label ref="Android Studio"/>

<tldr>
    <p>This tutorial uses IntelliJ IDEA, but you can also follow it in Android Studio – both IDEs share the same core functionality and Kotlin Multiplatform support.</p>
    <br/>
    <p>This is the first part of the <strong>Create a Kotlin Multiplatform app with shared logic and native UI</strong> tutorial.</p>
    <p><img src="icon-1.svg" width="20" alt="First step"/> <strong>Create your Kotlin Multiplatform app</strong><br/>
       <img src="icon-2-todo.svg" width="20" alt="Second step"/> Update the user interface<br/>
       <img src="icon-3-todo.svg" width="20" alt="Third step"/> Add dependencies<br/>       
       <img src="icon-4-todo.svg" width="20" alt="Fourth step"/> Share more logic<br/>
       <img src="icon-5-todo.svg" width="20" alt="Fifth step"/> Wrap up your project<br/>
    </p>
</tldr>

Here you will learn how to create and run your first Kotlin Multiplatform application using IntelliJ IDEA.

Kotlin Multiplatform technology simplifies the development of cross-platform projects.
Kotlin Multiplatform applications can work on a variety of platforms like iOS, Android, macOS, Windows, Linux, web, and others.

One of the major Kotlin Multiplatform use cases is sharing code between mobile platforms.
You can share application logic between iOS and Android apps and write platform-specific code only when you need to implement a native UI or work with platform APIs.

## Create a project

1. In the [quickstart](quickstart.md), complete the instructions to [set up your environment for Kotlin Multiplatform development](quickstart.md#set-up-the-environment).
2. In IntelliJ IDEA, select **File** | **New** | **Project**.
3. In the panel on the left, select **Kotlin Multiplatform**.
4. Specify the following fields in the **New Project** window:
   * **Name**: GreetingKMP
   * **Group**: com.jetbrains.greeting
   * **Artifact**: greetingkmp
   
   ![Create Compose Multiplatform project](create-first-multiplatform-app.png){width=800}
    TODO update the screenshot

5. Select **Android** and **iOS** targets.
6. For iOS, select the **Do not share UI** option to keep the UI native.
7. Once you've specified all the fields and targets, click **Create**.

> The IDE may automatically suggest upgrading the Android Gradle plugin in the project to the latest version.
> We don't recommend upgrading as Kotlin Multiplatform is not compatible with the latest AGP version
> (see the [compatibility table](https://kotlinlang.org/docs/multiplatform-compatibility-guide.html#version-compatibility)).
>
{style="note"}

## Examine the project structure

In IntelliJ IDEA, expand the `GreetingKMP` folder.

This Kotlin Multiplatform project includes the following modules:

* _sharedLogic_ is the multiplatform module that contains the logic common for both Android and iOS applications.
* _sharedUI_ is the module with Compose Multiplatform UI code: in this project, it is used only by the Android app,
  but it is a multiplatform module that can be used by other targets whenever you are ready for that.
* _androidApp_ is a Kotlin module that builds into an Android application. It uses Gradle as the build system.
  The composeApp module depends on and uses the shared module as a regular Android library.
* _iosApp_ is an Xcode project that builds into an iOS application.
  It depends on and uses the `sharedLogic` module as an iOS framework.
  By default, Kotlin Multiplatform projects created using the IDE wizard use the regular framework dependency through
  [direct integration](multiplatform-direct-integration.md).

Every module except for `iosApp` uses Gradle as the build system.

![Basic Multiplatform project structure](basic-project-structure.svg){width=700}

The shared module consists of three source sets:
`androidMain`, `commonMain`, and `iosMain`.
_Source set_ is a Gradle concept for a set of files logically grouped together where each set has its own dependencies.
In Kotlin Multiplatform, different source sets in a shared module can target different platforms.

The `commonMain` source set contains shared Kotlin code, and platform source sets use Kotlin code specific to each target.
Kotlin/JVM is used for `androidMain` and Kotlin/Native for `iosMain`:

![Source sets and modules structure](basic-project-structure-2.png){width=350}
TODO screenshot is outdated

When the shared module is built into an Android library, common Kotlin code is treated as Kotlin/JVM.
When it is built into an iOS framework, common Kotlin is treated as Kotlin/Native:

![Common Kotlin, Kotlin/JVM, and Kotlin/Native](modules-structure.png)

### Write common declarations

The common source set contains shared code that can be used across multiple target platforms.
It's designed to contain code that is platform-independent. If you try to use platform-specific APIs in the common source set,
the IDE will show a warning:

1. Open the `sharedLogic/src/commonMain/.../Greeting.kt` file
    where you can find the automatically generated `Greeting` class with a `greet()` function.
2. Let's add a bit of variety to the greeting. Navigate to the definition of the `sayHello()` function in the `GreetingUtil.kt` file.

3. Update the shared code with randomization and the `reversed()` call from the Kotlin standard library to reverse
   the received string:

    ```kotlin
    fun sayHello(to: String): String {
        val firstWord = if (Random.nextBoolean()) "Hi!" else "Hello!"

        return "$firstWord Guess what this is! > ${to.reversed()}!"
    }
    ```
4. Import the `kotlin.random.Random` class following the IDE's suggestion.

Writing the code only in common Kotlin has obvious limitations because it can't use any platform-specific functionality.
Using common interfaces with platform-specific implementations using the [expect/actual](multiplatform-connect-to-apis.md) mechanism solves this.

### Check out platform-specific implementations

The common source set can define expected declarations — interfaces, classes, and so on.
In each platform source set, in this case `androidMain` and `iosMain`,
you have to provide actual platform-specific implementations for the expected declarations.

While generating the code for a specific platform, the Kotlin compiler merges expected and actual declarations
and generates a single declaration with actual implementations.

1. When creating a Kotlin Multiplatform project with IntelliJ IDEA,
   you get a `Platform.kt` file in the `commonMain` module:

    ```kotlin
    interface Platform {
        val name: String
    }
    ```

   It's a common `Platform` interface supposed to include information about a platform.

2. You can find the platform-specific classes that implement the interface in the `androidMain` and the `iosMain` source sets:
    
    ```kotlin
    // Platform.android.kt in the androidMain source set
    import android.os.Build

    class AndroidPlatform : Platform {
        override val name: String = "Android ${Build.VERSION.SDK_INT}"
    }
    ```
   
    ```kotlin
    // Platform.ios.kt in the iosMain source set
    import platform.UIKit.UIDevice
    
    class IOSPlatform: Platform {
        override val name: String =
            UIDevice.currentDevice.systemName() + " " + UIDevice.currentDevice.systemVersion
    }
    ```

    * The `name` property of the `AndroidPlatform` class uses the Android-specific code, namely the `android.os.Build`
      dependency. This code is interpreted as Kotlin/JVM.
      If you try to access a JVM-specific class, such as `java.util.Random` here, this code will compile.
    * The `name` property of the `IOSPlatform` class uses iOS-specific code, namely the `platform.UIKit.UIDevice`
      dependency. This code is interpreted as Kotlin/Native, meaning that you can refer to iOS declarations in Kotlin.
      The code becomes a part of the iOS framework, which is imported in the Swift code of the `iosApp` module.

3. Each source set includes a `getPlatform()` function.
   Its expected declaration doesn't have a body, and actual implementations are provided in the platform code:

    ```kotlin
    // Platform.kt in the commonMain source set
    expect fun getPlatform(): Platform
    ```
   
    ```kotlin
    // Platform.android.kt in the androidMain source set
    actual fun getPlatform(): Platform = AndroidPlatform()
    ```
   
    ```kotlin
    // Platform.ios.kt in the iosMain source set
    actual fun getPlatform(): Platform = IOSPlatform()
    ```

Here, the common source set defines an expected `getPlatform()` function and has actual implementations,
`AndroidPlatform()` for the Android app and `IOSPlatform()` for the iOS app, in the platform source sets.

While generating the code for a specific platform, the Kotlin compiler merges expected and actual declarations
into a single `getPlatform()` function with the correct implementation.

That's why expected and actual declarations should be defined in the same package – they are merged into one declaration
in the resulting platform code. Any invocation of the expected `getPlatform()` function in the generated platform code
should refer to the correct actual implementation.

Now you can run the apps and see all of this in action.

#### Create an expect/actual variable (optional) {initial-collapse-state="collapsed" collapsible="true"}

The template project uses the expect/actual mechanism for functions, but it also works for most Kotlin declarations,
such as properties and classes. Let's implement an expected property:

1. Open `Platform.kt` in the `commonMain` module and add the following at the end of the file:

   ```kotlin
   expect val num: Int
   ```

   The Kotlin compiler complains that this property has no corresponding actual declarations in the platform modules.

2. Try to provide the implementation right away with:

   ```kotlin
   expect val num: Int = 42
   ```

   You'll get an error saying that expected declarations must not have a body, in this case an initializer.
   The implementations must be provided in actual platform modules. Remove the initializer.
3. Hover over the `expect` keyword for the `num` property and click **Create missed actuals...** in the IDE quick fix suggestion.
4. Choose the `androidMain` source set. Complete the implementation in `androidMain/Platform.android.kt` to be as follows:

   ```kotlin
   actual val num: Int = 1
    ```

5. Now provide the actual implementation for `num` in the `iosMain` module. Add the following to the `iosMain/Platform.ios.kt` file:

   ```kotlin
   actual val num: Int = 2
   ```

6. In the `commonMain/GreetingUtil.kt` file, add the `num` property to the string formed by the `sayHello()` function:

    ```kotlin
    fun sayHello(to: String): String {
        val firstWord = if (Random.nextBoolean()) "Hi!" else "Hello!"

        return "$firstWord [$num] Guess what this is! > ${to.reversed()}!"
    }
    ```

## Run your application

You can run your multiplatform application for both Android and iOS from IntelliJ IDEA.

If you have created the optional expect/actual variable earlier, you should see "[1]" added to the greeting for Android
and "[2]" added for iOS.

### Run your application on Android

1. In the list of run configurations, select **androidApp**.
2. Choose an Android virtual device next to the list of configurations and click **Run**.

   If you don't have a device in the list, create a [new Android virtual device](https://developer.android.com/studio/run/managing-avds#createavd).

   ![Run multiplatform app on Android](compose-run-android.png){width=350}

   ![First mobile multiplatform app on Android](first-multiplatform-project-on-android-1.png){width=300}

<include from="compose-multiplatform-create-first-app.md" element-id="run_android_other_devices"/>

### Run your application on iOS

If you haven't launched Xcode as part of the initial setup, do that before running the iOS app.

In IntelliJ IDEA, select **iosApp** in the list of run configurations, select a simulated device next to the run configuration,
and click **Run**.

If you don't have an available iOS configuration in the list, add a [new run configuration](#run-on-a-new-ios-simulated-device).

![Run multiplatform app on iOS](compose-run-ios.png){width=350}

![First mobile multiplatform app on iOS](first-multiplatform-project-on-ios-1.png){width=300}

<include from="compose-multiplatform-create-first-app.md" element-id="run_ios_other_devices"/>

## [Next step](multiplatform-update-ui.md)

In the next part of the tutorial, you'll learn how to update the UI elements using platform-specific libraries.

### See also

* See how to [create and run multiplatform tests](multiplatform-run-tests.md) to check that the code works correctly.
* Learn more about the [project structure](multiplatform-discover-project.md).
* If you want to convert your existing Android project into a cross-platform app, [complete this tutorial to make your Android app cross-platform](multiplatform-integrate-in-existing-app.md).

## Get help

* **Kotlin Slack**. Get an [invite](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up) and join
  the [#multiplatform](https://kotlinlang.slack.com/archives/C3PQML5NU) channel.
* **Kotlin issue tracker**. [Report a new issue](https://youtrack.jetbrains.com/newIssue?project=KT).
