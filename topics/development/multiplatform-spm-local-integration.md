[//]: # (title: Using Kotlin from local Swift packages)

> The feature is available starting with 2.0.0-Beta2. To try it out, [install the Kotlin EAP plugin](https://kotlinlang.org/docs/install-eap-plugin.html)
> if you create a new project or [adjust your build](https://kotlinlang.org/docs/configure-build-for-eap.html) for an existing project.
> 
{type="note"}

In this tutorial, you'll learn how to add a dependency on a Kotlin framework in a local package in the Swift package manager.
This is useful if you want to simultaneously connect your Kotlin Multiplatform module to several SPM modules
in your Xcode project.

> If you aren't familiar with Kotlin Multiplatform, learn how to [set up the environment](multiplatform-setup.md)
> and [create a cross-platform application from scratch](multiplatform-create-first-app.md) first.
>
{type="tip"}

The tutorial assumes that your project is using [direct integration](https://www.jetbrains.com/help/kotlin-multiplatform-dev/multiplatform-project-configuration.html#connect-a-kotlin-multiplatform-module-to-an-ios-app) approach. If you're connecting a Kotlin framework through CocoaPods plugin, please migrate first.

### Migrate from CocoaPods plugin to direct integration {initial-collapse-state="collapsed"}

> If you have dependencies on other Pods in the `cocoapods` block, you have to resort to the CocoaPods integration approach.
> Currently, it's impossible to both have dependencies on Pods and on the Kotlin framework in a multimodal SPM project. 
>
{type="warning"}

To remove the CocoaPods plugin from your project:

1. In the directory with `Podfile`, run the following command:

    ```none
   pod deintegrate
   ```

2. Remove the `cocoapods{ }` block from your `build.gradle(.kts)` files.
3. Delete the `.podspec` and `Podfile` files.

Then follow the steps descibed [here](https://www.jetbrains.com/help/kotlin-multiplatform-dev/multiplatform-integrate-in-existing-app.html#connect-the-framework-to-your-ios-project).

## Connect the framework to your project

> The integration into `swift build` is currently not supported.
>
{type="note"}

To connect the Kotlin framework to your Xcode project and use the Kotlin code in a local Swift package:

1. In Xcode, go to **Product** | **Scheme** | **Edit scheme** or gick the schemes icon in the top bar and select **Edit scheme**:

   ![Edit scheme](xcode-edit-schemes.png){width=700}

2. Select **Build** | **Pre-actions** item, then click **+** | **New Run Script Action**:

   ![New run script action](xcode-new-run-script-action.png){width=700}

3. Add the following script and replace `:shared` with your Gradle project name:

   ```bash
   cd "$SRCROOT/.."
   ./gradlew :shared:embedAndSignAppleFrameworkForXcode
   ```
4. Choose your app's target in the **Provide build settings from** section:

   ![Filled run script action](xcode-filled-run-script-action.png){width=700}

5. Use the code from Kotlin in the local Swift package added to your Xcode project:

   ![SPM usage](xcode-spm-usage.png){width=700}

6. Build the project in Xcode. If everything is set up correctly, the project build will be successful.

> If you have a custom build configuration that is different from the default `Debug` or `Release`, on the **Build Settings**
> tab, add the `KOTLIN_FRAMEWORK_BUILD_TYPE` setting under **User-Defined** and set it to `Debug` or `Release`.
>
{type="note"}

## What's next

* [Choose your project configuration](multiplatform-project-configuration.md)
* [Learn how to set up Swift package export](https://kotlinlang.org/docs/native-spm.html)
