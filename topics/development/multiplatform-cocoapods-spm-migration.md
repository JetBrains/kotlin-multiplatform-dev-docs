[//]: # (title: Switch Kotlin Multiplatform project from CocoaPods to SPM dependencies)
<primary-label ref="experimental"/>

<tldr>

* When using the CocoaPods Gradle plugin, you need to reconfigure your Xcode project before switching to SwiftPM.
* Check out these sample projects which use CocoaPods in the `main` branch and SwiftPM in the `spm-import` branch:
  * [Firebase sample](https://github.com/Kotlin/kmp-with-cocoapods-multitarget-xcode-sample)
  * [Compose Multiplatform sample](https://github.com/Kotlin/kmp-with-cocoapods-compose-sample/)

</tldr>

If you have a KMP module with CocoaPods dependencies and you want to switch to Swift packages using [SwiftPM import](multiplatform-spm-import.md),
follow the steps described below.

> You can hand over the CocoaPods to SwiftPM migration to your AI agent of choice using the [prepared skill](https://github.com/Kotlin/kmp-cocoapods-to-spm-migration/blob/master/SKILL.md).
> Keep in mind that results of AI processing are not entirely predictable.
>
{style="note"}

General migration sequence:

1. Update your build script with SwiftPM dependencies and corresponding configuration.
2. Reconfigure your Xcode project to use [direct integration](multiplatform-direct-integration.md) with the help
   of SwiftPM import tooling.
3. Disable the CocoaPods integration entirely or partially, depending on your project structure.

## Update your build script

Before doing anything else, update the version of your Kotlin Multiplatform Gradle plugin to the version **2.4.0-Beta1 or newer**.
<!--TODO check version-->

Without disabling the CocoaPods plugin or removing CocoaPods dependencies,
add the necessary SwiftPM dependencies to your `build.gradle.kts` file as per the [SwiftPM import page](multiplatform-spm-import.md).

For example, if you are using the `FirebaseAnalytics` pod:

1. Add the `FirebaseAnalytics` Swift package to the `swiftPMDependencies {}` block:

    ```kotlin
    // projectDir/sharedLogic/build.gradle.kts
    kotlin {
      swiftPMDependencies {
        `package`(
          url = url("https://github.com/firebase/firebase-ios-sdk.git"),
          version = from("12.5.0"),
          products = listOf(product("FirebaseAnalytics")),
        )
      } 
    
      cocoapods {
        // ...
    
        pod("FirebaseAnalytics") {
          version = "12.5.0"
          // ...
        }
      }
    }
    ```
2. Run the **Sync Project with Gradle Files** action to import the APIs from the Swift package.
3. Update your code to use the APIs imported from the Swift package.
   If the pod and the corresponding Swift package offer exactly the same APIs, you should only need to update the Kotlin import directives,
   for example:

    <compare type="top-bottom">
        <code-block lang="kotlin">
        import cocoapods.FirebaseAnalytics.FIRAnalytics
        </code-block>
        <code-block lang="kotlin">
        import swiftPMImport.org.example.package.FIRAnalytics
        </code-block>
    </compare>

4. If you are using the `cocoapods.framework {}` block in your build script,
   move that configuration to the `binaries.framework {}` block, for example:

    <compare type="left-right">
    <code-block lang="kotlin">
    kotlin {
      iosArm64()
      iosSimulatorArm64()
      iosX64()
    
      cocoapods {
        framework {
          baseName = "Shared"
          isStatic = true
        }
      }
    }
    </code-block>
    <code-block lang="kotlin">
    kotlin {
      listOf(
        iosArm64(),
        iosSimulatorArm64(),
        iosX64(),
      ).forEach { iosTarget ->
        iosTarget.binaries.framework {
          baseName = "Shared"
          isStatic = true
        }
      } 
    }
    </code-block>
    </compare>

## Reconfigure your Xcode project

If you are using the CocoaPods Gradle plugin (`kotlin(“native.cocoapods”)`),
you need to reconfigure your Xcode project to [direct integration](multiplatform-direct-integration.md) before switching to SPM.
The SwiftPM import tooling can generate the shell command to make the necessary changes to your .xcodeproj file.

1. Open the project in Xcode (in IntelliJ IDEA, select **File** | **Open Project in Xcode**).
2. Build the project in Xcode (**Product** | **Build**).
   The build will fail, but the build error will contain the necessary command.
   To see the build errors in Xcode, select **View** | **Navigators** | **Report**, then choose **Errors Only** in the filter at the top.
   The command looks like this and includes the correct paths to your project:

    ```text
    XCODEPROJ_PATH='/path/to/project/iosApp/iosApp.xcodeproj' GRADLE_PROJECT_PATH=':kotlin-library' '/path/to/project/gradlew' -p '/path/to/project' ':kotlin-library:integrateEmbedAndSign' ':kotlin-library:integrateLinkagePackage'
    ```
   
    > You can generate the same command without opening Xcode, by building the project from the terminal.
    > Run the following command in the `/path/to/project/iosApp` directory:
    > 
    > ```shell
    > xcodebuild -scheme "$(echo -n *.xcworkspace | python3 -c 'import sys, json; from subprocess import check_output; print(list(set(json.loads(check_output(["xcodebuild", "-workspace", sys.stdin.readline(), "-list", "-json"]))["workspace"]["schemes"]) - set(json.loads(check_output(["xcodebuild", "-project", "Pods/Pods.xcodeproj", "-list", "-json"]))["project"]["schemes"]))[0])')" -workspace *.xcworkspace -destination 'generic/platform=iOS Simulator' ARCHS=arm64 | grep -A5 'What went wrong'
    > ```
    {style="note"}
   
    The `grep` call in the end finds the specific error message with the command which you need to run.

3. Run the generated command in a terminal in the `/path/to/project/iosApp` directory.
   It will modify the .xcodeproj file of your `iosApp` project to trigger the `embedAndSignAppleFrameworkForXcode` task
   during the build, which inserts a Kotlin Multiplatform compilation phase into your iOS build.

4. In IntelliJ IDEA, select **Tools** | **Swift Package Manager** | **Resolve Dependencies** to resolve the SwiftPM dependencies
   declared in your `build.gradle.kts` file.

Now the iOS app is switched over to SwiftPM dependencies.
You can disable the CocoaPods plugin and deintegrate the pod.

## Remove the CocoaPods KMP integration

If you fully replace your CocoaPods dependencies with Swift packages, you can deintegrate the pod by running
the following command in the `/path/to/project/iosApp` directory:

```shell
pod deintegrate
```

If you want to continue using CocoaPods for dependencies that don't intersect with SwiftPM dependencies,
edit your `Podfile` to remove only the line that mentions the KMP module and run `pod install`.
For example:

```shell
target 'iosApp' do
    # Here 'sharedLogic' is the name of a shared code module.
    # Remove this line and run 'pod install' again.
    pod 'sharedLogic', :path => '../sharedLogic'
    ...
end
```

Finally, remove mentions of CocoaPods from your Gradle build configuration:

1. Remove the entire `cocoapods {}` block from the `build.gradle.kts` file in your shared code module,
   since all dependencies are now managed by SwiftPM import tooling.

2. If your project does not rely on CocoaPods anymore, remove references to the CocoaPods Gradle plugin
   from the `plugins {}` block in both the root `build.gradle.kts` file and `build.gradle.kts` in the shared module.
