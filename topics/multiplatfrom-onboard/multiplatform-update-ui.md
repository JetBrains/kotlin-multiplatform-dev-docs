[//]: # (title: Update the user interface)

<microformat>
    <p>This is the third part of the <strong>Getting started with Kotlin Multiplatform</strong> tutorial. Before proceeding, make sure you've completed previous steps.</p>
    <p><img src="icon-1-done.svg" width="20" alt="First step"/> <a href="multiplatform-setup.md">Set up an environment</a><br/>
       <img src="icon-2-done.svg" width="20" alt="Second step"/> <a href="multiplatform-create-first-app.md">Create your first cross-platform app</a><br/>
       <img src="icon-3.svg" width="20" alt="Third step"/> <strong>Update the user interface</strong><br/>       
       <img src="icon-4-todo.svg" width="20" alt="Fourth step"/> Add dependencies<br/>
       <img src="icon-5-todo.svg" width="20" alt="Fifth step"/> Share more logic<br/>
       <img src="icon-6-todo.svg" width="20" alt="Sixth step"/> Wrap up your project</p>
</microformat>

To build the user interface, you'll use the [Compose Multiplatform](https://www.jetbrains.com/lp/compose-multiplatform/) toolkit
for the Android part of your project and [SwiftUI](https://developer.apple.com/xcode/swiftui/) for the iOS one.
These are both declarative UI frameworks, and you'll see similarities in the UI implementations. In both cases,
you store the data in the `phrases` variable and later iterate over it to produce a list of `Text` items.

## Update the Android part

The `composeApp` module contains an Android application, defines its main activity and the UI views, and uses the
`shared` module as a regular Android library. The UI of the application uses the Compose Multiplatform framework.

Make some changes and see how they are reflected in the UI:

1. Navigate to the `App.kt` file in `composeApp/src/commonMain/kotlin`.
2. Find the `Greeting` class invocation. Select the `greet()` function, right-click it and select the **Go To** | **Declaration or Usages** menu item.
   You'll see that it's the same class from the `shared` module you edited in the previous step.
3. In `Greeting.kt`, update the `greet()` function:

   ```kotlin
   fun greet(): List<String> = buildList {
       add(if (Random.nextBoolean()) "Hi!" else "Hello!")
       add("Guess what this is! > ${platform.name.reversed()}!")
   }
   ```

   Now it returns a list of strings.

4. Go back to `App.kt` and update its definition:

   ```kotlin
   @Composable
   fun App() {
       MaterialTheme {
           val greeting = remember { Greeting().greet() }
   
           Column(
               modifier = Modifier.padding(all = 20.dp),
               verticalArrangement = Arrangement.spacedBy(8.dp),
           ) {
               greeting.forEach { greeting ->
                   Text(greeting)
                   Divider()
               }
           }
       }
   }
   ```

   Here the `Column` composable shows each of the `Text` items, adds padding around the content, and adds a space between the list items.

5. Follow Android Studio's suggestions to import the missing dependencies.
6. Now you can run the Android app to ensure it displays the list:

   ![Updated UI of Android multiplatform app](first-multiplatform-project-on-android-2.png){width=300}

## Work with the iOS module in Xcode

`iosApp` is an Xcode project that builds into an iOS application. It depends on and uses the `shared` module as an iOS
framework. The UI of the app is written in Swift.

Implement the same changes as in the Android app:

1. Navigate to the `iosApp` folder in the Project window.
2. Right-click the `iosApp.xcodeproj` folder and select **Open In** | **Xcode**.
3. In the `ContentView.swift` file, select the `greet()` function and use the <shortcut>⌃ ⌘ J</shortcut> shortcut to jump to the function's definition.

   You'll see the Objective-C declarations for the Kotlin functions defined in the `shared` module. Kotlin types are
   represented as Objective-C types when used from Objective-C/Swift. Here the `greet()` function
   returns `List<String>` in Kotlin and is seen from Swift as returning `NSArray<NSString>`. For more on type mappings,
   see [Interoperability with Swift/Objective-C](https://kotlinlang.org/docs/native-objc-interop.html).

4. If you try to run the project, the build will fail. The Swift code that uses the `greet()` function doesn't compile
   because its declaration is now different. Change the SwiftUI code to display a list of items:

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

5. Run the app to see the changes:

   ![Updated UI of your iOS multiplatform app](first-multiplatform-project-on-ios-2.png){width=300}

> You can find this state of the project in our [GitHub repository](https://github.com/kotlin-hands-on/get-started-with-kmp/tree/main/step3).
>
{type="tip"}

## Next step

In the next part of the tutorial, you'll learn about dependencies and add a third-party library to expand
the functionality of your project.

**[Proceed to the next part](multiplatform-dependencies.md)**

## Get help

* **Kotlin Slack**. Get an [invite](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up) and join
  the [#multiplatform](https://kotlinlang.slack.com/archives/C3PQML5NU) channel.
* **Kotlin issue tracker**. [Report a new issue](https://youtrack.jetbrains.com/newIssue?project=KT).