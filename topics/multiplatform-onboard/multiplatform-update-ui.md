[//]: # (title: Update the user interface)

<secondary-label ref="IntelliJ IDEA"/>
<secondary-label ref="Android Studio"/>

<tldr>
    <p>This tutorial uses IntelliJ IDEA, but you can also follow it in Android Studio – both IDEs share the same core functionality and Kotlin Multiplatform support.</p>
    <br/>
    <p>This is the second part of the <strong>Create a Kotlin Multiplatform app with shared logic and native UI</strong> tutorial. Before proceeding, make sure you've completed previous steps.</p>
    <p><img src="icon-1-done.svg" width="20" alt="First step"/> <a href="multiplatform-create-first-app.md">Create your Kotlin Multiplatform app</a><br/>
       <img src="icon-2.svg" width="20" alt="Second step"/> <strong>Update the user interface</strong><br/>
       <img src="icon-3-todo.svg" width="20" alt="Third step"/> Add dependencies<br/>       
       <img src="icon-4-todo.svg" width="20" alt="Fourth step"/> Share more logic<br/>
       <img src="icon-5-todo.svg" width="20" alt="Fifth step"/> Wrap up your project<br/>
    </p>
</tldr>

To build the user interface, you'll use the [Compose Multiplatform](https://www.jetbrains.com/lp/compose-multiplatform/) toolkit
for the Android part of your project and [SwiftUI](https://developer.apple.com/xcode/swiftui/) for the iOS one.
These are both declarative UI frameworks, and you'll see similarities in the UI implementations. In both cases,
you store the data in the `phrases` variable and later iterate over it to produce a list of `Text` items.

## Update the Android part

The `androidApp` module contains an Android application and defines its main activity.
The Compose Multiplatform UI code is implemented in the `sharedUI` module, which the Android app uses as an Android library.
The UI of the application uses the Compose Multiplatform framework.

Make some changes and see how they are reflected in the UI:

1. Navigate to the `App.kt` file in the `sharedUI/src/commonMain/.../greeting` directory. TODO make sure that directory names are correct
2. Find the `Greeting` class invocation. Select the `greet()` function, right-click it, and select **Go To** | **Declaration or Usages**.
3. In the `Greeting.kt` file, update the `Greeting` class so that the `greet()` function returns a list of strings:

   ```kotlin
   class Greeting {
   
       private val platform: Platform = getPlatform()
   
       fun greet(): List<String> = buildList {
           add(if (Random.nextBoolean()) "Hi!" else "Hello!")
           add("Guess what this is! > ${platform.name.reversed()}!")
       }
   }
   ```
4. Import the `kotlin.random.Random` package as the IDE suggests.
5. Go back to the `App.kt` file and update the `App()` implementation to display the list of strings:

   ```kotlin
   @Composable
   @Preview
   fun App() {
       MaterialTheme {
           val greeting = remember { Greeting().greet() }
   
           Column(
               modifier = Modifier
                   .padding(all = 10.dp)
                   .safeContentPadding()
                   .fillMaxSize(),
               verticalArrangement = Arrangement.spacedBy(8.dp),
           ) {
               greeting.forEach { greeting ->
                   Text(greeting)
                   HorizontalDivider()
               }
           }
       }
   }
   ```

   Here the `Column` composable shows each of the `Text` items, adding padding around them and space between them.

6. Follow the IDE's suggestions to import the missing dependencies.
7. Now you can run the Android app to see how it displays the list of strings:

   ![Updated UI of Android multiplatform app](first-multiplatform-project-on-android-2.png){width=300}

## Work with the iOS module

The `iosApp` directory builds into an iOS application.
It depends on and uses the `sharedLogic` module as an iOS framework.
The UI of the app is written in Swift.

Implement the same changes as in the Android app to account for the update in common code:

1. In IntelliJ IDEA, find the `iosApp/iosApp` folder at the root of your project in the **Project** tool window.
2. Open the `iosApp/ContentView.swift` file, right-click the `Greeting().greet()` call, and select **Go To** | **Declaration or Usages**.
3. To display a list of strings in the same way as in the Android app, replace the code for the `ContentView` structure with the following:

    ```Swift
    struct ContentView: View {
       let phrases = Greeting().greet()
    
       var body: some View {
           List(phrases, id: \.self) {
               Text($0)
           }
       }
    }
    ```

    * The results of the `greet()` call are stored in the `phrases` variable (`let` in Swift is similar to Kotlin's `val`).
    * The `List` function produces a list of `Text` items.

4. Start the iOS run configuration to see the changes:

    ![Updated UI of your iOS multiplatform app](first-multiplatform-project-on-ios-2.png){width=300}

## Possible issues and solutions

### Xcode reports errors in the code calling the shared framework

If you work in Xcode, your Xcode project may be using an old version of the framework.
To resolve this, return to IntelliJ IDEA and rebuild the project or start the iOS run configuration.

### Xcode reports an error when importing the shared framework

If you are using Xcode, you may need to clear cached binaries: try resetting the environment by choosing
**Product | Clean Build Folder** in the main menu.

## Next step

In the next part of the tutorial, you'll learn about dependencies and add a third-party library to expand
the functionality of your project.

**[Proceed to the next part](multiplatform-dependencies.md)**

## Get help

* **Kotlin Slack**. Get an [invite](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up) and join
  the [#multiplatform](https://kotlinlang.slack.com/archives/C3PQML5NU) channel.
* **Kotlin issue tracker**. [Report a new issue](https://youtrack.jetbrains.com/newIssue?project=KT).