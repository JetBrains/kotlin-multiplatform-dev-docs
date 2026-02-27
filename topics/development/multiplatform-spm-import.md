[//]: # (title: Adding Swift packages as dependencies to KMP modules)
<primary-label ref="alpha"/>

<tldr>
   Swift Package Manager fulfills the same role as CocoaPods:
   it lets you transparently orchestrate native iOS dependencies of your iOS app.
   Here you can learn how to set up an SPM dependency in your KMP project,
   and how to migrate your KMP setup from CocoaPods to SPM if necessary.
</tldr>

Kotlin Gradle plugin with Swift Package Manager import integration allows you to import Objective-C APIs
from Objective-C and Swift code of SPM dependencies declared for your Apple targets.

For transitive dependencies (projects that depend on those that use SwiftPM import),
Kotlin Gradle plugin automatically provisions the necessary machine code from SwiftPM dependencies.
So, for example, you don't need to do any additional configuration when running Kotlin/Native tests or linking a framework.

You cannot, however, [export](multiplatform-spm-export.md) the KMP module that uses SwiftPM import as a Swift package itself.

To configure your project:

1. [Set up your development environment](#environment-setup)
2. [Add the SwiftPM dependencies to your KMP module](#add-and-call-spm-dependencies)
3. [Use the imported APIs in your Kotlin code](#use-the-imported-apis)

## Environment setup

Since SPM import functionality is still in the Alpha stage, you need to request an EAP Kotlin version to try it out.

To do that:

1. In your `settings.gradle.kts` file, add the development package repository for dependencies and plugins:

    ```kotlin
    dependencyResolutionManagement {
        repositories {
            maven("https://packages.jetbrains.team/maven/p/kt/dev")
            mavenCentral()
        }
    }
    
    pluginManagement {
        repositories {
            maven("https://packages.jetbrains.team/maven/p/kt/dev")
            mavenCentral()
            gradlePluginPortal()
        }
    }
    ```
   
2. In your version catalog, apply the EAP version of the Kotlin Multiplatform Gradle plugin, for example:
   TODO check the final release version

    ```text
    kotlin = "2.4.0-titan-214"
   
    [plugins]
    kotlin-multiplatform = "2.4.0-titan-214"
    ```

3. Sync the Gradle files and try adding a `kotlin.swiftPMDependencies {}` block to the `build.gradle.kts` file
   of your KMP module.
   If the `swiftPMDependencies` name cannot be resolved, add the following block to the root `build.gradle.kts` file
   to add the necessary symbols to the classpath directly:

    ```kotlin
    buildscript {
        dependencies.constraints {
            "classpath"("org.jetbrains.kotlin:kotlin-gradle-plugin:2.4.0-titan-214!!")
        }
    }
    ```
   
### Set up the KMP IDE plugin

If you are using the [Kotlin Multiplatform IDE plugin]() recommended for any KMP project,
explicitly specify the path to the iOS project that is built from the KMP module.

In the `build.gradle.kts` file where you call the `iosTarget.binaries.framework` API,
add the API call that sets the path:

```kotlin
kotlin {
    // Example of iOS targets configuration
    listOf(
        iosArm64(),
        iosSimulatorArm64(),
        iosX64(),
    ).forEach { iosTarget ->
            iosTarget.binaries.framework { 
                baseName = "Shared"
                isStatic = false
            } 
    }
      
    swiftPMDependencies { 
        // Specify the path to the .xcodeproj file that uses
        // the ':embedAndSignAppleFrameworkForXcode' integration
        xcodeProjectPathForKmpIJPlugin.set(
            layout.projectDirectory.file("../iosApp/iosApp.xcodeproj")
        )
    }
}
```

## Add and call SPM dependencies

> For working examples, see our sample projects.
> In the `master` branch each project is set up using CocoaPods, while the `spm_import` branch uses SPM:
* [SwiftUI and Firebase example app](https://github.com/Kotlin/kmp-with-cocoapods-firebase-sample/tree/spm_import)
* [Compose Multiplatform iOS example app](https://github.com/Kotlin/kmp-with-cocoapods-compose-sample/tree/spm_import)

### Build configuration

Specific SPM dependencies can be added in the `swiftPMDependencies` block of the `build.gradle.kts` file,
where your Apple targets are declared.
For example, for Firebase:

```kotlin
kotlin {
    iosArm64()
    iosSimulatorArm64()
    iosX64()

    swiftPMDependencies {
        // Import FirebaseAnalytics into your Kotlin code
        `package`(
            url = url("https://github.com/firebase/firebase-ios-sdk.git"),
            version = from("12.5.0"),
            products = listOf(product("FirebaseAnalytics")),
        )
        // swift-protobuf is a transitive Firebase dependency,
        // so you only need to include it
        // if you want to use a specific version
        `package`(
            url = url("https://github.com/apple/swift-protobuf.git"),
            version = exact("1.32.0"),
            products = listOf(),
        )
    }
}
```


SPM integration is based on importing Clang modules.
By default, the import mechanism automatically discovers Clang modules available in specified Swift packages
and makes all discovered modules accessible to Kotlin code — similar to how API visibility works in Swift and Objective-C. TODO link to where it is explained?

To disable the default behavior and automatic module discovery, set the `discoverModulesImplicitly` to `false`.
When module discovery is disabled, SPM import uses product names as Clang module names.
To import Clang modules whose names differ from product names, use the `importedModules` parameter.

Example of 

```kotlin
kotlin {
    swiftPMDependencies {
        // If 'discoverModulesImplicitly' was set to 'true', 
        // the 'importedModules' parameter below would be ignored
        discoverModulesImplicitly = false

        // Imported packages, their products and Clang modules
        `package`(
            url = url("https://github.com/firebase/firebase-ios-sdk.git"),
            version = from("12.5.0"),
            products = listOf(
                product("FirebaseAnalytics"),
                product("FirebaseFirestore")
            ),
            importedModules = listOf(
                "FirebaseAnalytics", 
                // Objective-C APIs of FirebaseFirestore are located
                // in the 'FirebaseFirestoreInternal' Clang module
                "FirebaseFirestoreInternal"
            ),
        )
    }
}
```

### Set platform constraints

Some SwiftPM dependencies may not compile or provide valid APIs for all targets in your build script.
For example, the Google Maps SDK currently only supports iOS targets.
So, while your project only targets iOS, you don't have to declare platforms explicitly.
But as soon as you add another target, for example, macOS, you need to specify the platform constraint for each dependency.

To make sure that a dependency is applied only for relevant compilations,
specify the correct targets in the `platforms` parameter of a `product` specification:

```kotlin
kotlin {
    iosArm64()
    iosSimulatorArm64()
    iosX64()
    macosArm64()

    swiftPMDependencies {
        `package`(
            url = url("https://github.com/googlemaps/ios-maps-sdk.git"),
            version = exact("10.3.0"),
            products = listOf(
                product(
                    "GoogleMaps", 
                    platforms = setOf(
                        // The 'GoogleMaps' package will be visible
                        // only to iOS compilations
                        iOS()
                    )
                )
            )
        ) 
    }
}

```

### Use the imported APIs

Imported Objective-C APIs are contained in namespaces which start with the `swiftPMImport` prefix
and end with Gradle names of the project and its group.

For example, the Kotlin build script specifies the group name as follows:

```kotlin
// subproject/build.gradle.kts
group = "groupName"
```

Here `groupName` is the Gradle group name of the project, and `subproject` is the name of the project.
Now we can import Firebase APIs in the `iosMain` source set of that module:

```kotlin
// subproject/src/iosMain/kotlin/useFirebaseAnalytics.kt
import swiftPMImport.groupName.subproject.FIRAnalytics
import swiftPMImport.groupName.subproject.FIRApp
```

## Additional import options

### Importing local Swift packages

TODO do we need two examples like in the draft?

The SwiftPM import mechanism also allows importing Swift packages from the local file system.

For example, you have a Swift package with the following manifest, located in the `/path/to/ExamplePackage` directory:

```swift
// /path/to/ExamplePackage/Package.swift
let package = Package(
  name: "ExamplePackage",
  platforms: [.iOS("15.0")],
  products: [
    .library(name: "ExamplePackage", targets: ["ExamplePackage"]),
  ],
  dependencies: [
    .package(url: "https://github.com/grpc/grpc-swift.git", exact: "1.27.0",),
  ],
  targets: [
    // This target can be implemented in Swift with @objc API or in Objective-C
    .target(name: "ExamplePackage", dependencies: [.product(name: "GRPC", package: "grpc-swift")]),
  ]
)
```
{collapsible="true" collapsed-title-line-number="3"}

To import it in your Kotlin build script, use the `localPackage` API:

```kotlin
// <projectDir>/shared/build.gradle.kts
kotlin {
    swiftPMDependencies {
        localPackage(
            directory = project.layout.projectDirectory.dir("/path/to/ExamplePackage/"),
            products = listOf("ExamplePackage")
        )
    }
}
```

Sync the Gradle files to perform SwiftPM import, then use the imported APIs in your Kotlin code:

```kotlin
// /path/to/shared/src/appleMain/kotlin/useExamplePackage.kt

@OptIn(kotlinx.cinterop.ExperimentalForeignApi::class)
fun useExamplePackage() {
    // If the Swift package is successfully imported,
    // the IDE suggests the correct import for the class
    HelloFromExamplePackage().hello()
}
```

### Specific deployment versions

If your dependencies require a higher [deployment version](https://developer.apple.com/documentation/packagedescription/supportedplatform),
specify it in a `*DeploymentVersion` parameter, for example, for iOS:

```kotlin
kotlin {
    swiftPMDependencies {
        iosDeploymentVersion.set("16.0")
    }
}
```

### Location and version of a Swift package

Similar to a `Package.swift` manifest, you can specify the location and version of a Swift package in a `package` call.
Both have a couple of mutually exclusive options to choose from. 

For the location, you can use a URL or an [SPM registry ID](https://docs.swift.org/swiftpm/documentation/packagemanagerdocs/usingswiftpackageregistry):

```kotlin
`package`(
    // Option 1, URL string
    // Points to the Git repository of the package
    url = url("https://github.com/firebase/firebase-ios-sdk.git")

    // Option 2, Swift Package Registry ID
    // See Apple documentation on using a package registry linked above  
    repository = id("...")
)
```

For the version, you can use the following Gradle- and Git-style version specifications:

```kotlin
`package`(
    // Similar to the Gradle 'require' version constraint,
    // starting with the specified version
    version = from("1.0")
            
    // Similar to the Gradle 'strict' version constraint,
    // exactly matching the specified version
    version = exact("2.0")
  
    // Git-specific version specification,
    // matching the specified branch or revision
    version = branch("master")
    // Or
    version = revision("e74b07278b926c9ec6f9643455ea00d1ce04a021")
)
```