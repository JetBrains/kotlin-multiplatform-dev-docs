[//]: # (title: Add dependencies to your project)

<secondary-label ref="IntelliJ IDEA"/>
<secondary-label ref="Android Studio"/>


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
> Gradle dependency in a regular Android project. The only difference is that you need to add it to a specific source set.
>
{style="tip"}

Let's make the greeting a little more festive:
In addition to the OS version, add a function to display the number of days left until New Year's Day.
The `kotlinx-datetime` library, which has full multiplatform support, is the most convenient way to work with dates in your shared code.

1. Open the `gradle/libs.versions.toml` file and add the `kotlinx-datetime` dependency to the version catalog:
    ```text
    [versions]
    kotlinx-datetime = "0.8.0"
    
    [libraries]
    kotlinx-datetime = { module = "org.jetbrains.kotlinx:kotlinx-datetime", version.ref = "kotlinx-datetime" }
    ```
2. Open the `sharedLogic/build.gradle.kts` file and add a reference to that library entry
    to the section that configures the common code source set:

    ```kotlin
    kotlin {
        //... 
        sourceSets {
            commonMain.dependencies {
                implementation(libs.kotlinx.datetime)
            } 
        }
    }
    ```

3. Select the **Build | Sync Project with Gradle Files** menu item
   or click the **Sync Gradle Changes** button in the build script editor to synchronize Gradle files: ![Synchronize Gradle files](gradle-sync.png){width=50}

## Call a kotlinx-datetime API

With the dependency added, you can add date and time calculations to your common code:

1. Right-click the `sharedLogic/src/commonMain/.../greetingkmp` directory and select **New | Kotlin Class/File** to create a new file, `NewYear.kt`.
2. In `NewYear.kt`, add two functions that calculate
   the number of days from today until the start of next year using the `datetime` date arithmetic and form the phrase to be displayed:
   
   ```kotlin
   fun daysUntilNewYear(): Int {
       val today = Clock.System.todayIn(TimeZone.currentSystemDefault())
       val closestNewYear = LocalDate(today.year + 1, 1, 1)
       return today.daysUntil(closestNewYear)
   }
   
   fun daysPhrase(): String = "There are only ${daysUntilNewYear()} days left until New Year! 🎆"
   ```
3. Add all necessary imports as suggested by the IDE.
   Make sure to import `kotlin.time.Clock`, not `kotlinx.datetime.Clock`. 
4. In the `Greeting.kt` file, update the `Greeting` class to see the result:
    
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

5. To see the results, re-run your **androidApp** and **iosApp** run configurations from IntelliJ IDEA:

![Updated mobile multiplatform app with external dependencies](first-multiplatform-project-3.png){width=600}

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