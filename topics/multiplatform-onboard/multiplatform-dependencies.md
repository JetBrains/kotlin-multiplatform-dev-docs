[//]: # (title: Add dependencies to your project)

<secondary-label ref="IntelliJ IDEA"/>
<secondary-label ref="Android Studio"/>

<tldr>
    <p>This tutorial uses IntelliJ IDEA, but you can also follow it in Android Studio – both IDEs share the same core functionality and Kotlin Multiplatform support.</p>
    <br/>   
    <p>This is the third part of the <strong>Create a Kotlin Multiplatform app with shared logic and native UI</strong> tutorial. Before proceeding, make sure you've completed previous steps.</p>
    <p><img src="icon-1-done.svg" width="20" alt="First step"/> <a href="multiplatform-create-first-app.md">Create your Kotlin Multiplatform app</a><br/>
        <img src="icon-2-done.svg" width="20" alt="Second step"/> <a href="multiplatform-update-ui.md">Update the user interface</a><br/>
        <img src="icon-3.svg" width="20" alt="Third step"/> <strong>Add dependencies</strong><br/>
        <img src="icon-4-todo.svg" width="20" alt="Fourth step"/> Share more logic<br/>
        <img src="icon-5-todo.svg" width="20" alt="Fifth step"/> Wrap up your project<br/>
    </p>
</tldr>

You've created and tweaked your first Kotlin Multiplatform project!
Now let's learn how to add dependencies to third-party libraries, which is necessary for building successful cross-platform applications.

## Dependency types

There are two types of dependencies that you can use in Kotlin Multiplatform projects:

* _Multiplatform dependencies_. These are multiplatform libraries that support multiple targets and can be used in the
  common source set, `commonMain`.

  Many modern Android libraries already have multiplatform support, like [Koin](https://insert-koin.io/),
  [Coil](https://coil-kt.github.io/coil/), and [SQLDelight](https://sqldelight.github.io/sqldelight/latest/).
  Find more multiplatform libraries on [klibs.io](https://klibs.io/),
  a search service offered by JetBrains for discovering Kotlin Multiplatform libraries.

* _Native dependencies_. These are regular libraries from specific ecosystems.
  In native projects you usually work with them, for example, using Gradle for Android and Swift Package Manager for iOS. 
  
  When you work with a multiplatform project module, typically, you still need native dependencies to use platform APIs
  such as security storage, specific system calls, and so on.
  In the build script, you specify native dependencies in the configuration of native source sets, for example, `androidMain` and `iosMain`.

For both types of dependencies, you can use local and external repositories.

## Add a multiplatform dependency

> If you have experience developing Android apps, adding a multiplatform dependency is similar to adding a
> Gradle dependency in a regular Android project. The only difference is that you need to specify the source set.
>
{style="tip"}

Let's go back to the app and make the greeting a little more festive:
In addition to the OS version, add a function to display the number of days left until New Year's Day.
The `kotlinx-datetime` library, which has full multiplatform support, is the most convenient way to work with dates in your shared code.

1. Open the `build.gradle.kts` file located in the `sharedLogic` directory.
2. Add the following dependency and the Kotlin time opt-in to the `commonMain` source set dependencies:

    ```kotlin
    kotlin {
        //... 
        sourceSets {
            // Opt-in necessary to use experimental kotlin.time APIs
            // from the standard Kotlin library,
            // distinct from the kotlinx-datetime library.
            all { languageSettings.optIn("kotlin.time.ExperimentalTime") }
   
            commonMain.dependencies {
                implementation("org.jetbrains.kotlinx:kotlinx-datetime:%dateTimeVersion%")
            } 
        }
    }
    ```

3. Select the **Build | Sync Project with Gradle Files** menu item
   or click the **Sync Gradle Changes** button in the build script editor to synchronize Gradle files: ![Synchronize Gradle files](gradle-sync.png){width=50}
4. Right-click the `sharedLogic/src/commonMain/.../greeting` directory and select **New | Kotlin Class/File** to create a new file, `NewYear.kt`.
5. In that file, add two functions that calculate
   the number of days from today until the start of next year using the `datetime` date arithmetic and form the phrase to be displayed:
   
   ```kotlin
   @OptIn(ExperimentalTime::class)
   fun daysUntilNewYear(): Int {
       val today = Clock.System.todayIn(TimeZone.currentSystemDefault())
       val closestNewYear = LocalDate(today.year + 1, 1, 1)
       return today.daysUntil(closestNewYear)
   }
   
   fun daysPhrase(): String = "There are only ${daysUntilNewYear()} days left until New Year! 🎆"
   ```
6. Add all necessary imports as suggested by the IDE.
7. In the `Greeting.kt` file, update the `Greeting` class to see the result:
    
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

8. To see the results, re-run your **androidApp** and **iosApp** run configurations from IntelliJ IDEA:

![Updated mobile multiplatform app with external dependencies](first-multiplatform-project-3.png){width=500}

## Next step

In the next part of the tutorial, you'll add more dependencies and more complex logic to your project.

**[Proceed to the next part](multiplatform-upgrade-app.md)**

### See also

* Discover how to work with multiplatform dependencies of all
  kinds: [Kotlin libraries, Kotlin Multiplatform libraries, and other multiplatform projects](multiplatform-add-dependencies.md).
* Learn how to [add Android dependencies](multiplatform-android-dependencies.md)
  and [iOS dependencies with or without CocoaPods](multiplatform-ios-dependencies.md) for use in
  platform-specific source sets.
* Check out the examples of [how to use Android and iOS libraries](multiplatform-samples.md) in sample projects.

## Get help

* **Kotlin Slack**. Get an [invite](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up) and join the [#multiplatform](https://kotlinlang.slack.com/archives/C3PQML5NU) channel.
* **Kotlin issue tracker**. [Report a new issue](https://youtrack.jetbrains.com/newIssue?project=KT).