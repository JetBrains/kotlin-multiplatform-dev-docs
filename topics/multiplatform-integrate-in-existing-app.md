[//]: # (title: Make your Android application work on iOS – tutorial)

<secondary-label ref="IntelliJ IDEA"/>
<secondary-label ref="Android Studio"/>

<tldr>
<p>This tutorial uses IntelliJ IDEA, but you can also follow it in Android Studio – both IDEs share the same core functionality and Kotlin Multiplatform support.</p>
</tldr>

Learn how to make your existing Android application cross-platform so that it works both on Android and iOS.
You'll be able to write code and test it for both Android and iOS only once, in one place.

This tutorial uses a [sample Android application](https://github.com/Kotlin/kmp-integration-sample) with a single screen
for entering a username and password. The credentials are validated and saved to an in-memory database.

To make your application work on both iOS and Android,
you'll first make your code cross-platform by moving some of it to a shared module.
After that you'll use your cross-platform code in the Android application, and then you'll use the same code in a new iOS application.

> If you aren't familiar with Kotlin Multiplatform, learn how to [create a cross-platform application from scratch](quickstart.md)
> first.
>
{style="tip"}

## Prepare an environment for development

1. In the [quickstart](quickstart.md), complete the instructions to [set up your environment for Kotlin Multiplatform development](quickstart.md#set-up-the-environment).

   > You will need a Mac with macOS to complete certain steps in this tutorial, which include writing iOS-specific code
   > and running an iOS application. These steps can't be performed on other operating systems, such as Microsoft Windows.
   > This is due to an Apple requirement.
   >
   {style="note"}

2. In IntelliJ IDEA, create a new project from version control:

   ```text
   https://github.com/Kotlin/kmp-integration-sample
   ```

   > The `master` branch contains the project's initial state – a simple Android application. To see the final state
   > with the iOS application and the shared module, switch to the `final` branch.
   >
   {style="tip"}

3. Switch to the **Project** view:

   ![Project view](switch-to-project.png){width="513"}

## Make your code cross-platform

To make your code cross-platform, you'll follow these steps:

1. [Decide what code to make cross-platform](#decide-what-code-to-make-cross-platform)
2. [Create a shared module for cross-platform code](#create-a-shared-module-for-cross-platform-code)
3. [Test the code sharing](#add-code-to-the-shared-module)
4. [Add a dependency on the shared module to your Android application](#add-a-dependency-on-the-shared-module-to-your-android-application)
5. [Make the business logic cross-platform](#make-the-business-logic-cross-platform)
6. [Run your cross-platform application on Android](#run-your-cross-platform-application-on-android)

### Decide what code to make cross-platform

Decide which code of your Android application is better to share for iOS and which to keep native. A simple rule is:
share what you want to reuse as much as possible. The business logic is often the same for both Android and iOS,
so it's a great candidate for reuse.

In your sample Android application, the business logic is stored in the package `com.jetbrains.simplelogin.androidapp.data`.
Your future iOS application will use the same logic, so you should make it cross-platform, as well.

![Business logic to share](business-logic-to-share.png){width=366}

### Create a shared module for cross-platform code

The cross-platform code used for both iOS and Android will be stored in a shared module.
Starting with the Meerkat version, Android Studio provides a wizard for creating such shared modules.

Create a shared module to connect to both the existing Android application and your future iOS application:

1. In IntelliJ IDEA, select **File** | **New** | **Module** from the main menu.
2. In the list of generators, select **Android**, then **Kotlin Multiplatform Shared Module**
   (if using Android Studio, select **Kotlin Multiplatform Shared Module** in the list of module templates).
3. Leave the name `shared` and enter this package name:
   ```
   com.jetbrains.simplelogin.shared
   ```
4. Click **Create**. The wizard creates a shared module, changes the build script accordingly, and starts a Gradle sync.
5. When the setup is complete, you will see the following file structure in the `shared` directory:

   ![Final file structure inside the shared directory](shared-directory-structure.png){width="341"}

6. Make sure that the `kotlin.androidLibrary.minSdk` property in the `shared/build.gradle.kts` file is the same as
   the value of that property in the `app/build.gradle.kts` file.

### Add code to the shared module

Now that you have a shared module,
add some common code to be shared in the `commonMain/kotlin/com.jetbrains.simplelogin.shared` directory:

1. Create a new `Greeting` class with the following code:

    ```kotlin
    package com.jetbrains.simplelogin.shared

    class Greeting {
        private val platform = getPlatform()

        fun greet(): String {
            return "Hello, ${platform.name}!"
        }
    }
    ```

2. Replace the code in created files with the following:

     * In `commonMain/Platform.kt`:

         ```kotlin
         package com.jetbrains.simplelogin.shared
       
         interface Platform {
             val name: String
         }
        
         expect fun getPlatform(): Platform
         ```
     
     * In `androidMain/Platform.android.kt`:

         ```kotlin
         package com.jetbrains.simplelogin.shared
         
         import android.os.Build

         class AndroidPlatform : Platform {
             override val name: String = "Android ${Build.VERSION.SDK_INT}"
         }

         actual fun getPlatform(): Platform = AndroidPlatform()
         ```
     * In `iosMain/Platform.ios.kt`:

         ```kotlin
         package com.jetbrains.simplelogin.shared
       
         import platform.UIKit.UIDevice

         class IOSPlatform: Platform {
             override val name: String = UIDevice.currentDevice.systemName() + " " + UIDevice.currentDevice.systemVersion
         }

         actual fun getPlatform(): Platform = IOSPlatform()
         ```

If you want to better understand the layout of the resulting project,
see the [basics of Kotlin Multiplatform project structure](multiplatform-discover-project.md).

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
3. In the `app/src/main/java/` directory, open the `LoginActivity.kt` file in the `com.jetbrains.simplelogin.androidapp.ui.login`
   package.
4. To make sure that the shared module is successfully connected to your application, dump the `greet()` function
   result to the log by adding a `Log.i()` call to the `onCreate()` method:

    ```kotlin
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        Log.i("Login Activity", "Hello from shared module: " + (Greeting().greet()))
   
        // ...
    }
    ```
5. Follow Android Studio's suggestions to import missing classes.
6. In the toolbar, click the `app` dropdown, then click the debug icon:

   ![App from list to debug](app-list-android.png){width="300"}

7. In the **Logcat** tool window, search for "Hello" in the log, and you'll find the greeting from the shared
   module:

   ![Greeting from the shared module](shared-module-greeting.png){width="700"}

### Make the business logic cross-platform

You can now extract the business logic code to the Kotlin Multiplatform shared module and make it platform-independent.
This is necessary for reusing the code for both Android and iOS.

1. Move the business logic code `com.jetbrains.simplelogin.androidapp.data` from the `app` directory to
   the `com.jetbrains.simplelogin.shared` package in the `shared/src/commonMain` directory.

   ![Drag and drop the package with the business logic code](moving-business-logic.png){width=300}

2. When Android Studio asks what you'd like to do, select to move the package and then approve the refactoring.

   ![Refactor the business logic package](refactor-business-logic-package.png){width=300}

3. Ignore all warnings about platform-dependent code and click **Refactor Anyway**.

   ![Warnings about platform-dependent code](warnings-android-specific-code.png){width=450}

4. Remove Android-specific code by replacing it with cross-platform Kotlin code or connecting to Android-specific APIs
   using [expected and actual declarations](multiplatform-connect-to-apis.md). See the following sections for details:

   #### Replace Android-specific code with cross-platform code {initial-collapse-state="collapsed" collapsible="true"}
   
   To make your code work well on both Android and iOS, replace all JVM dependencies with Kotlin dependencies in the
   moved `data` directory wherever possible.

   1. In the `LoginDataValidator` class, replace the `Patterns` class from the `android.utils` package with a Kotlin
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
   
   2. Remove the import directive for the `Patterns` class:
   
       ```kotlin
       import android.util.Patterns
       ```

   3. In the `LoginDataSource` class, replace `IOException` in the `login()` function with `RuntimeException`.
      `IOException` is not available in Kotlin/JVM.

          ```kotlin
          // Before
          return Result.Error(IOException("Error logging in", e))
          ```

          ```kotlin
          // After
          return Result.Error(RuntimeException("Error logging in", e))
          ```

   4. Remove the import directive for `IOException` as well:

       ```kotlin
       import java.io.IOException
       ```


   #### Connect to platform-specific APIs from the cross-platform code {initial-collapse-state="collapsed" collapsible="true"}
   
   In the `LoginDataSource` class, a universally unique identifier (UUID) for `fakeUser` is generated using
   the `java.util.UUID` class, which is not available for iOS.
   
   ```kotlin
   val fakeUser = LoggedInUser(java.util.UUID.randomUUID().toString(), "Jane Doe")
   ```
   
   Even though the Kotlin standard library provides an [experimental class for UUID generation](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.uuid/-uuid/),
   let's use platform-specific functionality for this case to practice doing that.
   
   Provide the `expect` declaration for the `randomUUID()` function in the shared code and its `actual` implementations for
   each platform – Android and iOS – in the corresponding source sets.
   You can learn more about [connecting to platform-specific APIs](multiplatform-connect-to-apis.md).
   
   1. Change the `java.util.UUID.randomUUID()` call in the `login()` function to a `randomUUID()` call, which you will
       implement for each platform:
   
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
   
   5. Import the `randomUUID` function in the `LoginDataSource.kt` file of the `shared/src/commonMain`
      directory:
   
      ```kotlin
      import com.jetbrains.simplelogin.shared.randomUUID
      ```
   
Now, Kotlin will use platform-specific implementations of UUID for Android and iOS.

### Run your cross-platform application on Android

Run your cross-platform application for Android to make sure it works as before.

![Android login application](android-login.png){width=300}

## Make your cross-platform application work on iOS

Once you've made your Android application cross-platform, you can create an iOS application and reuse the shared
business logic in it.

1. [Create an iOS project in Xcode](#create-an-ios-project-in-xcode)
2. [Configure the iOS project to use a KMP framework](#configure-the-ios-project-to-use-a-kmp-framework)
3. [Set up an iOS run configuration in IntelliJ IDEA](#set-up-an-ios-run-configuration-in-android-studio)
4. [Use the shared module in the iOS project](#use-the-shared-module-in-the-ios-project)

### Create an iOS project in Xcode

1. In Xcode, click **File** | **New** | **Project**.
2. Select a template for an iOS app and click **Next**.

   ![iOS project template](ios-project-wizard-1.png){width=700}

3. As the product name, specify "simpleLoginIOS" and click **Next**.

   ![iOS project settings](ios-project-wizard-2.png){width=700}

4. As the location for your project, select the directory that stores your cross-platform application, for
   example, `kmp-integration-sample`.

In IntelliJ IDEA, you'll get the following structure:

![iOS project in IntelliJ IDEA](ios-project-in-as.png){width=194}

You can rename the `simpleLoginIOS` directory to `iosApp` for consistency with other top-level directories of your
cross-platform project.
To do that, close Xcode and then rename the `simpleLoginIOS` directory to `iosApp`.
If you rename the folder with Xcode open, you'll get a warning and may corrupt your project.

![Renamed iOS project directory in IntelliJ IDEA](ios-directory-renamed-in-as.png){width=194}

### Configure the iOS project to use a KMP framework

You can set up integration between the iOS app and the framework built by Kotlin Multiplatform directly.
Alternatives to this method are covered in the [iOS integration methods overview](multiplatform-ios-integration-overview.md),
but they are beyond the scope of this tutorial.

1. In IntelliJ IDEA, right-click the `iosApp/simpleLoginIOS.xcodeproj` directory and select
   **Open In** | **Open In Associated Application** to launch Xcode.
2. In Xcode, open the iOS project settings by double-clicking the project name in the **Project** navigator.

3. In the **Targets** section on the left, select **simpleLoginIOS**, then click the **Build Phases** tab.

4. Click the **+** icon and select **New Run Script Phase**.

    ![Add a run script phase](xcode-run-script-phase-1.png){width=700}

    The new phase is created at the bottom of the list.

5. Click the **>** icon to expand the created **Run Script** item, then paste the following script in the text field:

    ```text
    cd "$SRCROOT/.."
    ./gradlew :shared:embedAndSignAppleFrameworkForXcode
    ```

   ![Add the script](xcode-add-run-phase-2.png){width=700}

6. Move the **Run Script** phase higher in the order, placing it before the **Compile Sources** phase:

   ![Move the Run Script phase](xcode-run-script-phase-3.png){width=700}

7. Click the **Build Settings** tab, then find and disable the **User Script Sandboxing** option under **Build Options**:

   ![User Script Sandboxing](disable-sandboxing-in-xcode-project-settings.png){width=700}

   > If you have a custom build configuration different from the default `Debug` or `Release`, on the **Build Settings**
   > tab, add the `KOTLIN_FRAMEWORK_BUILD_TYPE` setting under **User-Defined** and set it to `Debug` or `Release`.
   >
   {style="note"}

8. Build the project in Xcode (**Product** | **Build** in the main menu).
    If everything is configured correctly, the project should build successfully 
    (you can safely ignore the "build phase will be run during every build" warning)
   
    > Build may fail if you built the project before disabling the **User Script Sandboxing** option:
    > the Gradle daemon process may be sandboxed and needs to be restarted.
    > Stop it before building the project again by running this command in the project directory (`kmp-integration-sample` in our example):
    > ```shell
    > ./gradlew --stop
    > ```

### Set up an iOS run configuration in IntelliJ IDEA

Once you've made sure that Xcode is set up correctly, return to IntelliJ IDEA:

1. Press double **Shift** to open the **Search Everywhere** window and find the option **Sync All Gradle, Swift Package Manager projects**
   (if using Android Studio, the necessary option is called **Sync Project with Gradle Files**).

   IntelliJ IDEA automatically generates a run configuration called **simpleLoginIOS** and marks the `iosApp`
   directory as a linked Xcode project.

2. In the list of run configurations, select **simpleLoginIOS**.
   Choose an iOS emulator and then click **Run** to check that the iOS app runs correctly.

   ![The iOS run configuration in the list of run configurations](ios-run-configuration-simplelogin.png){width=400}

### Use the shared module in the iOS project

The `build.gradle.kts` file of the `shared` module defines the `binaries.framework.baseName`
property for each iOS target as `sharedKit`.
This is the name of the framework that Kotlin Multiplatform builds for the iOS app to consume.

To test the integration, add a call to common code in Swift code:

1. In Android Studio, open the `iosApp/simpleloginIOS/ContentView.swift` file and import the framework:

   ```swift
   import sharedKit
   ```

2. To check that it is properly connected, change the `ContentView` structure to use the `greet()` function
   from the shared module of your cross-platform app:

   ```swift
   struct ContentView: View {
       var body: some View {
           Text(Greeting().greet())
           .padding()
       }
   }
   ```

3. Run the app using the Android Studio iOS run configuration to see the result:

   ![Greeting from the shared module](xcode-iphone-hello.png){width=300}

4. Update code in the `ContentView.swift` file again to use the business logic from the shared module to render the application UI:

   ```kotlin
   ```
   {src="android-ios-tutorial/ContentView.swift" initial-collapse-state="collapsed" collapsible="true"}

5. In the `simpleLoginIOSApp.swift` file, import the `sharedKit` module and specify the arguments for the `ContentView()` function:

    ```swift
    import SwiftUI
    import sharedKit
    
    @main
    struct SimpleLoginIOSApp: App {
        var body: some Scene {
            WindowGroup {
                ContentView(viewModel: .init(loginRepository: LoginRepository(dataSource: LoginDataSource()), loginValidator: LoginDataValidator()))
            }
        }
    }
    ```

6. Run the iOS run configuration again to see that the iOS app shows the login form.
7. Enter "Jane" as the username and "password" as the password.
8. As you have [set up the integration earlier](#configure-the-ios-project-to-use-a-kmp-framework),
    the iOS app validates input using common code:

   ![Simple login application](xcode-iphone-login.png){width=300}

## Enjoy the results – update the logic only once

Now your application is cross-platform. You can update the business logic in the `shared` module and see results on both Android
and iOS.

1. Change the validation logic for a user's password: "password" shouldn't be a valid option.
    To do that, update the `checkPassword()` function of the `LoginDataValidator` class
    (to find it quickly, press **Shift** twice, paste the name of the class, and switch to the **Classes** tab):

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

2. Run both the iOS and Android applications from Android Studio to see the changes:

   ![Android and iOS applications password error](android-iphone-password-error.png){width=600}

You can review the [final code for this tutorial](https://github.com/Kotlin/kmp-integration-sample/tree/final).

## What else to share?

You've shared the business logic of your application, but you can also decide to share other layers of your application.
For example, the `ViewModel` class code is almost the same for [Android](https://github.com/Kotlin/kmp-integration-sample/blob/final/app/src/main/java/com/jetbrains/simplelogin/androidapp/ui/login/LoginViewModel.kt)
and [iOS applications](https://github.com/Kotlin/kmp-integration-sample/blob/final/iosApp/SimpleLoginIOS/ContentView.swift#L84),
and you can share it if your mobile applications should have the same presentation layer.

## What's next?

Once you've made your Android application cross-platform, you can move on and:

* [Add dependencies on multiplatform libraries](multiplatform-add-dependencies.md)
* [Add Android dependencies](multiplatform-android-dependencies.md)
* [Add iOS dependencies](multiplatform-ios-dependencies.md)

You can use Compose Multiplatform to create a unified UI across all platforms:

* [Learn about Compose Multiplatform and Jetpack Compose](compose-multiplatform-and-jetpack-compose.md)
* [Explore available resources for Compose Multiplatform](compose-multiplatform-resources.md)
* [Create an app with shared logic and UI](compose-multiplatform-create-first-app.md)

You can also check out community resources:

* [Video: How to migrate an Android project to Kotlin Multiplatform](https://www.youtube.com/watch?v=vb-Pt8SdfEE&t=1s)
* [Video: 3 ways to get your Kotlin JVM code ready for Kotlin Multiplatform](https://www.youtube.com/watch?v=X6ckI1JWjqo)
