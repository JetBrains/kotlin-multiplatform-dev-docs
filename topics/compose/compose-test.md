[//]: # (title: Testing Compose UI)

UI testing in Compose Multiplatform is implemented using the same finders, assertions, actions, and matchers
as the Jetpack Compose testing APIs. If you're not familiar with them, we highly recommend reading the [Jetpack Compose guide](https://developer.android.com/jetpack/compose/testing)
before you continue with this article.

## How Compose Multiplatform testing is different from Jetpack  

Compose Multiplatform test API does not rely on JUnit's `TestRule` class. Instead, you call the `runComposeUiTest` function and invoke
the test functions on the `ComposeUiTest` receiver.

JUnit-based API is available for desktop targets, however. See [Running JUnit-based tests with Compose for Desktop](#running-junit-based-tests-with-compose-for-desktop).

## Setting up test environment

Create a sample project to write a simple test for. This is best achieved with the [Kotlin Multiplatform wizard](https://kmp.jetbrains.com):
<!-- this only works if the KMP wizard correctly uses the new Resources API!--> 
1. Select the **Android**, **Desktop**, and **Web** options. If you're using a Mac, select **iOS** as well. 
   Make sure that the **Share UI** option is selected.
2. Click the **Download** button and unpack the resulting archive.
3. Open the unpacked folder as a project in Android Studio.
6. Switch the view from **Android** to **Project**:

   ![Select the Project view](select-project-view.png){width=200}

Now, configure test source sets:
1. Create a directory for the test source set: `composeApp/src/commonTest/kotlin`
2. In the `composeApp/build.gradle.kts` file, add the following dependencies:
   ```kotlin
   kotlin {
      //...
      sourceSets {
         // Add a variable for the desktopTest source set 
         val desktopTest by getting
   
         // Add common test dependencies
         commonTest.dependencies {
            implementation(kotlin("test"))
            
            @OptIn(org.jetbrains.compose.ExperimentalComposeLibrary::class)
            implementation(compose.uiTest)
         }
   
         // Add the desktop test dependency
         desktopTest.dependencies { 
            implementation(compose.desktop.currentOs)
         }
      }
   }
   ```
3. For Android instrumented tests to work:
   1. Add the following code to the `androidTarget` block, and follow IDE's suggestions to add missing imports:
      ```kotlin
      kotlin { 
         //...
         androidTarget { 
            @OptIn(ExperimentalKotlinGradlePluginApi::class)
            instrumentedTestVariant { 
               sourceSetTree.set(KotlinSourceSetTree.test)
      
               dependencies {
                  implementation("androidx.compose.ui:ui-test-junit4-android:1.5.4")
                  debugImplementation("androidx.compose.ui:ui-test-manifest:1.5.4")
               }
            }
           //...
         }
         //... 
      }
      ```
   2. Add the following line to the `android.defaultConfig` block:
      ```kotlin
      android {
         //...
         defaultConfig {
            //...
            testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
         }
      }
      ```

Now you are ready to write and run tests.

## Running common tests

In the `composeApp/src/commonTest/kotlin` directory, create a file named `ExampleTest.kt` and copy the following code into it:

```kotlin
import androidx.compose.material.*
import androidx.compose.runtime.*
import androidx.compose.ui.Modifier
import androidx.compose.ui.platform.testTag
import androidx.compose.ui.test.*
import kotlin.test.Test

class ExampleTest {

    @OptIn(ExperimentalTestApi::class)
    @Test
    fun myTest() = runComposeUiTest {
        // Declare the UI state that will be tested
        setContent {
            var text by remember { mutableStateOf("Hello") }
            Text(
                text = text,
                modifier = Modifier.testTag("text")
            )
            Button(
                onClick = { text = "Compose" },
                modifier = Modifier.testTag("button")
            ){
                Text("Click me")
            }
        }

        // Describe assertions and actions with the declared UI which should succeed for the test to pass
        onNodeWithTag("text").assertTextEquals("Hello")
        onNodeWithTag("button").performClick()
        onNodeWithTag("text").assertTextEquals("Compose")
    }
}
```

Note that the UI state being tested here is completely independent of the project code: the entirety of UI state is set
inside the `setContent` call.

To run tests:
* For the desktop target:
  * Click the green run icon in the gutter next to the `myTest()` function and choose **Run... | desktop**.
  * Or, run the following command in the terminal: `./gradlew :composeApp:desktopTest`
* For the iOS Simulator:
  * Click the green run icon in the gutter next to the `myTest()` function and choose **Run... | iosSimulatorArm64**.
  * Or, run the following command in the terminal: `./gradlew :composeApp:iosSimulatorArm64Test`
* For the Wasm target (in a headless browser): run the command `./gradlew :composeApp:wasmJsTest`
* For the Android Emulator: run the command `./gradlew :composeApp:connectedAndroidTest`

### Running tests on Android Emulator

For Android, common Compose Multiplatform tests currently cannot be run as instrumented tests using Android Studio
or Fleet GUI. Use Gradle tasks set up as described above.  If you try to run tests from the gutter in the IDE,
you will get an `FINGERPRINT must not be null` build error.

For more advanced usage of Android Studio tests, including automation, see the [Test your app](https://developer.android.com/studio/test)
article on the Android for Developers portal.

## Running JUnit-based tests with Compose for Desktop

JUnit-based test API is only available for desktop Compose targets.

Create a sample project to write a simple test for, using the [Kotlin Multiplatform wizard](https://kmp.jetbrains.com):
<!-- this only works if the KMP wizard correctly uses the new Resources API!--> 
1. Select only the **Desktop** option.
2. Click the **Download** button and unpack the resulting archive.
3. Open the unpacked folder as a project in Android Studio.
4. Switch the view from **Android** to **Project**:

   ![Select the Project view](select-project-view.png){width=200}

5. Create a directory for tests: `composeApp/src/desktopTest/kotlin`
6. In the `composeApp/build.gradle.kts` file, add the following dependencies:
   ```kotlin
   kotlin {
      //...
      sourceSets {
         //...
         val desktopTest by getting {
            dependencies {
               implementation(compose.desktop.uiTestJUnit4)
               implementation(compose.desktop.currentOs)
            }
         }
      }
   }
   ```

Now, create a test file called `ExampleTest.kt`, then copy the following code into it:

```kotlin
import androidx.compose.material.*
import androidx.compose.runtime.*
import androidx.compose.ui.Modifier
import androidx.compose.ui.test.*
import androidx.compose.ui.platform.testTag
import androidx.compose.ui.test.junit4.createComposeRule
import org.junit.Rule
import org.junit.Test

class ExampleTest {
    @get:Rule
    val rule = createComposeRule()

    @Test
    fun myTest(){
        // Declare the UI state that will be tested
        rule.setContent {
            var text by remember { mutableStateOf("Hello") }
            Text(
                text = text,
                modifier = Modifier.testTag("text")
            )
            Button(
                onClick = { text = "Compose" },
                modifier = Modifier.testTag("button")
            ){
                Text("Click me")
            }
        }

        // Describe assertions and actions with the declared UI which should succeed for the test to pass
        rule.onNodeWithTag("text").assertTextEquals("Hello")
        rule.onNodeWithTag("button").performClick()
        rule.onNodeWithTag("text").assertTextEquals("Compose")
    }
}

```

To run the test, click the run icon in the gutter next to the `myTest` function
or run the following command in the terminal: `./gradlew desktopTest`
