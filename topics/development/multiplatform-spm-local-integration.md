[//]: # (title: Using Kotlin from local Swift packages)

In this tutorial, you'll learn how to add a dependency on a Kotlin framework in a local package in the Swift package manager.
This is useful if you want to simultaneously connect your Kotlin Multiplatform module to several SPM modules
in your Xcode project.

> If you aren't familiar with Kotlin Multiplatform, learn how to [set up the environment](multiplatform-setup.md)
> and [create a cross-platform application from scratch](multiplatform-create-first-app.md) first.
>
{type="tip"}

## Set up the project

The feature is available starting with Kotlin 2.0.0.

If you still haven't upgraded to Kotlin 2.0.0, you can use a special Kotlin version, `1.9.24-spm` from the
`https://packages.jetbrains.team/maven/p/mpp/dev` Maven repository. You can add the repository, for instance,
in your `settings.gradle.kts`:

```kotlin
pluginManagement {
    repositories {
        maven("https://packages.jetbrains.team/maven/p/mpp/dev")
        //...
    }
}

dependencyResolutionManagement {
    repositories {
        maven("https://packages.jetbrains.team/maven/p/mpp/dev")
        //...
    }
}
```

The tutorial assumes that your project is using [direct integration](multiplatform-project-configuration.md#connect-a-kotlin-multiplatform-module-to-an-ios-app)
approach with the `embedAndSignAppleFrameworkForXcode` task. If you're connecting a Kotlin framework through CocoaPods
plugin, migrate first.

### Migrate from CocoaPods plugin to direct integration {initial-collapse-state="collapsed"}

> If you have dependencies on other Pods in the `cocoapods {}` block, you have to resort to the CocoaPods integration approach.
> Currently, it's impossible to both have dependencies on Pods and on the Kotlin framework in a multimodal SPM project. 
>
{type="warning"}

To migrate from the CocoaPods plugin:

1. In the directory with `Podfile`, run the following command:

    ```none
   pod deintegrate
   ```

2. Remove the `cocoapods {}` block from your `build.gradle(.kts)` files.
3. Delete the `.podspec` and `Podfile` files.
4. To set up direct integration, follow the steps described in [this tutorial](multiplatform-integrate-in-existing-app.md#connect-the-framework-to-your-ios-project).

## Connect the framework to your project

> The integration into `swift build` is currently not supported.
>
{type="note"}

To connect the Kotlin framework to your Xcode project and use the Kotlin code in a local Swift package:

1. In Xcode, go to **Product** | **Scheme** | **Edit scheme** or click the schemes icon in the top bar and select **Edit scheme**:

   ![Edit scheme](xcode-edit-schemes.png){width=700}

2. Select the **Build** | **Pre-actions** item, then click **+** | **New Run Script Action**:

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
