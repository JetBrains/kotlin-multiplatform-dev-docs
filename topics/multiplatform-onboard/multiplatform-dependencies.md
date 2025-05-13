[//]: # (title: Add dependencies to your project)

<tldr>
    <p>This is the third part of the <strong>Create a Kotlin Multiplatform app with shared logic and native UI</strong> tutorial. Before proceeding, make sure you've completed previous steps.</p>
    <p><img src="icon-1-done.svg" width="20" alt="First step"/> <a href="multiplatform-create-first-app.md">Create your Kotlin Multiplatform app</a><br/>
        <img src="icon-2-done.svg" width="20" alt="Second step"/> <a href="multiplatform-update-ui.md">Update the user interface</a><br/>
        <img src="icon-3.svg" width="20" alt="Third step"/> <strong>Add dependencies</strong><br/>
        <img src="icon-4-todo.svg" width="20" alt="Fourth step"/> Share more logic<br/>
        <img src="icon-5-todo.svg" width="20" alt="Fifth step"/> Wrap up your project<br/>
    </p>
</tldr>

You've already created your first cross-platform Kotlin Multiplatform project! Now let's learn how to add dependencies
to third-party libraries, which is necessary for building successful cross-platform applications.

## Dependency types

There are two types of dependencies that you can use in Kotlin Multiplatform projects:

* _Multiplatform dependencies_. These are multiplatform libraries that support multiple targets and can be used in the
  common source set, `commonMain`.

  Many modern Android libraries already have multiplatform support, like [Koin](https://insert-koin.io/),
  [Apollo](https://www.apollographql.com/), and [Okio](https://square.github.io/okio/). Find more multiplatform libraries on [klibs.io](https://klibs.io/),
  an experimental search service from JetBrains for discovering Kotlin Multiplatform libraries.

* _Native dependencies_. These are regular libraries from relevant ecosystems. In native projects you usually work with them
  using Gradle for Android and using CocoaPods or another dependency manager for iOS. 
  
  When you work with a shared module, typically, you still need native dependencies when you want to use platform APIs
  such as security storage. You can add native dependencies to the native source sets, `androidMain` and `iosMain`.

For both types of dependencies, you can use local and external repositories.

## Add a multiplatform dependency

> If you have experience developing Android apps, adding a multiplatform dependency is similar to adding a
> Gradle dependency in a regular Android project. The only difference is that you need to specify the source set.
>
{style="tip"}

Let's go back to the app and make the greeting a little more festive. In addition to the device information, add a
function to display the number of days left until New Year's Day. The `kotlinx-datetime` library, which has full
multiplatform support, is the most convenient way to work with dates in your shared code.

1. Open the `build.gradle.kts` file located in the `shared` directory.
2. Add the following dependency to the `commonMain` source set dependencies:

   ```kotlin
   kotlin {
       sourceSets {
           commonMain.dependencies {
               implementation("org.jetbrains.kotlinx:kotlinx-datetime:%dateTimeVersion%")
           } 
       }
   }
   ```

3. Click the **Sync Gradle Changes** button to synchronize Gradle files: ![Synchronize Gradle files](gradle-sync.png){width=50}
4. In `shared/src/commonMain/kotlin`, create a new file `NewYear.kt` in the project directory where your `Greeting.kt` file is located.
5. Update the file with a short function that calculates
   the number of days from today until the New Year using the `date-time` date arithmetic:
   
   ```kotlin
   import kotlinx.datetime.*
   
   fun daysUntilNewYear(): Int {
       val today = Clock.System.todayIn(TimeZone.currentSystemDefault())
       val closestNewYear = LocalDate(today.year + 1, 1, 1)
       return today.daysUntil(closestNewYear)
   }
   
   fun daysPhrase(): String = "There are only ${daysUntilNewYear()} days left until New Year! 🎆"
   ```

6. In `Greeting.kt`, update the `Greeting` class to see the result:
    
    ```kotlin
    class Greeting {
        private val platform: Platform = getPlatform()
   
        fun greet(): List<String> = buildList {
            add(if (Random.nextBoolean()) "Hi!" else "Hello!")
            add("Guess what this is! > ${platform.name.reversed()}!")
            add(daysPhrase())
        }
    }
    ```

7. To see the results, re-run your **composeApp** and **iosApp** configurations from IntelliJ IDEA:

![Updated mobile multiplatform app with external dependencies](first-multiplatform-project-3.png){width=500}

> You can find this state of the project in our [GitHub repository](https://github.com/kotlin-hands-on/get-started-with-kmp/tree/main/step4).
>
{style="tip"}

## Next step

In the next part of the tutorial, you'll add more dependencies and more complex logic to your project.

**[Proceed to the next part](multiplatform-upgrade-app.md)**

### See also

* Discover how to work with multiplatform dependencies of all
  kinds: [Kotlin libraries, Kotlin Multiplatform libraries, and other multiplatform projects](https://kotlinlang.org/docs/multiplatform-add-dependencies.html).
* Learn how to [add Android dependencies](https://kotlinlang.org/docs/multiplatform-android-dependencies.html)
  and [iOS dependencies with or without CocoaPods](https://kotlinlang.org/docs/multiplatform-ios-dependencies.html) for use in
  platform-specific source sets.
* Check out the examples of [how to use Android and iOS libraries](multiplatform-samples.md) in sample projects.

## Get help

* **Kotlin Slack**. Get an [invite](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up) and join the [#multiplatform](https://kotlinlang.slack.com/archives/C3PQML5NU) channel.
* **Kotlin issue tracker**. [Report a new issue](https://youtrack.jetbrains.com/newIssue?project=KT).