[//]: # (title: Use Kotlin from local Swift packages)

### Prerequisites

> If you aren't familiar with Kotlin Multiplatform, learn how to [set up environment and create a cross-platform application from scratch](multiplatform-setup.md)
> first.
>
{type="tip"}

This tutorial uses a special Kotlin version — `2.0.0-ux1` from `https://packages.jetbrains.team/maven/p/mpp/dev` Maven repository. 

You can add the repository, for instance, in your `settings.gradle.kts`:
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

If you're migrating from other integration approach, please remove it first:


###### CocoaPods plugin {initial-collapse-state="collapsed"}

> If you have dependencies to other pods in `cocoapods` block, you have to stay on CocoaPods integration approach
>
{type="warning"}

1. Call `pod deintegrate` in directory with `Podfile`
2. Remove `cocoapods` block from `build.gradle(.kts)` files
3. Remove `.podspec`, `Podfile` files

###### Direct integration through embedAndSignAppleFrameworkForXcode {initial-collapse-state="collapsed"}

1. In Xcode, open the iOS project settings by double-clicking the project name.

2. Remove `Run Script` build phase from Xcode project
![Add the script](xcode-add-run-phase-2.png){width=700}

3. On the **Build Settings** tab, remove the following from **Framework Search Path** property:

   ```text
   $(SRCROOT)/../shared/build/xcode-frameworks/$(CONFIGURATION)/$(SDK_NAME)
   ```

   ![Framework search path](xcode-add-framework-search-path.png){width=700}

4. On the **Build Settings** tab, remove the following from **Other Linker flags** property:

   ```text
   $(inherited) -framework shared
   ```

   ![Linker flag](xcode-add-flag.png){width=700}

### Connect the framework to your Xcode project

> Note that integrating into `swift build` is currently not supported
>
{type="note"}

2. Make sure that frameworks are declared for all targets corresponding the Xcode project platforms, for instance: 
```kotlin
    listOf(
        iosX64(),
        iosArm64(),
        iosSimulatorArm64(),
    ).forEach {
        it.binaries.framework {
            baseName = "shared"
        }
    }
```

You can learn more about declaring binaries [here](https://kotlinlang.org/docs/multiplatform-build-native-binaries.html).

3. In Xcode go to `Product -> Scheme -> Edit scheme...` or click the schemes icon in the top bar and select `Edit scheme..`: 

   ![Edit scheme](xcode-edit-schemes.png){width=700}

4. Select `Build -> Pre-actions` item, then click *+* and *New Run Script Action*:

   ![New run script action](xcode-new-run-script-action.png){width=700}

5. Add the following script (replace `:shared` with your Gradle project name):
```bash
cd "$SRCROOT/.."
./gradlew :shared:embedAndSignAppleFrameworkForXcode
```
and choose your app's target in the *Provide build settings from* item:

   ![Filled run script action](xcode-filled-run-script-action.png){width=700}
   
6. Use code from Kotlin in local Swift (SPM) package added into Xcode project:
   
   ![SPM usage](xcode-spm-usage.png){width=700}
 
7. Build the project in Xcode. If everything is set up correctly, the project will successfully build.


> If you have a custom build configuration different from the default `Debug` or `Release`, on the **Build Settings**
> tab, add the `KOTLIN_FRAMEWORK_BUILD_TYPE` setting under **User-Defined** and set it to `Debug` or `Release`.
>
{type="note"}

