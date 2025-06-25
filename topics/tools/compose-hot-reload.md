[//]: # (title: Compose Hot Reload)

<primary-label ref="alpha"/>

[Compose Hot Reload](https://github.com/JetBrains/compose-hot-reload) helps you visualize and experiment with UI changes while working on a Compose Multiplatform project.

At the moment, Compose Hot Reload is only available when you include a desktop target in your multiplatform project. 
We're exploring adding support for other targets in the future. In the meantime, using the desktop app as your sandbox
lets you quickly experiment with UI changes in common code without interrupting your flow.

![Compose Hot Reload](compose-hot-reload.gif){width=500}

## Add Compose Hot Reload to your project

Compose Hot Reload can be added in two ways, by:

* [Creating a project from scratch in IntelliJ IDEA or Android Studio](#from-scratch)
* [Adding it as a Gradle plugin  to an existing project](#to-an-existing-project)

### From scratch

This section walks you through the steps to create a multiplatform project with a desktop target in IntelliJ IDEA and
Android Studio. When your project is created, Compose Hot Reload is automatically added.

1. In the [quickstart](quickstart.md), complete the instructions to [set up your environment for Kotlin Multiplatform development](quickstart.md#set-up-the-environment).
2. In IntelliJ IDEA, select **File** | **New** | **Project**.
3. In the panel on the left, select **Kotlin Multiplatform**.
4. Specify the **Name**, **Group**, and **Artifact** fields in the **New Project** window
5. Select the **Desktop** target and click **Create**.
   ![Create multiplatform project with desktop target](create-desktop-project.png){width=700}

### To an existing project

This section walks you through the steps to add Compose Hot Reload to an existing multiplatform project. The steps refer
to the project from the [Create an app with shared logic and UI](compose-multiplatform-create-first-app.md) tutorial as a reference.

> To find the latest version of Compose Hot Reload, see [Releases](https://github.com/JetBrains/compose-hot-reload/releases).
> 
{style="tip"}

1. In your project, update the version catalog. In `gradle/libs.versions.toml`, add the following code:
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
   This prevents the Compose Hot Reload plugin from being loaded multiple times in each of your subprojects.

3. In the `build.gradle.kts` of the subproject containing your multiplatform application (`ComposeDemo/composeApp/build.gradle.kts`), add the following code to your `plugins{}` block:
   ```kotlin
   plugins { 
       alias(libs.plugins.composeHotReload)
   }
   ```

4. To use the full functionality of Compose Hot Reload, your project must run on [JetBrains Runtime](https://github.com/JetBrains/JetBrainsRuntime)
   (JBR), an OpenJDK fork that supports enhanced class redefinition.
   Compose Hot Reload can automatically provision a compatible JBR for your project.
   To allow this, add the following Gradle plugin to your `settings.gradle.kts` file:

   ```kotlin
   plugins {
       id("org.gradle.toolchains.foojay-resolver-convention") version "%foojayResolverConventionVersion%"
   }
   ```

5. Click the **Sync Gradle Changes** button to synchronize Gradle files: ![Synchronize Gradle files](gradle-sync.png){width=50}

## Use Compose Hot Reload

1. In the `desktopMain` directory, open the `main.kt` file and update the `main()` function:
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

2. In the `commonMain` directory, open the `App.kt` file and update the `Button` composable:
   ```kotlin
   Button(onClick = { showContent = !showContent }) {
       Column {
           Text(Greeting().greet())
       }
   }
   ```
   Now, the text for the button is controlled by the `greet()` function.

3. In the `commonMain` directory, open the `Greeting.kt` file and update the `greet()` function:
   ```kotlin
    fun greet(): String {
        return "Hello!"
    }
   ```

4.  In the `desktopMain` directory, open the `main.kt` file and click the **Run** icon in the gutter. 
    Select **Run 'composeApp [desktop]' with Compose Hot Reload (Alpha)**.

    ![Run Compose Hot Reload from gutter](compose-hot-reload-gutter-run.png){width=350}

    ![First Compose Hot Reload on desktop app](compose-hot-reload-hello.png){width=500}

5. Update the string returned from the `greet()` function and see the desktop app update automatically.

   ![Compose Hot Reload](compose-hot-reload.gif){width=500}

Congratulations! You've seen Compose Hot Reload in action. Now you can experiment with changing text, images, formatting, 
UI structure, and more, without having to restart the desktop run configuration after every change.

## Get help

If you encounter any problems using Compose Hot Reload, let us know by [creating a GitHub issue](https://github.com/JetBrains/compose-hot-reload/issues).