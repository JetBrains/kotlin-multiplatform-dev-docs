[//]: # (title: Compose hot reload)

<primary-label ref="beta"/>

Compose hot reload is a feature for your IDE that helps you visualize and experiment with UI changes while
working on a Compose Multiplatform project.

At the moment, Compose hot reload is only available when you include a desktop target in your multiplatform project. 
We're exploring adding support for other targets in the future. In the meantime, using the desktop app as your sandbox
lets you quickly experiment with UI changes without interrupting your flow.

![Compose hot reload](compose-hot-reload.gif){width=500}

## Add Compose hot reload to your project

Compose hot reload is available as a Gradle plugin. This tutorial guides you through the steps to add Compose hot
reload to an existing multiplatform project. We'll use the same project that we use for the [Create an app with shared logic and UI](compose-multiplatform-create-first-app.md)
tutorial.

### Create a project

1. In the [quickstart](quickstart.md), complete the instructions to [set up your environment for Kotlin Multiplatform development](quickstart.md#set-up-the-environment).
2. In IntelliJ IDEA, select **File** | **New** | **Project**.
3. In the panel on the left, select **Kotlin Multiplatform**.
4. Specify the following fields in the **New Project** window:

   * **Name**: ComposeDemo
   * **Group**: compose.project.demo
   * **Artifact**: composedemo

   ![Create Compose Multiplatform project](create-compose-multiplatform-project.png){width=800}

5. Select **Android**, **Desktop**, and **Web** targets.
   * If you're using a Mac, select **iOS** as well. Make sure that the **Share UI** option is selected.
6. Once you've specified all the fields and targets, click **Create**.

### Update your Gradle files

1. Update the version catalog. In `gradle/libs.versions.toml`, add the following code:
   ```kotlin
   composeHotReload = { id = "org.jetbrains.compose.hot-reload", version.ref = "composeHotReload"}
   ```

   > To learn more about how to use a version catalog to centrally manage dependencies across your project, see our [Gradle best practices](https://kotlinlang.org/gradle-best-practices.html).

2. In the `build.gradle.kts` of your parent project (`ComposeDemo/build.gradle.kts`), add the following code to your `plugins{}` block:
   ```kotlin
   plugins {
       alias(libs.plugins.composeHotReload) apply false
   }
   ```
   This prevents the Compose hot reload plugin from being loaded multiple times in each of your subprojects.

3. In the `build.gradle.kts` of the subproject containing your multiplatform application (`ComposeDemo/composeApp/build.gradle.kts`), add the following code to your `plugins{}` block:
   ```kotlin
   plugins { 
       alias(libs.plugins.composeHotReload)
   }
   ```

4. In your `settings.gradle.kts` file, add the following code:
   ```kotlin
   plugins {
       id("org.gradle.toolchains.foojay-resolver-convention") version "0.10.0"
   }
   ```

5. Click the **Sync Gradle Changes** button to synchronize Gradle files: ![Synchronize Gradle files](gradle-sync.png){width=50}

## Use Compose hot reload

1. In the `desktopMain` source set, open the `main.kt` file and update the `main()` function:
   ```kotlin
   fun main() = application {
       Window(
           onCloseRequest = ::exitApplication,
           alwaysOnTop = true,
           title = "composedemo",
       ) {
           App()
       }
   }
   ```
   By setting the `alwaysOnTop` variable to `true`, the generated desktop app stays on top of all your windows, making it easier
   to edit your code and see changes live.

2. In the `commonMain` source set, open the `App.kt` file and update the `Button` composable:
   ```kotlin
   Button(onClick = { showContent = !showContent }) {
       Column {
           Text(Greeting().greet())
       }
   }
   ```
   Now, the text for the button is controlled by the `greet()` function.

3. In the `commonMain` source set, open the `Greeting.kt` file and update the `greet()` function:
   ```kotlin
    fun greet(): String {
        return "Hello!"
    }
   ```

4. Select **composeApp [desktop]** in the list of run configurations and click **Run**. The desktop app is automatically generated.

   ![First Compose hot reload on desktop app](compose-hot-reload-hello.png){width=500}

5. Update the string returned from the `greet()` function and see the desktop app update automatically.

![Compose hot reload](compose-hot-reload.gif){width=500}

Congratulations! You've run your first Compose hot reload. Now you can experiment with changing text, images, formatting, 
UI structure, and more, without having to restart the desktop run configuration every time.

## Get help

If you encounter any problems using Compose hot reload, let us know by [creating a GitHub issue](https://github.com/JetBrains/compose-hot-reload/issues).
   


