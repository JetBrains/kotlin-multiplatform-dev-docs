[//]: # (title: Use Fleet for Multiplatform development — tutorial)

This article shows the basics of working with Kotlin Multiplatform projects
in [JetBrains Fleet](https://www.jetbrains.com/fleet/).

Fleet is a modern IDE platform designed for collaborative development. It strives to provide a superior experience for
Kotlin Multiplatform. As the first step, Fleet brings Kotlin Multiplatform to macOS.

With Fleet, you can quickly open and run multiplatform projects targeting the Android, iOS, and desktop platforms. The
Fleet's Smart Mode selects the appropriate code-processing engine.

When targeting iOS, navigation, refactoring, and debugging are all available across languages used in the project. That
allows managing a mixed-language codebase with ease. Fleet has full support for Swift, so you can easily extend and
maintain the native parts of your application as well.

## Prepare your development environment

> Currently, Fleet supports Kotlin Multiplatform only on macOS, so you'll need a Mac to complete this tutorial.
>
{type="note"}

1. Install the [JetBrains Toolbox](https://www.jetbrains.com/toolbox-app/) app.

   The Toolbox app provides an easy upgrade path to new versions as they become available. You can also manage multiple
   versions of JetBrains products, working with stable releases while exploring experimental ones.

2. Log into the Toolbox with your [JetBrains account](https://account.jetbrains.com/login).
3. Click **Install** next to the **Fleet** icon.
4. Check your version of the JDK.Currently, Fleet requires the Java Development Kit 17 and later to be installed for
   Kotlin Multiplatform development. To do that, run the following command in your command-line tool:

   ```bash 
   java -version
   ```

5. Install Android Studio and Xcode if you want to target the Android and iOS platforms.
   See [Set up an environment](compose-multiplatform-setup.md) for more details on these prerequisites and the
   information on how to use the KDoctor tool to verify your setup.

    * To ensure the installation is complete, open both Android Studio and Xcode at least once. It may also be necessary
      to reopen Android Studio and Xcode following an update.
    * To run and debug Android applications on the current machine, you'll need to create
      a [virtual device in Android Studio](https://developer.android.com/studio/run/managing-avds#createavd).
    * If you experience problems running an iOS project, on either a simulator or a real device, verify that you can run
      the embedded project in the `iosApp` folder from Xcode on this destination. Then restart Fleet.

## Getting started with Fleet

When using Fleet, you can create a Multiplatform project by:

* Creating a new project from scratch, using the [Kotlin Multiplatform wizard](https://kmp.jetbrains.com/).
* Cloning an [existing Git repository](https://www.jetbrains.com/help/fleet/workspace.html#clone-project),
  using the **Clone from Git** option on the Fleet welcome screen.

### Create a project

In this tutorial, you'll create a new project with the web wizard:

1. Open the [Kotlin Multiplatform wizard](https://kmp.jetbrains.com).
2. On the **New project** tab, change the project name to "SampleProject" and use "com.example.project" as the project's ID.
3. Select the **Android**,**iOS**, and **Desktop** options as your platforms.
4. For iOS, make sure that the **Share UI** option is selected.
5. Click the download button and unpack the resulting archive.

![Kotlin Multiplatform wizard](fleet-web-wizard.png){width=450}

### Enable the Smart Mode

1. Launch Fleet.
2. On the welcome screen, click **Open File or Folder** or select **File | Open** in the editor.
3. Navigate to the unpacked "SampleProject" folder and click **Open**.

   ![Fleet project wizard](fleet-project-wizard-1.png){width=700}

   Fleet detects the project type and opens it for you.

4. When opening a folder with a build file, Fleets gives you an option to enable
   the [Smart Mode](https://www.jetbrains.com/help/fleet/smart-mode.html). Click **Trust and Open**:

   ![Smart mode in Fleet](fleet-project-wizard-2.png){width=700}

    At the top of the window, you'll see messages from Fleet as the project is imported and indexed.

When the Smart Mode is enabled, Fleet operates as an IDE, with all the code-completion, navigation, debugging and
refactoring features available. When it's disabled, Fleet works as a simple file editor. You can quickly open files and
make changes, but most of the productivity features won't be available.

Under the hood, Fleet chooses which backend to use to process code. When the Smart Mode is enabled, the backend for
Kotlin is the same code-processing engine used by IntelliJ IDEA. So the functionality you might be already familiar with
is retained.

To check the Smart Mode status, click the lightning bolt icon at the top:

![Enabling Smart Mode in Fleet](fleet-smart-mode.png){width=300}

### Run your project

When opening a project, Fleet creates [run configurations](https://www.jetbrains.com/help/fleet/run-configs.html) for
the targets found in the build file. To access run configurations, you can either:

* Use the `⌘Cmd + R` shortcut.
* Select **Run** | **Run & Debug** in the main menu.
* Click the run icon at the top of the window.

![Run configurations](fleet-run-configurations.png){width=700}

Choose the `iosApp` configuration from the list and run it. The application will automatically be executed on the iPhone
simulator.

In the build window, you'll see how the application has been compiled and launched:

![Run the iOS app on the iPhone simulator](fleet-run-ios-app.png){width=700}

## Working with multiplatform projects

The sample code generated by the web wizard provides a useful starting point for exploring the code editing
functionality in Fleet. For a detailed explanation of the sample code, see
the [Get started with the Compose Multiplatform](compose-multiplatform-explore-composables.md) tutorial.

Let's examine how Fleet helps you navigate, edit, refactor, and debug code in a multiplatform project.

### Navigating code

Let's see how the cross-language navigation works in Fleet using the example of navigating from Kotlin to Swift:

1. Open the `composeApp/src/commonMain/kotlin/App.kt` file. Here's the `App` composable, the center of the sample code.
2. Select the `App()` function and run the **Usages** action form the context menu or with the `Cmd + U` shortcut. You
   can see it is invoked in three files:

   ![Function show usages](fleet-show-usages.png){width=700}

   This is where the composable is used to launch the Android, iOS, and desktop targets.

3. Select the `MainViewController.kt` file. You'll see the `MainViewController` type that is invoked from Swift code.
4. Instead of manually browsing to find the Swift file that implements this functionality, navigate to it directly, by
   using the **Usages** action again:

   <img src="fleet-navigation.png" alt="Using cross-language navigation in Fleet" width="700" animated="true"/>

   Because there is only one usage, Fleet automatically opens the file for you.

### Editing in multiple languages

Since the `ContentView.swift` file is already open, you can explore Fleet support for editing Swift cod:

1. Add a new function to the `ComposeView` type. Fleet will provide all the expected coding assistance:

   ```swift
   struct ComposeView: UIViewControllerRepresentable {
       // …
       func sumArray(input: [Int]) -> String {
           var total = 0
           for item in input {
               total += item
           }
           return "Total value is \(total)"
       }
   }
   ```

   The IntelliSense completion and code hints work in Fleet as expected. For example, if you forget the `return` keyword,
   or return a value of the wrong type, Fleet will highlight the error.

2. Update  `makeUIViewController()` to invoke the new function:

   ```swift
   func makeUIViewController(context: Context) -> UIViewController {
       MainViewControllerKt.MainViewController(text: sumArray(input: [10,20,30,40]))
   }
   ```

3. Get back to the Kotlin code. Update the `MainViewController.kt` file to accommodate this change:

   ```kotlin
   fun MainViewController(text: String) = ComposeUIViewController { App(text) }
   ```

4. Adjust the `App()` composable as well:

   ```kotlin
   @OptIn(ExperimentalResourceApi::class)
   @Composable
   fun App(text: String? = null) {
       MaterialTheme {
           var greetingText by remember { mutableStateOf(GREETING) }
           var showImage by remember { mutableStateOf(false) }
           Column(
               Modifier.fillMaxWidth(),
               horizontalAlignment = Alignment.CenterHorizontally
           ) {
               Button(onClick = {
                   greetingText = "${getPlatformName()}: $GREETING"
                   showImage = !showImage
               }) {
                   Text(text ?: greetingText)
               }
               AnimatedVisibility(showImage) {
                   Image(
                       painterResource("compose-multiplatform.xml"),
                       null
                   )
               }
           }
       }
   }
   ```

5. Rerun the application in iOS. The simulator will display the message from the new function instead of the original
   greeting:

   ![Editing in multiple languages](fleet-code-editing.png){width=300}

### Refactoring across multiple languages

Navigation is not the only feature in Fleet which is cross-language. The same applies to refactoring.

You have already seen that the `MainViewController()` function is declared in Kotlin, but invoked from Swift.

Select the `MainViewController()` function and use the **Rename** | **Refactoring** action to change its name
to `MainSwiftUIViewController`:

<img src="fleet-refactor-code.png" alt="Refactoring across multiple languages in Fleet" width="700" animated="true"/>

The function name is changed in both Kotlin and Swift files.

### Debugging

One more example of cross language functionality is debugging. The Fleet debugger can automatically step between Swift
and Kotlin code.

1. In the `ContentView.swift` file, set a breakpoint on the return expression of the  `sumArray()` function on line 17.
2. In `App.kt`, set a breakpoint in the `App` composable on line 22.
3. Open the **Run & Debug** menu and click **Debug** next to the `iosApp` configuration.
4. When the breakpoint on line 17 of `ContentView.swift` is reached, use **Step Out** to move to line 7, then **Step
   Into** to move into the `MainSwiftUIViewController()`Kotlin function.
5. Use **Resume** to move on to the breakpoint on line 22 of `App.kt`.

![Refactoring across multiple languages in Fleet](fleet-debug.png){width=700}

## Leave feedback

We encourage you to try out Fleet with your multiplatform projects, share your experience (good or bad), and provide
feedback on the features that matter to you. Since the Kotlin Multiplatform support in Fleet is Experimental, your
feedback and comments will have an impact on how the product grows and matures.

You can leave comments in [YouTrack](https://youtrack.jetbrains.com/issues/FL) or in
the [#fleet channel in Slack](https://app.slack.com/client/T09229ZC6/C063G2ZBF8V).

## What's next?

Learn more about [JetBrains Fleet](https://www.jetbrains.com/help/fleet)
