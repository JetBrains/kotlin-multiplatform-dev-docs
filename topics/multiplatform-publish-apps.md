[//]: # (title: Publish your application)

Once your apps are ready for release, it's time to deliver them to the users by publishing them.

For mobile apps, multiple stores are available for each platform. However, in this article, we'll focus on the official ones:
[Google Play Store](https://play.google.com/store) and [Apple App Store](https://www.apple.com/ios/app-store/). For web apps, we'll use [GitHub pages](https://pages.github.com/). 

You'll learn how to prepare Kotlin Multiplatform applications for publishing, and we'll highlight
the parts of this process that deserve special attention.

## Android app

Since [Kotlin is the main language for Android development](https://developer.android.com/kotlin),
Kotlin Multiplatform has no obvious effect on compiling the project and building the Android app. Both the Android library produced from
the shared module and the Android app itself are typical Android Gradle modules; they are no different from other Android
libraries and apps. Thus, publishing the Android app from a Kotlin Multiplatform project is no different from the usual process described
in the [Android developer documentation](https://developer.android.com/studio/publish).

## iOS app

The iOS app from a Kotlin Multiplatform project is built from a typical Xcode project, so the main stages involved in publishing it are
the same as described in the [iOS developer documentation](https://developer.apple.com/ios/submit/).

> With Spring'24 changes to App Store policy, missing or incomplete privacy manifests may lead to warnings or even rejection
> for your app.
> For details and workarounds, particularly for Kotlin Multiplatform apps, see [Privacy manifest for iOS apps](https://kotlinlang.org/docs/apple-privacy-manifest.html). 
>
{style="note"}

What is specific to Kotlin Multiplatform projects is compiling the shared Kotlin module into a framework and linking it to the Xcode project.
Generally, integration between the shared module and the Xcode project is done automatically by the [Kotlin Multiplatform plugin for Android Studio](https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform-mobile).
However, if you don't use the plugin, bear in mind the following when building and bundling the iOS project in Xcode:

* The shared Kotlin library compiles down to the native framework.
* You need to connect the framework compiled for the specific platform to the iOS app project.
* In the Xcode project settings, specify the path to the framework to search for the build system.
* After building the project, you should launch and test the app to make sure that there are no issues when working with the framework in runtime.

There are two ways you can connect the shared Kotlin module to the iOS project:
* Use the [Kotlin CocoaPods Gradle plugin](multiplatform-cocoapods-overview.md), which allows you to use a multiplatform project with native targets as a CocoaPods dependency in your iOS project.
* Manually configure your Multiplatform project to create an iOS framework and the Xcode project to obtain its latest version.
  The Kotlin Multiplatform wizard or Kotlin Multiplatform plugin for Android Studio usually does this configuration.
  See [Connect the framework to your iOS project](multiplatform-integrate-in-existing-app.md#configure-the-ios-project-to-use-a-kmp-framework)
  to learn how to add the framework directly in Xcode.

### Configure your iOS application

You can configure basic properties that affect the resulting app without Xcode.

#### Bundle ID

The [bundle ID](https://developer.apple.com/documentation/bundleresources/information_property_list/cfbundleidentifier#discussion)
uniquely identifies your app in the operating system. To change it,
open the `iosApp/Configuration/Config.xcconfig` file in Android Studio and update the `BUNDLE_ID`.

#### App name

The app name sets the target executable and application bundle name. To change your app name:

* If you have _not_ opened the project in Android Studio yet, you can change the `APP_NAME` option in the
  `iosApp/Configuration/Config.xcconfig` file directly in any text editor.
* If you've already opened the project in Android Studio, do the following:

  1. Close the project.
  2. In any text editor, change the `APP_NAME` option in the `iosApp/Configuration/Config.xcconfig` file.
  3. Re-open the project in Android Studio.

If you need to configure other settings, use Xcode: after opening the project in Android Studio,
open the `iosApp/iosApp.xcworkspace` file in Xcode and make changes there.

### Symbolicating crash reports

To help developers make their apps better, iOS provides a means for analyzing app crashes. For detailed crash analysis,
it uses special debug symbol (`.dSYM`) files that match memory addresses in crash reports with locations in the source code,
such as functions or line numbers.

By default, the release versions of iOS frameworks produced from the shared Kotlin module have an accompanying `.dSYM`
file. This helps you analyze crashes that happen in the shared module's code.

For more information on crash report symbolication, see the [Kotlin/Native documentation](https://kotlinlang.org/docs/native-debugging.html#debug-ios-applications).

## Web app

To publish your web application, create the artifacts containing the compiled files 
and resources that make up your application. These artifacts are necessary to deploy your application to a web hosting platform like GitHub Pages.

### Generate artifacts

Create a run configuration for running the **wasmJsBrowserDistribution** task:

1. Select the **Run | Edit Configurations** menu item.
2. Click the plus button and choose **Gradle** from the dropdown list.
3. In the **Tasks and arguments** field, paste this command:

   ```shell
   wasmJsBrowserDistribution
   ```

4. Click **OK**.

Now, you can use this configuration to run the task:

![Run the Wasm distribution task](compose-run-wasm-distribution-task.png){width=350}

Once the task completes, you can find the generated artifacts in the `composeApp/build/dist/wasmJs/productionExecutable`
directory:

![Artifacts directory](compose-web-artifacts.png){width=400}

### Publish your application on GitHub Pages

With the artifacts ready, you can deploy your application on the web hosting platform:

1. Copy the contents of your `productionExecutable` directory into the repository where you want to create a site.
2. Follow GitHub's instructions for [creating your site](https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-github-pages-site#creating-your-site).

   > It can take up to 10 minutes for changes to your site to publish after you push the changes to GitHub.
   >
   {style="note"}

3. In a browser, navigate to your GitHub pages domain.

   ![Navigate to GitHub pages](publish-your-application-on-web.png){width=650}

   Congratulations! You have published your artifacts on GitHub pages.

### Debug your web application

You can debug your web application in your browser out of the box, without additional configurations. To learn how to debug
in the browser, see the [Debug in your browser](https://kotlinlang.org/docs/wasm-debugging.html#debug-in-your-browser)
guide in Kotlin docs.