[//]: # (title: Create your Kotlin Multiplatform library – tutorial)

<secondary-label ref="IntelliJ IDEA"/>
<secondary-label ref="Android Studio"/>

<tldr>
<p>This tutorial uses IntelliJ IDEA, but you can also follow it in Android Studio – both IDEs share the same core functionality and Kotlin Multiplatform support.</p>
</tldr>

In this tutorial, you'll learn how to create a multiplatform library in IntelliJ IDEA, 
publish the library to a local Maven repository, and add it as a dependency in another project.

The tutorial is based on our [multiplatform library template](https://github.com/Kotlin/multiplatform-library-template),
which is a simple library containing a function to generate the Fibonacci 
sequence.

## Set up the environment 

[Install all the necessary tools and update them to the latest versions](quickstart.md).

## Create a project

1. In IntelliJ IDEA, select **File** | **New** | **Project from Version Control**.
2. Enter the URL of the [multiplatform library template project](https://github.com/Kotlin/multiplatform-library-template):

    ```text
    https://github.com/Kotlin/multiplatform-library-template
    ```
   
3. Click **Clone**.

## Examine the project structure

The Kotlin Multiplatform library template project provides a foundational structure 
for developing Kotlin Multiplatform libraries. This template helps create libraries that can operate across 
various platforms.

In the template project, `library` serves as the core module and contains the main source code and build resources for 
the Multiplatform library.

![Multiplatform library project structure](multiplatform-library-template-project.png){width=350}

The `library` module is structured to accommodate shared code as well as platform-specific implementations. 
Here's a breakdown of the contents in its main source code (`src`):

* **`commonMain`:** contains Kotlin code shared across all target platforms. This is where you place code that doesn't 
     rely on any platform-specific APIs.
* **`androidMain`, `iosMain`, `jvmMain`, and `linuxX64Main`:** contain code specific to the Android, iOS, JVM, and Linux platforms. 
     This is where you implement functionalities that are unique to these platforms.
* **`commonTest`, `androidUnitTest`, `iosTest`, `jvmTest`, and `linuxX64Test`:** contain tests for the shared `commonMain` code and
     tests specific to the Android, iOS, JVM, and Linux platforms, respectively.

Let's focus on the `library` code that is shared across all platforms. Inside the `src/commonMain/kotlin` directory, 
you can find the `CustomFibi.kt` file with Kotlin Multiplatform code that defines a Fibonacci sequence generator:

```kotlin
package io.github.kotlin.fibonacci

// Defines the function to generate the Fibonacci sequence
fun generateFibi() = sequence {
    var a = firstElement
    yield(a)
    
    var b = secondElement
    yield(b)
    
    while (true) {
        val c = a + b
        yield(c)
        a = b
        b = c
    }
}

// Declares the expected values for `firstElement` and `secondElement`
expect val firstElement: Int
expect val secondElement: Int
```

The `firstElement` and `secondElement` properties are placeholders that the platform-specific code can implement.
Each target should provide actual values by using the `actual` keyword in their respective source sets.

The `expect` declarations are [matched](multiplatform-connect-to-apis.md#expected-and-actual-functions-and-properties)
with `actual` implementations.
This mechanism is useful when writing cross-platform code that requires platform-specific behavior. 

In this case, the multiplatform library template includes platform-specific implementations of the `firstElement` and 
`secondElement` properties. The `androidMain`, `iosMain`, `jvmMain`, and `linuxX64Main` directories contain `actual` 
declarations that provide values for these properties.

For example, here's the Android implementation included in `androidMain/kotlin/fibiprops.android.kt`:

```kotlin
package io.github.kotlin.fibonacci

actual val firstElement: Int = 1
actual val secondElement: Int = 2
```

The other platforms follow the same pattern, with variations in the values of `firstElement` and `secondElement` properties.

## Add a new platform

Now that you're familiar with how shared and platform-specific code work in the template, let's extend the project by adding support for an additional platform.

Configure support for the [Kotlin/Wasm](https://kotlinlang.org/docs/wasm-overview.html) platform by using 
the [`expect`/`actual` mechanism](multiplatform-connect-to-apis.md#expected-and-actual-functions-and-properties). You can implement platform-specific functionality for the `firstElement` and `secondElement` properties.

### Add the Kotlin/Wasm target to your project

1. In the `library/build.gradle.kts` file, add the Kotlin/Wasm target (`wasmJs`) and source sets:

    ```kotlin
    kotlin {
        // ...
        @OptIn(org.jetbrains.kotlin.gradle.ExperimentalWasmDsl::class)
        wasmJs {
            browser()
            binaries.executable()
        }
        // ...
        sourceSets {
            //...
            val wasmJsMain by getting {
                dependencies {
                    // Wasm-specific dependencies
                }
            }
        }
    }
    ```

2. Synchronize the Gradle files by clicking the **Sync Gradle Changes** icon (![Gradle sync icon](gradle-sync-icon.png){width=30}{type="joined"}) that appears in your build file. Alternatively,
   click the refresh button in the Gradle tool window.

### Create platform-specific code for Wasm

After adding the Wasm target, you need a Wasm directory to hold the platform-specific implementation of
`firstElement` and `secondElement`:

1. Right-click the `library/src` directory and select **New | Directory**. 
2. Select **wasmJsMain/kotlin** from the **Gradle Source Sets** lists.

   ![Gradle source sets list](gradle-source-sets-list.png){width=450}

3. Right-click the newly created `wasmJsMain/kotlin` directory and select **New | Kotlin Class/File**. 
4. Enter **fibiprops.wasm** as the name of the file and select **File**.
5. Add the following code to the `fibiprops.wasm.kt` file:

    ```kotlin
    package io.github.kotlin.fibonacci
    
    actual val firstElement: Int = 3
    actual val secondElement: Int = 5
    ```

    This code sets a Wasm-specific implementation, defining `actual` values for `firstElement` as `3` and `secondElement` as `5`.

### Build the project

Make sure your project compiles correctly with the new platform:

1. Open the Gradle tool window by selecting **View** | **Tool Windows** | **Gradle**.
2. In **multiplatform-library-template** | **library** | **Tasks** | **build**, run the **build** task.

   ![Gradle tool window](library-gradle-build-window-tasks.png){width=450}

   Alternatively, run the following command in the terminal from the `multiplatform-library-template` root directory:

   ```bash
   ./gradlew build
   ```

You can see the successful output in the **Build** tool window. 

## Publish your library to the local Maven repository

Your multiplatform library is ready for publishing locally so that you can use it in other projects on the same machine.

To publish your library, use the [`maven-publish`](https://docs.gradle.org/current/userguide/publishing_maven.html) Gradle plugin as follows:

1. In the `library/build.gradle.kts` file, locate the `plugins { }` block and apply the `maven-publish` plugin:

   ```kotlin
      plugins {
          // ...
          // Add the following line:
          id("maven-publish")
      }
   ```

2. Locate the `mavenPublishing { }` block and comment out the `signAllPublications()`
   method to indicate that the publication is local-only:

    ```kotlin
    mavenPublishing{
        // ...
        // Comment out the following method:
        // signAllPublications()
    }
    ```

3. Synchronize the Gradle files by clicking the **Sync Gradle Changes** icon (![Gradle sync icon](gradle-sync-icon.png){width=30}{type="joined"}) that appears in your build file. Alternatively,
   click the refresh button on the Gradle tool window.

4. In the Gradle tool window, go to **multiplatform-library-template** | **Tasks** | **publishing** and run the **publishToMavenLocal** Gradle task.

   ![Multiplatform library Gradle tool window](publish-maven-local-gradle-task.png){width=450}

   Alternatively, run the following command in the terminal from the `multiplatform-library-template` root directory:

   ```bash
   ./gradlew publishToMavenLocal
   ```

Your library is published to the local Maven repository. 

To locate your published library, use your file explorer or terminal and navigate to the following directory:

```text
C:\Users\<your-username>\.m2\repository\io\github\kotlin\library\1.0.0\
```

## Add your library as a dependency in another project

After publishing your Multiplatform library to the local Maven repository, you can use it in other Kotlin projects on the same machine.

In your consumer project's `build.gradle.kts` file, add a dependency on the published library:

```kotlin
repositories {
    // ...
    mavenLocal()
}

dependencies {
    // ...
    implementation("io.github.kotlin:library:1.0.0")
}
```

The `repositories{}` block tells Gradle to resolve the library from the local Maven repository and make it available in shared code.

The `implementation` dependency consists of your library's group and version specified in its `build.gradle.kts` file.

If you're adding it to another multiplatform project, you can add it to shared or platform-specific source sets:

```kotlin
kotlin {
    sourceSets {
        // For all platforms
        val commonMain by getting {
            dependencies {
                implementation("io.github.kotlin:library:1.0.0")
            }
        }
        // Or for specific platforms
        val wasmJsMain by getting {
            dependencies {
                implementation("io.github.kotlin:library:1.0.0")
            }
        }
    }
}
```

Sync the consumer project and start using your library!

## What's next

We encourage you to explore multiplatform development further:

* [Publish your library to Maven Central](multiplatform-publish-libraries.md)
* [Create an app with shared logic and native UI](multiplatform-create-first-app.md)
* [Create an app with shared logic and UI](compose-multiplatform-create-first-app.md)
* [Publish your application](multiplatform-publish-apps.md)

Join the community:

* ![GitHub](git-hub.svg){width=25}{type="joined"} **Compose Multiplatform GitHub**: star [the repository](https://github.com/JetBrains/compose-multiplatform) and contribute
* ![Slack](slack.svg){width=25}{type="joined"} **Kotlin Slack**: Get
  an [invitation](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up) and join
  the [#multiplatform](https://kotlinlang.slack.com/archives/C3PQML5NU) channel
* ![Stack Overflow](stackoverflow.svg){width=25}{type="joined"} **Stack Overflow**: Subscribe to
  the ["kotlin-multiplatform" tag](https://stackoverflow.com/questions/tagged/kotlin-multiplatform)
* ![YouTube](youtube.svg){width=25}{type="joined"} **Kotlin YouTube channel**: Subscribe and watch videos
  about [Kotlin Multiplatform](https://www.youtube.com/playlist?list=PLlFc5cFwUnmy_oVc9YQzjasSNoAk4hk_C)