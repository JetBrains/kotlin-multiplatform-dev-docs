[//]: # (title: Switch Kotlin Multiplatform project from CocoaPods to SPM dependencies)

<tldr>

* When using the CocoaPods Gradle plugin, you need to reconfigure your Xcode project before switching to SwiftPM.
* Check out these sample projects which use CocoaPods in the `main` branch and SwiftPM in the `spm-import` branch:
  * [Firebase sample](https://github.com/Kotlin/kmp-with-cocoapods-multitarget-xcode-sample)
  * [Compose Multiplatform sample](https://github.com/Kotlin/kmp-with-cocoapods-compose-sample/)

</tldr>

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

1. Build the Xcode project (**Product** | **Build**).
   The build should fail, but in the build output you will see the command to run
   to reconfigure the project which should look like this:

    ```text
    XCODEPROJ_PATH='/path/to/project/iosApp/iosApp.xcodeproj' GRADLE_PROJECT_PATH=':kotlin-library' '/path/to/project/gradlew' -p '/path/to/project' ':kotlin-library:integrateEmbedAndSign' ':kotlin-library:integrateLinkagePackage'
    ```

2. Run the provided command in a terminal.
   It will modify the .xcodeproj file of your `iosApp` to trigger the `embedAndSignAppleFrameworkForXcode` task
   during the build, which inserts a Kotlin Multiplatform phase into your iOS build.

3. Select **Tools** | **Swift Package Manager** | **Resolve Dependencies** to resolve the SwiftPM dependencies
   declared in your `build.gradle.kts` file.

> Whenever you change your SwiftPM configuration, you need to update the generated code using the import tooling.
> You can do that by pressing the 

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

Finally, remove mentions of CocoaPods from your build configuration:

1. Remove the entire `cocoapods {}` block from the `build.gradle.kts` file in your shared code module,
   since all dependencies are now managed by SwiftPM import tooling.

2. If your project does not rely on CocoaPods anymore, remove references to the CocoaPods Gradle plugin
   from the `plugins {}` block in both the root `build.gradle.kts` file and `build.gradle.kts` in the shared module.
