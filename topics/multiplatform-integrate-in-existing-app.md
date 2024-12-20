[//]: # (title: Make your Android application work on iOS – tutorial)

Learn how to make your existing Android application cross-platform so that it works both on Android and iOS.
You'll be able to write code and test it for both Android and iOS only once, in one place.

This tutorial uses a [sample Android application](https://github.com/Kotlin/kmp-integration-sample) with a single screen
for entering a username and password. The credentials are validated and saved to an in-memory database.

> If you aren't familiar with Kotlin Multiplatform, learn how to [set up environment and create a cross-platform application from scratch](multiplatform-setup.md)
> first.
>
{style="tip"}

## Prepare an environment for development

1. [Install all the necessary tools and update them to the latest versions](multiplatform-setup.md).

   > You will need a Mac with macOS to complete certain steps in this tutorial, which include writing iOS-specific code
   > and running an iOS application. These steps can't be performed on other operating systems, such as Microsoft Windows.
   > This is due to an Apple requirement.
   >
   {style="note"}

2. In Android Studio, create a new project from version control:

   ```text
   https://github.com/Kotlin/kmp-integration-sample
   ```

   > The `master` branch contains the project's initial state — a simple Android application. To see the final state
   > with the iOS application and the shared module, switch to the `final` branch.
   >
   {style="tip"}

3. Switch to the **Project** view.

   ![Project view](select-project-view.png){width=200}

## Make your code cross-platform

To make your application work on iOS, you'll first make your code cross-platform, and then you'll reuse your
cross-platform code in a new iOS application.

To make your code cross-platform:

1. [Decide what code to make cross-platform](#decide-what-code-to-make-cross-platform).
2. [Create a shared module for cross-platform code](#create-a-shared-module-for-cross-platform-code).
3. [Add a dependency on the shared module to your Android application](#add-a-dependency-on-the-shared-module-to-your-android-application).
4. [Make the business logic cross-platform](#make-the-business-logic-cross-platform).
5. [Run your cross-platform application on Android](#run-your-cross-platform-application-on-android).

### Decide what code to make cross-platform

Decide which code of your Android application is better to share for iOS and which to keep native. A simple rule is:
share what you want to reuse as much as possible. The business logic is often the same for both Android and iOS,
so it's a great candidate for reuse.

In your sample Android application, the business logic is stored in the package `com.jetbrains.simplelogin.androidapp.data`.
Your future iOS application will use the same logic, so you should make it cross-platform, as well.

![Business logic to share](business-logic-to-share.png){width=350}

### Create a shared module for cross-platform code

> You can find the sample project with the shared module already added
> in the [shared_module](https://github.com/Kotlin/kmp-integration-sample/tree/shared_module) branch of the GitHub repository.
> 
{style="tip"}

The cross-platform code that is used for both iOS and Android will be stored in a shared module.
The Kotlin Multiplatform plugin for Android Studio provides a wizard for creating such modules.

Create a shared module and connect it to both the existing Android application and your future iOS application:

1. In Android Studio settings, select the **Advanced Settings** section and turn on the **Enable experimental Multiplatform IDE features** option.
2. Restart Android Studio for the changes to take effect.
3. Select **File** | **New** | **New Module** from the main menu.
4. In the list of templates, select **Java or Kotlin Library**.
   Enter the library name `shared` and the package name `com.jetbrains.simplelogin.shared`.
5. Click **Finish**.
   The wizard creates a base module that you'll expand into a Kotlin Multiplatform module.
6. In the root `build.gradle.kts` file, replace the contents with the following code to properly apply Gradle plugins:

   ```kotlin
   plugins {
       alias(libs.plugins.androidApplication) apply false
       alias(libs.plugins.kotlinAndroid) apply false
       alias(libs.plugins.kotlinMultiplatform) apply false
       alias(libs.plugins.androidLibrary) apply false
   }
   ```

7. In the `shared/build.gradle.kts` file, define the necessary KMP targets.
   To do that, replace the contents of the file with the following code:

    ```kotlin
    import org.jetbrains.kotlin.gradle.ExperimentalKotlinGradlePluginApi
    import org.jetbrains.kotlin.gradle.dsl.JvmTarget
    
    plugins {
        alias(libs.plugins.kotlinMultiplatform)
        alias(libs.plugins.androidLibrary)
    }
    
    kotlin {
        androidTarget {
            @OptIn(ExperimentalKotlinGradlePluginApi::class)
            compilerOptions {
                jvmTarget.set(JvmTarget.JVM_11)
            }
        }
        
        listOf(
            iosX64(),
            iosArm64(),
            iosSimulatorArm64()
        ).forEach { iosTarget ->
            iosTarget.binaries.framework {
                baseName = "Shared"
                isStatic = true
            }
        }
        
        sourceSets {
            commonMain.dependencies {
                // Contains your multiplatform dependencies
            }
        }
    }
    
    android {
        namespace = "com.jetbrains.simplelogin.shared"
        compileSdk = libs.versions.android.compileSdk.get().toInt()
        compileOptions {
            sourceCompatibility = JavaVersion.VERSION_11
            targetCompatibility = JavaVersion.VERSION_11
        }
        defaultConfig {
            minSdk = libs.versions.android.minSdk.get().toInt()
        }
    }
    ```
    {initial-collapse-state="collapsed" collapsible="true"  collapsed-title="kotlin { ... }"}

8. Sync the Gradle files as suggested by the IDE or using the **File** | **Sync Project with Gradle Files** menu item.

9. In the `shared/src` directory, create `androidMain/kotlin`, `commonMain/kotlin`, and `iosMain/kotlin` directories.
10. In the `shared/src` directory, delete the `main` directory.
11. Inside those directories, create packages and files to replicate the following structure:

   ![Final file structure inside the shared directory](shared-directory-structure.png){width="363"}

12. Add code to the files that you created:

     * For `commonMain/Platform.kt`:

         ```kotlin
         package com.jetbrains.simplelogin.shared
    
         interface Platform {
             val name: String
         }
        
         expect fun getPlatform(): Platform
         ```
     * For `commonMain/Greeting.kt`:

         ```kotlin
         package com.jetbrains.simplelogin.shared

         class Greeting {
             private val platform = getPlatform()

             fun greet(): String {
                 return "Hello, ${platform.name}!"
             }
         }
         ```
     * For `androidMain/Platform.android.kt`:

         ```kotlin
         package com.jetbrains.simplelogin.shared

         import android.os.Build

         class AndroidPlatform : Platform {
             override val name: String = "Android ${Build.VERSION.SDK_INT}"
         }

         actual fun getPlatform(): Platform = AndroidPlatform()
         ```
     * For `iosMain/Platform.ios.kt`:

         ```kotlin
         package com.jetbrains.simplelogin.shared

         import platform.UIKit.UIDevice

         class IOSPlatform: Platform {
             override val name: String = UIDevice.currentDevice.systemName() + " " + UIDevice.currentDevice.systemVersion
         }

         actual fun getPlatform(): Platform = IOSPlatform()
         ```

13. In the `app/build.gradle.kts` file, set the `android.defaultConfig.minSdk` value to 24.
14. Sync the Gradle files as suggested by the IDE or using the **File** | **Sync Project with Gradle Files** menu item.

You can find the resulting state of the project in the [shared_module](https://github.com/Kotlin/kmp-integration-sample/tree/shared_module) branch of the GitHub repository.

If you want to better understand the layout of the resulting project, see [basics of Kotlin Multiplatform project structure](https://kotlinlang.org/docs/multiplatform-discover-project.html).

### Add a dependency on the shared module to your Android application

To use cross-platform code in your Android application, connect the shared module to it, move the business logic code
there, and make this code cross-platform.

1. Add a dependency on the shared module to the `app/build.gradle.kts` file:

    ```kotlin
    dependencies {
        // ...
        implementation(project(":shared"))
    }
    ```

2. Sync the Gradle files as suggested by the IDE or using the **File** | **Sync Project with Gradle Files** menu item.

   ![Synchronize the Gradle files](gradle-sync.png)

3. In the `app/src/main/java/` directory, open the `LoginActivity.kt` file in the `com.jetbrains.simplelogin.androidapp.ui.login`
   package.
4. To make sure that the shared module is successfully connected to your application, dump the `greet()` function
   result to the log by adding a line to the `onCreate()` method:

    ```kotlin
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        Log.i("Login Activity", "Hello from shared module: " + (Greeting().greet()))
   
        // ...
    }
    ```
5. Follow Android Studio's suggestions to import missing classes.
6. In the toolbar, select `app` from the dropdown and click **Debug** ![](debug-android.png){width=20}.

   ![App from list to debug](app-list-android.png){width="300"}

7. On the **Logcat** tab, search for `Hello` in the log, and you'll find the greeting from the shared
   module.

   ![Greeting from the shared module](shared-module-greeting.png){width="700"}

### Make the business logic cross-platform

You can now extract the business logic code to the Kotlin Multiplatform shared module and make it platform-independent.
This is necessary for reusing the code for both Android and iOS.

1. Move the business logic code `com.jetbrains.simplelogin.androidapp.data` from the `app` directory to
   the `com.jetbrains.simplelogin.shared` package in the `shared/src/commonMain` directory.

   ![Drag and drop the package with the business logic code](moving-business-logic.png){width=350}

2. When Android Studio asks what you'd like to do, select to move the package, and then approve the refactoring.

   ![Refactor the business logic package](refactor-business-logic-package.png){width=500}

3. Ignore all warnings about platform-dependent code and click **Continue**.

   ![Warnings about platform-dependent code](warnings-android-specific-code.png){width=450}

4. Remove Android-specific code by replacing it with cross-platform Kotlin code or connecting to Android-specific APIs
   using [expected and actual declarations](multiplatform-connect-to-apis.md). See the following sections for details:

#### Replace Android-specific code with cross-platform code {initial-collapse-state="collapsed" collapsible="true"}

To make your code work well on both Android and iOS, replace all JVM dependencies with Kotlin dependencies in the
moved `data` directory wherever possible.

1. In the `LoginDataSource` class, replace `IOException` in the `login()` function with `RuntimeException`.
   `IOException` is not available in Kotlin/JVM.

    ```kotlin
    // Before
    return Result.Error(IOException("Error logging in", e))
    ```

    ```kotlin
    // After
    return Result.Error(RuntimeException("Error logging in", e))
    ```

2. Remove the import directive for `IOException` as well:

    ```kotlin
    import java.io.IOException
    ```

3. In the `LoginDataValidator` class, replace the `Patterns` class from the `android.utils` package with a Kotlin
   regular expression matching the pattern for email validation:

    ```kotlin
    // Before
    private fun isEmailValid(email: String) = Patterns.EMAIL_ADDRESS.matcher(email).matches()
    ```

    ```kotlin
    // After
    private fun isEmailValid(email: String) = emailRegex.matches(email)
    
    companion object {
        private val emailRegex = 
            ("[a-zA-Z0-9\\+\\.\\_\\%\\-\\+]{1,256}" +
                "\\@" +
                "[a-zA-Z0-9][a-zA-Z0-9\\-]{0,64}" +
                "(" +
                "\\." +
                "[a-zA-Z0-9][a-zA-Z0-9\\-]{0,25}" +
                ")+").toRegex()
    }
    ```

4. And remove the import directive for the `Patterns` class:

    ```kotlin
    import android.util.Patterns
    ```

#### Connect to platform-specific APIs from the cross-platform code {initial-collapse-state="collapsed" collapsible="true"}

In the `LoginDataSource` class, a universally unique identifier (UUID) for `fakeUser` is generated using
the `java.util.UUID` class, which is not available for iOS.

```kotlin
val fakeUser = LoggedInUser(java.util.UUID.randomUUID().toString(), "Jane Doe")
```

Since the Kotlin standard library doesn't provide functionality for generating UUIDs, you still need to use
platform-specific functionality for this case.

Provide the `expect` declaration for the `randomUUID()` function in the shared code and its `actual` implementations for
each platform – Android and iOS – in the corresponding source sets.
You can learn more about [connecting to platform-specific APIs](multiplatform-connect-to-apis.md).

1. Remove the `java.util.UUID` class from the common code:

    ```kotlin
    val fakeUser = LoggedInUser(randomUUID(), "Jane Doe")
    ```

2. Create the `Utils.kt` file in the `com.jetbrains.simplelogin.shared` package of the `shared/src/commonMain` directory
   and provide the `expect` declaration:

    ```kotlin
    package com.jetbrains.simplelogin.shared
    
    expect fun randomUUID(): String
    ```

3. Create the `Utils.android.kt` file in the `com.jetbrains.simplelogin.shared` package of the `shared/src/androidMain`
   directory and provide the `actual` implementation for `randomUUID()` in Android:

    ```kotlin
    package com.jetbrains.simplelogin.shared
    
    import java.util.*
   
    actual fun randomUUID() = UUID.randomUUID().toString()
    ```

4. Create the `Utils.ios.kt` file in the `com.jetbrains.simplelogin.shared` of the `shared/src/iosMain` directory and
   provide the `actual` implementation for `randomUUID()` in iOS:

    ```kotlin
    package com.jetbrains.simplelogin.shared
    
    import platform.Foundation.NSUUID
   
    actual fun randomUUID(): String = NSUUID().UUIDString()
    ```

5. All that is left to do is to explicitly import `randomUUID` in the `LoginDataSource.kt` file of the `shared/src/commonMain`
   directory:

   ```kotlin
   import com.jetbrains.simplelogin.shared.randomUUID
   ```

   Now, Kotlin will use different platform-specific implementations of UUID for Android and iOS.

### Run your cross-platform application on Android

Run your cross-platform application for Android to make sure it works.

![Android login application](android-login.png){width=300}

## Make your cross-platform application work on iOS

Once you've made your Android application cross-platform, you can create an iOS application and reuse the shared
business logic in it.

1. [Create an iOS project in Xcode](#create-an-ios-project-in-xcode).
2. [Connect the framework to your iOS project](#connect-the-framework-to-your-ios-project).
3. [Use the shared module from Swift](#use-the-shared-module-from-swift).

### Create an iOS project in Xcode

1. In Xcode, click **File** | **New** | **Project**.
2. Select a template for an iOS app and click **Next**.

   ![iOS project template](ios-project-wizard-1.png){width=700}

3. As the product name, specify **simpleLoginIOS** and click **Next**.

   ![iOS project settings](ios-project-wizard-2.png){width=700}

4. As the location for your project, select the directory that stores your cross-platform application, for
   example, `kmp-integration-sample`.

In Android Studio, you'll get the following structure:

![iOS project in Android Studio](ios-project-in-as.png){width=194}

You can rename the `simpleLoginIOS` directory to `iosApp` for consistency with other top-level directories of your
cross-platform project.
To do that, close Xcode and then rename the `simpleLoginIOS` directory to `iosApp`.
If you rename the folder with Xcode open, you'll get a warning and may corrupt your project.

![Renamed iOS project directory in Android Studio](ios-directory-renamed-in-as.png){width=194}

### Connect the framework to your iOS project

Once you have the framework, you can connect it to your iOS project manually.

> An alternative is to [configure integration via CocoaPods](https://kotlinlang.org/docs/native-cocoapods.html), but that integration is beyond the
> scope of this tutorial.
>
{style="note"}

Connect your framework to the iOS project manually:

1. In Xcode, open the iOS project settings by double-clicking the project name.

2. On the **Build Phases** tab of the project settings, click the **+** and add **New Run Script Phase**.

   ![Add run script phase](xcode-run-script-phase-1.png){width=700}

3. Add the following script:

    ```text
    cd "$SRCROOT/.."
    ./gradlew :shared:embedAndSignAppleFrameworkForXcode
    ```

   ![Add the script](xcode-add-run-phase-2.png){width=700}

4. Move the **Run Script** phase before the **Compile Sources** phase.

   ![Move the Run Script phase](xcode-run-script-phase-3.png){width=700}

5. On the **Build Settings** tab, disable the **User Script Sandboxing** under **Build Options**:

   ![User Script Sandboxing](disable-sandboxing-in-xcode-project-settings.png){width=700}

   > This may require restarting your Gradle daemon, if you built the iOS project without disabling sandboxing first.
   > Stop the Gradle daemon process that might have been sandboxed:
   > ```shell
   > ./gradlew --stop
   > ```
   >
   > {style="tip"}

6. Build the project in Xcode. If everything is set up correctly, the project will build successfully.

> If you have a custom build configuration different from the default `Debug` or `Release`, on the **Build Settings**
> tab, add the `KOTLIN_FRAMEWORK_BUILD_TYPE` setting under **User-Defined** and set it to `Debug` or `Release`.
>
{style="note"}

### Use the shared module from Swift

1. In Xcode, open the `ContentView.swift` file and import the `shared` module:

   ```swift
   import shared
   ```

2. To check that it is properly connected, use the `greet()` function from the shared module of your cross-platform app:

   ```swift
   import SwiftUI
   import shared
   
   struct ContentView: View {
       var body: some View {
           Text(Greeting().greet())
           .padding()
       }
   }
   ```

3. Run the app from Xcode to see the result:

   ![Greeting from the shared module](xcode-iphone-hello.png){width=300}

4. In the `ContentView.swift` file, write code for using data from the shared module and rendering the application UI:

   ```kotlin
   ```
   {src="android-ios-tutorial/ContentView.swift" initial-collapse-state="collapsed" collapsible="true"}

5. In `simpleLoginIOSApp.swift`, import the `shared` module and specify the arguments for the `ContentView()` function:

    ```swift
    import SwiftUI
    import shared
    
    @main
    struct SimpleLoginIOSApp: App {
        var body: some Scene {
            WindowGroup {
                ContentView(viewModel: .init(loginRepository: LoginRepository(dataSource: LoginDataSource()), loginValidator: LoginDataValidator()))
            }
        }
    }
    ```

6. Run the Xcode project to see that the iOS app shows the login form. Enter "Jane" for the username and "password" for the password.
   The app validates the input using the shared code:

   ![Simple login application](xcode-iphone-login.png){width=300}

## Enjoy the results – update the logic only once

Now your application is cross-platform. You can update the business logic in one place and see results on both Android
and iOS.

1. In Android Studio, change the validation logic for a user's password: "password" shouldn't be a valid option.
   To do that, update the `checkPassword()` function of the `LoginDataValidator` class:

   ```kotlin
   package com.jetbrains.simplelogin.shared.data
   
   class LoginDataValidator {
   //...
       fun checkPassword(password: String): Result {
           return when {
               password.length < 5 -> Result.Error("Password must be >5 characters")
               password.lowercase() == "password" -> Result.Error("Password shouldn't be \"password\"")
               else -> Result.Success
           }
       }
   //...
   }
   ```

2. In Android Studio, add a run configuration for the iOS app:

    1. Select **Run | Edit configurations** in the main menu.

    2. To add a new configuration, click the plus sign and choose **iOS Application**.

    3. Name the configuration "SimpleLoginIOS".

    4. In the **Xcode project file** field, select the location of the `simpleLoginIOS.xcodeproj` file.

    5. Choose a simulation environment in the **Execution target** list and click **OK**.

3. Run both the iOS and Android applications from Android Studio to see the changes:

   ![iOS run configuration](ios-run-configuration-simplelogin.png){width=300}

   ![iOS application password error](iphone-password-error.png){width=300}

   ![Android application password error](android-password-error.png){width=300}

You can review the [final code for this tutorial](https://github.com/Kotlin/kmp-integration-sample/tree/final).

## What else to share?

You've shared the business logic of your application, but you can also decide to share other layers of your application.
For example, the `ViewModel` class code is almost the same for [Android](https://github.com/Kotlin/kmp-integration-sample/blob/final/app/src/main/java/com/jetbrains/simplelogin/androidapp/ui/login/LoginViewModel.kt)
and [iOS applications](https://github.com/Kotlin/kmp-integration-sample/blob/final/iosApp/SimpleLoginIOS/ContentView.swift#L84),
and you can share it if your mobile applications should have the same presentation layer.

## What's next?

Once you've made your Android application cross-platform, you can move on and:

* [Add dependencies on multiplatform libraries](https://kotlinlang.org/docs/multiplatform-add-dependencies.html)
* [Add Android dependencies](https://kotlinlang.org/docs/multiplatform-android-dependencies.html)
* [Add iOS dependencies](https://kotlinlang.org/docs/multiplatform-ios-dependencies.html)

You can also check out community resources:

* [Video: How to migrate an Android project to Kotlin Multiplatform](https://www.youtube.com/watch?v=vb-Pt8SdfEE&t=1s)
* [Video: 3 ways to get your Kotlin JVM code ready for Kotlin Multiplatform](https://www.youtube.com/watch?v=X6ckI1JWjqo)
