[//]: # (title: Adding Swift packages as dependencies to KMP modules)
<primary-label ref="Experimental"/>

<tldr>
   <p>Swift Package Manager (SwiftPM) fulfills the same role as CocoaPods:
   it lets you transparently orchestrate native iOS dependencies of your iOS app.</p>
   <p>Here you can learn how to set up a SwiftPM dependency in your KMP project,
   and how to migrate your KMP setup from CocoaPods to SwiftPM if necessary.</p>
</tldr>

> This feature is [Experimental](https://kotlinlang.org/docs/components-stability.html#stability-levels-explained).
> Please share any problems or feedback you have in the dedicated Kotlin Slack channel: [#kmp-swift-package-manager](https://kotlinlang.slack.com/archives/C09TW68099C)
>
{style="warning"}

Kotlin Gradle plugin with SwiftPM import integration allows you to import Objective-C APIs
from Objective-C and Swift code using SwiftPM dependencies declared for your Apple targets.

For transitive dependencies (projects that depend on those that use SwiftPM import), the
Kotlin Gradle plugin automatically provides the necessary machine code from SwiftPM dependencies.
For example, you don't need to do any additional configuration when running Kotlin/Native tests or linking a framework.

> The [export](multiplatform-spm-export.md) of KMP modules that use SwiftPM import as a Swift package itself is not yet supported and might not work.
> See this [YouTrack issue](https://youtrack.jetbrains.com/issue/KT-84420) for more details and let us know about your use case.
>
{style="note"}

To configure your project:

1. [Set up your development environment](#set-the-kotlin-multiplatform-gradle-plugin-version)
2. [Add and use the SwiftPM dependencies in your KMP module](#add-and-use-swiftpm-dependencies)

## Set the Kotlin Multiplatform Gradle plugin version

To try out the SwiftPM import functionality, make sure that you are using the **%kotlinEapVersion%** version
of the Kotlin Multiplatform Gradle plugin.
Example for a `gradle/libs.versions.toml` file:

```text
[versions]
kotlin = "%kotlinEapVersion%"

[plugins]
kotlin-multiplatform = { id = "org.jetbrains.kotlin.multiplatform", version.ref = "kotlin" }
```

## Add and use SwiftPM dependencies

> For working examples, see our sample projects.
> On the `master` branch, each project is set up using CocoaPods, while the `spm_import` branch uses SwiftPM:
> 
> * [SwiftUI and Firebase example app](https://github.com/Kotlin/kmp-with-cocoapods-firebase-sample/tree/spm_import)
> * [Compose Multiplatform iOS example app](https://github.com/Kotlin/kmp-with-cocoapods-compose-sample/tree/spm_import)
>
{type="tip"}

### Configure the build

Specific SwiftPM dependencies can be added in the `swiftPMDependencies {}` block of the `build.gradle.kts` file,
where your Apple targets are declared.
For example, for Firebase:

```kotlin
kotlin {
    iosArm64()
    iosSimulatorArm64()
    iosX64()

    swiftPMDependencies {
        // Import FirebaseAnalytics into your Kotlin code
        swiftPackage(
            url = url("https://github.com/firebase/firebase-ios-sdk.git"),
            version = from("12.5.0"),
            products = listOf(product("FirebaseAnalytics")),
        )
        // swift-protobuf is a transitive Firebase dependency,
        // so you only need to include it
        // if you want to use a specific version
        swiftPackage(
            url = url("https://github.com/apple/swift-protobuf.git"),
            version = exact("1.32.0"),
            products = listOf(),
        )
    }
}
```

SwiftPM integration is based on importing Clang modules.
By default, the import mechanism automatically discovers Clang modules in specified Swift packages
and makes all available modules accessible to Kotlin code — similar to how API visibility works in Swift and Objective-C.
<!-- TODO link to where it is explained? -->

To disable the default behavior and automatic module discovery, set the `discoverClangModulesImplicitly` to `false`.
When module discovery is disabled, SwiftPM import uses product names as Clang module names.

To import Clang modules whose names differ from product names, use the `importedClangModules` parameter, for example:

```kotlin
kotlin {
    swiftPMDependencies {
        // If 'discoverClangModulesImplicitly' was set to 'true',
        // the 'importedClangModules' parameter below would be ignored
        discoverClangModulesImplicitly = false

        // Imported packages, their products, and Clang modules
        swiftPackage(
            url = url("https://github.com/firebase/firebase-ios-sdk.git"),
            version = from("12.5.0"),
            products = listOf(
                product("FirebaseAnalytics"),
                product("FirebaseFirestore")
            ),
            importedClangModules = listOf(
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

If your project only targets iOS, you don't need to declare platforms explicitly.
But as soon as you add another target (for example, macOS), you need to specify the platform constraint for each dependency.

To make sure that a dependency is applied only for relevant compilations,
specify the correct targets in the `platforms` parameter of a `product` specification:

```kotlin
kotlin {
    iosArm64()
    iosSimulatorArm64()
    iosX64()
    macosArm64()

    swiftPMDependencies {
        swiftPackage(
            url = url("https://github.com/googlemaps/ios-maps-sdk.git"),
            version = exact("10.3.0"),
            products = listOf(
                product(
                    "GoogleMaps", 
                    platforms = setOf(
                        // The `GoogleMaps` package will be visible
                        // only to iOS compilations
                        iOS()
                    )
                )
            )
        ) 
    }
}
```

### Run the SwiftPM integration task

SwiftPM import tooling generates an intermediary package to keep track of the list of the current SwiftPM dependencies.
When you add a SwiftPM dependency to your project for the first time,
you need to link your Xcode project with the generated package.

To do that, run the special Gradle task with the following command in your project's directory:

```shell
XCODEPROJ_PATH='/path/to/project/iosApp/iosApp.xcodeproj' ./gradlew :kotlin-library:integrateLinkagePackage
```

The command will generate the SwiftPM package and perform the necessary changes in the Xcode project.
Make sure to commit the generated package, as well as the updated Xcode project, to your repository.

After the initial integration, the synthetic package will be updated automatically
every time you change the set of SwiftPM dependencies or their versions.

### Use imported APIs

Imported Objective-C APIs are contained in namespaces that start with the `swiftPMImport` prefix
and end with Gradle names of the project and its group.

For example, the Kotlin build script specifies the group name as follows:

```kotlin
// subproject/build.gradle.kts
group = "groupName"
```

Here, `groupName` is the Gradle group name of the project, and `subproject` is the project name.
Now you can import Firebase APIs in the `iosMain` source set of that module, for example:

```kotlin
// subproject/src/iosMain/kotlin/useFirebaseAnalytics.kt
import swiftPMImport.groupName.subproject.FIRAnalytics
import swiftPMImport.groupName.subproject.FIRApp
```

## Generated `Package.resolved` files

To make builds that depend on Swift packages more stable, the SwiftPM import tooling introduces a locking mechanism with
`Package.resolved` files. They are generated for each subproject during the initial package resolution.

By default, these files are merged into a single `Package.resolved` file placed inside a synthetic package
in the `.swiftpm-locks/default/swiftImport` directory.
This shared lock file is then used to build the project, making sure that all subprojects use the same versions of Swift packages.
You can customize lock-file merging behavior by [grouping subprojects or excluding them from synchronization](#customize-aggregation-of-swift-package-versions).

You should commit the lock file to your repository to ensure that all builds use the same dependencies.
To simplify file management, you can commit the entire `.swiftpm-locks` directory to your repository.
Although only the `Package.resolved` files are crucial for dependency synchronization,
having the entire directory can speed up resolution during the first build. 

The lock files are updated automatically when you change the set or versions of SwiftPM dependencies in your build scripts.
You can also [force an update of a lock file manually](#force-an-update-of-the-lock-file).

### Customize aggregation of Swift package versions

Instead of using the `default` group for all subprojects,
you can define custom groups to generate a separate `Package.resolved` lock file for each group.

The merge behavior is controlled by the `packageResolvedSynchronization` option in the `swiftDependencies {}` block:

```kotlin
kotlin {
    swiftDependencies {
        // When no value is set for `packageResolvedSynchronization`, 
        // the subproject is assigned a default group identifier
        // as if it were set like this: 
        // packageResolvedSynchronization = identifier("default")
    }
}
```

To customize the merge behavior, assign a non-default group identifier to each subproject.
In the following example, subprojects `one` and `two` use the same `custom` set of package versions,
while the subproject `three` uses the default set:

<tabs>
<tab title="Subproject `one`">

```kotlin
// one/build.gradle.kts

kotlin {
    swiftDependencies {
        packageResolvedSynchronization = identifier("custom"),
        ...
    }
}
```
</tab>

<tab title="Subproject `two`">

```kotlin
// two/build.gradle.kts

kotlin {
    swiftDependencies {
        packageResolvedSynchronization = identifier("custom"),
        ...
    }
}
```

</tab>

<tab title="Subproject `three`">

```kotlin
// three/build.gradle.kts

kotlin {
    swiftDependencies {
        // The default identifier is used, as if the following is set:
        // packageResolvedSynchronization = identifier("default")
        ...
    }
}
```

</tab>

</tabs>

If you'd like to disable the synchronization mechanism for a subproject altogether,
use a `noSynchronization()` call instead of `identifier()`:

```kotlin
kotlin {
    swiftDependencies { 
        // The Package.resolved file for this subproject
        // won't be merged with any other
        packageResolvedSynchronization = noSynchronization()
    }
}
```

Subprojects with disabled synchronization will have their own `Package.resolved` lock file,
placed in the subproject directory next to the `build.gradle.kts` file.

Just like with the default synchronization, all `Package.resolved` files for customized subprojects
should be committed to your repository.

### Force an update of the lock file

If you want to force an update of a lock file manually:

1. Delete the `build` directory for every subproject whose lock files should be updated.
2. Remove the existing `Package.resolved` file:
   * For subprojects without a specific synchronization configuration, delete the `.swiftpm-locks/default/` directory.
   * For subprojects with a [custom synchronization group](#customize-aggregation-of-swift-package-versions), find and delete the `.swiftpm-locks/<group-name>/` directory.
   * For subprojects with `noSynchronization()` set, find and delete the `Package.resolved` file in the subproject directory.
3. Run the dependency resolution task again: `./gradlew :yourModuleName:fetchSyntheticImportProjectPackages`.

## Additional import options

### Importing local Swift packages

The SwiftPM import mechanism also allows importing Swift packages from the local file system.

Let's consider a Swift package with the following manifest, located in the `/path/to/ExamplePackage` directory:

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

To import it in your Kotlin build script, use the `localSwiftPackage` API:

```kotlin
// <projectDir>/shared/build.gradle.kts
kotlin {
    swiftPMDependencies {
        localSwiftPackage(
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
specify it in the `*MinimumDeploymentTarget` parameter. For example, for iOS:

```kotlin
kotlin {
    swiftPMDependencies {
        iosMinimumDeploymentTarget.set("16.0")
    }
}
```

### Location and version of Swift packages

Similar to `Package.swift` manifest files, you can specify the location and version of your Swift package in the `swiftPackage()` call.
Both have a couple of mutually exclusive options. 

To set the location, you can use a URL or a [SwiftPM registry ID](https://docs.swift.org/swiftpm/documentation/packagemanagerdocs/usingswiftpackageregistry):

```kotlin
swiftPackage(
    // Option 1, URL string
    // Points to the Git repository of the package
    url = url("https://github.com/firebase/firebase-ios-sdk.git")

    // Option 2, Swift Package Registry ID
    // See Apple documentation on using a package registry linked above  
    repository = id("...")
)
```

To specify the version, use the following Gradle- and Git-style version specifications:

```kotlin
swiftPackage(
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

## Known limitations with dynamic Kotlin/Native frameworks

Currently, SwiftPM import integration doesn't support all edge cases that might arise when producing a dynamic
Kotlin/Native framework. You might encounter issues during the build in Xcode or see warnings at runtime, for example:

* `Undefined symbols for architecture ...: "...", referenced from: ld: symbol(s) not found ...`
* `dyld: Symbol not found: ...`
* `objc[...]: Class _Foo is implemented in both /path/to/Shared and /path/to/Bar. This may cause spurious casting failures and mysterious crashes. One of the duplicates must be removed or renamed.`

A general fix for these issues is to change the linkage mode of your framework by setting the `isStatic` property to `true`:

```kotlin
// shared/build.gradle.kts
kotlin {
    listOf(
        iosArm64(),
        iosSimulatorArm64()
    ).forEach { iosTarget ->
        iosTarget.binaries.framework {
            baseName = "Shared"

            // Set this property to "true"
            isStatic = true
        }
    }
}
```

If you encountered any of these issues, need to keep `isStatic=false`, or if changing this property didn't help resolve
build failures, let us know in our Slack channel. Get an [invite](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up)
and join [#kmp-swift-package-manager](https://kotlinlang.slack.com/archives/C09TW68099C).

## What's next?

Learn more about [switching from CocoaPods to SwiftPM dependencies in your KMP project](multiplatform-cocoapods-spm-migration.md).
