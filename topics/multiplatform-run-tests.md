[//]: # (title: Test your multiplatform app − tutorial)

In this tutorial, you'll learn how to create, configure, and run tests in Kotlin Multiplatform applications.

Tests for multiplatform projects can be divided into two categories:

* Tests for common code. These tests can be run on any platform using any supported framework.
* Tests for platform-specific code. These are essential to test platform-specific logic. They use a platform-specific
  framework and can benefit from its additional features, such as a richer API and a wider range of assertions.

Both categories are supported in multiplatform projects. This tutorial will first show you how to set up, create,
and run unit tests for common code in a simple Kotlin Multiplatform project. Then, you'll work with a more complex example
that requires tests both for common and platform-specific code.

> This tutorial assumes that you are familiar with:
> * The layout of a Kotlin Multiplatform project. If this is not the case,
    complete [this tutorial](multiplatform-create-first-app.md) before you begin.
> * The basics of popular unit testing frameworks, such as [JUnit](https://junit.org/junit5/).
>
{style="tip"}

## Test a simple multiplatform project

### Create a project

1. In the [quickstart](quickstart.md), complete the instructions to [set up your environment for Kotlin Multiplatform development](quickstart.md#set-up-the-environment).
2. In IntelliJ IDEA, select **File** | **New** | **Project**.
3. In the panel on the left, select **Kotlin Multiplatform**.
4. Specify the following fields in the **New Project** window:

    * **Name**: KotlinProject
    * **Group**: kmp.project.demo
    * **Artifact**: kotlinproject
    * **JDK**: Amazon Corretto version 17
        > This JDK version is required for one of the tests that you add later to run successfully.
        >
        {style="note"}

5. Select the **Android** target.
    * If you're using a Mac, select **iOS** as well. Make sure that the **Do not share UI** option is selected.
6. Deselect **Include tests** and click **Create**.

   ![Create simple multiplatform project](create-test-multiplatform-project.png){width=800}

### Write code

In the `shared/src/commonMain/kotlin` directory, create a new `common.example.search` directory.
In this directory, create a Kotlin file, `Grep.kt`, with the following function:

```kotlin
fun grep(lines: List<String>, pattern: String, action: (String) -> Unit) {
    val regex = pattern.toRegex()
    lines.filter(regex::containsMatchIn)
        .forEach(action)
}
```

This function is designed to resemble the [UNIX `grep` command](https://en.wikipedia.org/wiki/Grep). Here, the function
takes lines of text, a pattern used as a regular expression, and a function that is invoked every time a line matches
the pattern.

### Add tests

Now, let's test the common code. An essential part will be a source set for common tests,
which has the [`kotlin.test`](https://kotlinlang.org/api/latest/kotlin.test/) API library as a dependency.

1. In the `shared/build.gradle.kts` file, check that there is a dependency on the `kotlin.test` library:

    ```kotlin
   sourceSets {
       //...
       commonTest.dependencies {
           implementation(libs.kotlin.test)
       }
   }
   ```
   
2. The `commonTest` source set stores all common tests. You need to create a directory with the same name in your project:

    1. Right-click the `shared/src` directory and select **New | Directory**. The IDE presents a list of options.
    2. Start typing the `commonTest/kotlin` path to narrow down the selection, then choose it from the list:

      ![Creating common test directory](create-common-test-dir.png){width=350}

3. In the `commonTest/kotlin` directory, create a new `common.example.search` package.
4. In this package, create the `Grep.kt` file and update it with the following unit test:

    ```kotlin
    import kotlin.test.Test
    import kotlin.test.assertContains
    import kotlin.test.assertEquals
    
    class GrepTest {
        companion object {
            val sampleData = listOf(
                "123 abc",
                "abc 123",
                "123 ABC",
                "ABC 123"
            )
        }
    
        @Test
        fun shouldFindMatches() {
            val results = mutableListOf<String>()
            grep(sampleData, "[a-z]+") {
                results.add(it)
            }
    
            assertEquals(2, results.size)
            for (result in results) {
                assertContains(result, "abc")
            }
        }
    }
    ```

As you can see, imported annotations and assertions are neither platform- nor framework-specific.
When you run this test later, a platform-specific framework will provide the test runner.

#### Explore the `kotlin.test` API {initial-collapse-state="collapsed" collapsible="true"}

The [`kotlin.test`](https://kotlinlang.org/api/latest/kotlin.test/) library provides platform-agnostic
annotations and assertions for you to use in your tests. Annotations, such as `Test`,
map to those provided by the selected framework or their nearest equivalent.

Assertions are executed through an implementation of the [`Asserter` interface](https://kotlinlang.org/api/latest/kotlin.test/kotlin.test/-asserter/).
This interface defines the different checks commonly performed in testing. The API has a default implementation,
but typically you will use a framework-specific implementation.

For example, the JUnit 4, JUnit 5, and TestNG frameworks are all supported on JVM. On Android, a call to `assertEquals()`
might result in a call to `asserter.assertEquals()`, where the `asserter` object is an instance of `JUnit4Asserter`.
On iOS, the default implementation of the `Asserter` type is used in conjunction with the Kotlin/Native test runner.

### Run tests

You can execute the test by running:

* The `shouldFindMatches()` test function using the **Run** icon in the gutter.
* The test file using its context menu.
* The `GrepTest` test class using the **Run** icon in the gutter.

There's also a handy <shortcut>⌃ ⇧ F10</shortcut>/<shortcut>Ctrl+Shift+F10</shortcut> shortcut.
Regardless of the option you choose, you'll see a list of targets to run the test on:

![Run test task](run-test-tasks.png){width=300}

For the `android` option, tests are run using JUnit 4. For `iosSimulatorArm64`, the Kotlin compiler detects testing
annotations and creates a _test binary_ that is executed by Kotlin/Native's own test runner.

Here is an example of the output generated by a successful test run:

![Test output](run-test-results.png){width=700}

## Work with more complex projects

### Write tests for common code

You've already created a test for common code with the `grep()` function. Now, let's consider a more advanced common code
test with the `CurrentRuntime` class. This class contains details of the platform on which the code is executed.
For example, it might have the values "OpenJDK" and "17.0" for Android unit tests that run on a local JVM.

An instance of `CurrentRuntime` should be created with the name and version of the platform as strings, where the
version is optional. When the version is present, you only need the number at the start of the string, if available.

1. In the `commonMain/kotlin` directory, create a new `org.kmp.testing` directory.
2. In this directory, create the `CurrentRuntime.kt` file and update it with the following implementation:

    ```kotlin
    class CurrentRuntime(val name: String, rawVersion: String?) {
        companion object {
            val versionRegex = Regex("^[0-9]+(\\.[0-9]+)?")
        }
    
        val version = parseVersion(rawVersion)
    
        override fun toString() = "$name version $version"
    
        private fun parseVersion(rawVersion: String?): String {
            val result = rawVersion?.let { versionRegex.find(it) }
            return result?.value ?: "unknown"
        }
    }
    ```

3. In the `commonTest/kotlin` directory, create a new `org.kmp.testing` package.
4. In this package, create the `CurrentRuntimeTest.kt` file and update it with the following platform and framework-agnostic test:

    ```kotlin
    import kotlin.test.Test
    import kotlin.test.assertEquals

    class CurrentRuntimeTest {
        @Test
        fun shouldDisplayDetails() {
            val runtime = CurrentRuntime("MyRuntime", "1.1")
            assertEquals("MyRuntime version 1.1", runtime.toString())
        }
    
        @Test
        fun shouldHandleNullVersion() {
            val runtime = CurrentRuntime("MyRuntime", null)
            assertEquals("MyRuntime version unknown", runtime.toString())
        }
    
        @Test
        fun shouldParseNumberFromVersionString() {
            val runtime = CurrentRuntime("MyRuntime", "1.2 Alpha Experimental")
            assertEquals("MyRuntime version 1.2", runtime.toString())
        }
    
        @Test
        fun shouldHandleMissingVersion() {
            val runtime = CurrentRuntime("MyRuntime", "Alpha Experimental")
            assertEquals("MyRuntime version unknown", runtime.toString())
        }
    }
    ```

You can run this test using any of the ways [available in the IDE](#run-tests).

### Add platform-specific tests

> Here, the [mechanism of expected and actual declarations](multiplatform-connect-to-apis.md)
> is used for brevity and simplicity. In more complex code, a better approach is to use interfaces and factory functions.
>
{style="note"}

Now you have experience writing tests for common code, let's explore writing platform-specific tests for Android and iOS.

To create an instance of `CurrentRuntime`, declare a function in the common `CurrentRuntime.kt` file as follows:

```kotlin
expect fun determineCurrentRuntime(): CurrentRuntime
```

The function should have separate implementations for each supported platform. Otherwise, the build will fail.
As well as implementing this function on each platform, you should provide tests. Let's create them for Android and iOS.

#### For Android

1. In the `androidMain/kotlin` directory, create a new `org.kmp.testing` package.
2. In this package, create the `AndroidRuntime.kt` file and update it with the actual implementation of the expected
   `determineCurrentRuntime()` function:

    ```kotlin
    actual fun determineCurrentRuntime(): CurrentRuntime {
        val name = System.getProperty("java.vm.name") ?: "Android"
    
        val version = System.getProperty("java.version")
    
        return CurrentRuntime(name, version)
    }
    ```

3. Create a directory for tests inside the `shared/src` directory:
 
   1. Right-click the `shared/src` directory and select **New | Directory**. The IDE presents a list of options.
   2. Start typing the `androidUnitTest/kotlin` path to narrow down the selection, then choose it from the list:

   ![Creating Android test directory](create-android-test-dir.png){width=350}

4. In the `kotlin` directory, create a new `org.kmp.testing` package.
5. In this package, create the `AndroidRuntimeTest.kt` file and update it with the following Android test:

    ```kotlin
    import kotlin.test.Test
    import kotlin.test.assertContains
    import kotlin.test.assertEquals
    
    class AndroidRuntimeTest {
        @Test
        fun shouldDetectAndroid() {
            val runtime = determineCurrentRuntime()
            assertContains(runtime.name, "OpenJDK")
            assertEquals(runtime.version, "17.0")
        }
    }
    ```
   
   > If you chose a different JDK version at the beginning of the tutorial, you may need to change the `name` and `version` for the test to run successfully.
   > 
   {style="note"}

It may seem strange that an Android-specific test is run on a local JVM. This is because these tests run as local unit
tests on the current machine. As described in the [Android Studio documentation](https://developer.android.com/studio/test/test-in-android-studio),
these tests differ from instrumented tests, which run on a device or an emulator.

You can add other types of tests to your project. To learn about instrumented tests, see this [Touchlab guide](https://touchlab.co/understanding-and-configuring-your-kmm-test-suite/).

#### For iOS

1. In the `iosMain/kotlin` directory, create a new `org.kmp.testing` directory.
2. In this directory, create the `IOSRuntime.kt` file and update it with the actual implementation of the expected
   `determineCurrentRuntime()` function:

    ```kotlin
    import kotlin.experimental.ExperimentalNativeApi
    import kotlin.native.Platform
    
    @OptIn(ExperimentalNativeApi::class)
    actual fun determineCurrentRuntime(): CurrentRuntime {
        val name = Platform.osFamily.name.lowercase()
        return CurrentRuntime(name, null)
    }
    ```

3. Create a new directory in the `shared/src` directory:
   
   1. Right-click the `shared/src` directory and select **New | Directory**. The IDE presents a list of options.
   2. Start typing the `iosTest/kotlin` path to narrow down the selection, then choose it from the list:

   ![Creating iOS test directory](create-ios-test-dir.png){width=350}

4. In the `iosTest/kotlin` directory, create a new `org.kmp.testing` directory.
5. In this directory, create the `IOSRuntimeTest.kt` file and update it with the following iOS test:

    ```kotlin 
    import kotlin.test.Test
    import kotlin.test.assertEquals
    
    class IOSRuntimeTest {
        @Test
        fun shouldDetectOS() {
            val runtime = determineCurrentRuntime()
            assertEquals(runtime.name, "ios")
            assertEquals(runtime.version, "unknown")
        }
    }
    ```

### Run multiple tests and analyze reports

At this stage, you have the code for common, Android, and iOS implementations, as well as their tests.
The directory structure in your project should look something like this:

![Whole project structure](code-and-test-structure.png){width=300}

You can run individual tests from the context menu or use the shortcut. One more option is to use Gradle tasks. For
example, if you run the `allTests` Gradle task, every test in your project will be run with the corresponding test runner:

![Gradle test tasks](gradle-alltests.png){width=700}

When you run tests, in addition to the output in your IDE, HTML reports are generated. You can find them in
the `shared/build/reports/tests` directory:

![HTML reports for multiplatform tests](shared-tests-folder-reports.png){width=300}

Run the `allTests` task and examine the reports it generated:

* The `allTests/index.html` file contains combined reports for common and iOS tests
  (iOS tests depend on common tests and are run after them).
* The `testDebugUnitTest` and `testReleaseUnitTest` folders contain reports for both default Android build flavors.
  (Currently, Android test reports are not automatically merged with the `allTests` report.)

![HTML report for multiplatform tests](multiplatform-test-report.png){width=700}

## Rules for using tests in multiplatform projects

You've now created, configured, and executed tests in Kotlin Multiplatform applications.
When working with tests in your future projects, remember:

* When writing tests for common code, use only multiplatform libraries, like [kotlin.test](https://kotlinlang.org/api/latest/kotlin.test/). Add dependencies to
  the `commonTest` source set.
* The `Asserter` type from the `kotlin.test` API should only be used indirectly.
  Although the `Asserter` instance is visible, you don't need to use it in your tests.
* Always stay within the testing library API. Fortunately,
  the compiler and the IDE prevent you from using framework-specific functionality.
* Although it doesn't matter which framework you use for running tests in `commonTest`, it's a good idea to run your
  tests with each framework you intend to use to check that your development environment is set up correctly.
* When writing tests for platform-specific code, you can use the functionality of the corresponding framework, for example,
  annotations and extensions.
* You can run tests both from the IDE and using Gradle tasks.
* When you run tests, HTML test reports are generated automatically.

## What's next?

* Explore the layout of multiplatform projects in [Understand Multiplatform project structure](https://kotlinlang.org/docs/multiplatform-discover-project.html).
* Check out [Kotest](https://kotest.io/), another multiplatform testing framework provided by the Kotlin ecosystem.
  Kotest allows writing tests in a range of styles and supports complementary approaches to regular testing.
  These include [data-driven](https://kotest.io/docs/framework/datatesting/data-driven-testing.html)
  and [property-based](https://kotest.io/docs/proptest/property-based-testing.html) testing.