[//]: # (title: Umbrella module placement for iOS apps)

When Kotlin Multiplatform code is developed by several teams, it's usually split into libraries that are stored in
different repositories and released on their own schedules.
An iOS app should consume these libraries through a single [umbrella framework](multiplatform-project-module-configurations.md#several-shared-modules),
so the location of the umbrella module that produces this framework determines how changes reach the iOS app.

This page compares the options for placing the umbrella module and describes the approach we recommend for multi-team
setups: keeping an umbrella module in each iOS app repository.
This applies whether you have one iOS app or several.

## The problem with a separately published umbrella framework

An iOS app should link only one framework produced by Kotlin/Native.
Each Kotlin framework includes the Kotlin runtime and all of its dependencies, so linking several of them duplicates
code and breaks state shared through common dependencies.
For more details, see [Why do you need an umbrella framework?](multiplatform-project-module-configurations.md#why-do-you-need-an-umbrella-framework)

A common way to provide this single framework is to build it in the repository with the shared code, or in a separate
repository, and publish it as a binary XCFramework through SwiftPM or CocoaPods.
In this setup, the umbrella repository effectively acts as a lock file for the versions of all libraries in the iOS app.
With several independent libraries, this causes the following problems:

* **Every change requires several steps**. To deliver an update of a single library, you need to update the umbrella
  module, build and publish a new umbrella framework, and then update the framework version in the iOS app.
* **Updates get batched**. To reduce this overhead, teams tend to release the umbrella framework less often, with changes
  from several libraries at once.
* **iOS changes get larger**. When a new umbrella version contains changes from several libraries, iOS developers have to
  adapt to all of them in one change, even if they work on an unrelated feature.
  A problem in one library blocks the update of all others.
* **Distribution costs**. CI has to build release versions of the umbrella framework for all Apple targets, and you need
  to host, version, and clean up the XCFramework artifacts.

<!-- TODO: diagram of the separate umbrella repository workflow, see the KEDOC-337 report -->

## Options for placing the umbrella module

The following table summarizes where the umbrella module can be located in different
[repository configurations](multiplatform-project-repository-configurations.md):

<table>
  <tr>
     <th>Umbrella module location</th>
     <th>How the iOS app gets the framework</th>
     <th>Works best for</th>
     <th>Drawbacks</th>
  </tr>
  <tr>
     <td>Monorepo with the apps and the shared code</td>
     <td>Built from source as part of the iOS build</td>
     <td>Teams that work on the whole codebase together</td>
     <td>Often impractical for existing apps that are already stored in different repositories</td>
  </tr>
  <tr>
     <td>The repository with the shared code</td>
     <td>Published as a prebuilt XCFramework</td>
     <td>One team that owns all of the shared code, for one iOS app or for several apps that use the same libraries and update them together</td>
     <td>All shared modules are released together; serving different subsets to different iOS apps requires a separate umbrella module and framework release for each app</td>
  </tr>
  <tr>
     <td>A separate umbrella repository for each iOS app</td>
     <td>Published as a prebuilt XCFramework</td>
     <td>iOS teams that can't run Gradle in their builds</td>
     <td>Two changes per library update, batched updates, XCFramework distribution, as described above</td>
  </tr>
  <tr>
     <td>The iOS app repository</td>
     <td>Built locally from published Kotlin libraries as part of the iOS build</td>
     <td>Several teams that publish independent libraries for one or more iOS apps</td>
     <td>The iOS build runs Gradle and links the framework, which adds to iOS build times</td>
  </tr>
</table>

## Recommended: umbrella module in the iOS app repository

In this setup, the shared code is split into libraries that are published as regular Kotlin Multiplatform artifacts,
for example, to an internal Maven repository.
Each iOS app repository contains a small Gradle project with a single umbrella module.
The umbrella module has no feature code of its own: it declares the libraries and the versions that the iOS app uses.

When the iOS app is built, Xcode runs a Gradle task that resolves the library versions, builds the umbrella framework
for the current build configuration, and embeds it into the app.
This is the same mechanism as [direct integration](multiplatform-direct-integration.md), applied to a Gradle project
that contains only the umbrella module.

<!-- TODO: diagram of the umbrella module in the iOS app repository, see the KEDOC-337 report -->

This setup has the following benefits:

* **Independent library updates**. Updating a library means changing one version in the iOS repository.
  Each library can be updated in a separate change, several updates can be prepared in parallel,
  and a problematic update can be reverted without affecting the others.
* **Smaller iOS changes**. Each update brings changes from one library, so iOS developers adapt to one set of API changes
  at a time.
* **No XCFramework distribution**. You don't need to build, publish, host, or version an umbrella XCFramework.
  The library repositories publish klib artifacts, which, in most cases, can be built on any host.
  For details, see [Host requirements](multiplatform-publish-lib-setup.md#host-requirements).
* **Faster delivery of library updates**. A new library version reaches the iOS app as soon as it's published, without
  waiting for a release build of the whole umbrella framework. During development, the framework is linked in the debug
  configuration, which is faster than a release build.
* **Different subsets for different apps**. Each iOS app repository has its own umbrella module, so it includes only the
  libraries the app needs. See [Multiple iOS apps with different sets of libraries](#multiple-ios-apps-with-different-sets-of-libraries).

Keep the following in mind:

* The iOS app build runs Gradle, so developer machines and CI agents that build the iOS app need to be able to run
  the Gradle wrapper committed to the iOS repository.
  iOS developers don't need to edit Kotlin code or work in IntelliJ IDEA or Android Studio.
* Each iOS build includes building the framework. The first build downloads the Kotlin/Native toolchain and the
  libraries, and release builds link an optimized framework, which takes noticeably longer than a debug build.
  Account for this in iOS CI, which now builds the release framework as part of the app build.
* Exporting a dependency to the framework disables dead code elimination for it.
  Export only the libraries whose API is used from Swift and add other libraries as regular dependencies.
  See [Export dependencies to binaries](multiplatform-build-native-binaries.md#export-dependencies-to-binaries).
* With several iOS apps, each app adopts a library update separately, and the apps can end up on different library
  versions. Library teams may need to support older versions for longer.
  If all apps use the same libraries and update them together, a single shared umbrella framework can be simpler.

### Set up the umbrella module

The following steps describe the general setup. For a complete example project with a library repository
and an iOS app repository, see the [ios-side-umbrella](https://github.com/zsmb13/ios-side-umbrella) sample.

1. In the library repositories, publish each library with its iOS targets, for example, `iosArm64` and
   `iosSimulatorArm64`. See [](multiplatform-publish-lib-setup.md).
2. In the iOS app repository, create a Gradle project with an `umbrella` module.
   Generate the [Gradle wrapper](https://docs.gradle.org/current/userguide/gradle_wrapper.html) for it and commit
   the wrapper files, so that Xcode can run `./gradlew` without a separate Gradle installation.
   In the `settings.gradle.kts` file, add the repository where the libraries are published:

    ```kotlin
    dependencyResolutionManagement {
        repositories {
            // Your internal repository with the published libraries
            maven("https://maven.example.com/releases")
            mavenCentral()
        }
    }

    rootProject.name = "ios-app"
    include(":umbrella")
    ```

3. Declare the library versions in a version catalog, for example, `gradle/libs.versions.toml`.
   This file defines which version of each library the iOS app uses, similar to a lock file for the direct dependencies:

    ```toml
    [versions]
    kotlin = "%kotlinVersion%"
    feature-a = "1.4.0"
    feature-b = "2.0.1"

    [libraries]
    feature-a = { module = "com.example:feature-a", version.ref = "feature-a" }
    feature-b = { module = "com.example:feature-b", version.ref = "feature-b" }

    [plugins]
    kotlin-multiplatform = { id = "org.jetbrains.kotlin.multiplatform", version.ref = "kotlin" }
    ```

4. In the `umbrella/build.gradle.kts` file, declare the libraries as `api` dependencies and export them to the framework:

    ```kotlin
    plugins {
        alias(libs.plugins.kotlin.multiplatform)
    }

    kotlin {
        listOf(
            iosArm64(),
            iosSimulatorArm64(),
        ).forEach { iosTarget ->
            iosTarget.binaries.framework {
                // Name of the module imported in Swift code
                baseName = "Shared"
                isStatic = true
                export(libs.feature.a)
                export(libs.feature.b)
            }
        }

        sourceSets {
            commonMain.dependencies {
                api(libs.feature.a)
                api(libs.feature.b)
            }
        }
    }
    ```

5. Add an empty Kotlin file to the umbrella module, for example, `umbrella/src/commonMain/kotlin/Umbrella.kt`.
   Gradle can't assemble a framework from a module that doesn't contain any source code.
6. Connect the framework to the Xcode project as described in [](multiplatform-direct-integration.md#connect-the-framework-to-your-project).
   In the run script, specify the path to the root of the Gradle project in your repository in the `cd` command,
   and the umbrella module in the `./gradlew` command, for example:

    ```bash
    if [ "YES" = "$OVERRIDE_KOTLIN_BUILD_IDE_SUPPORTED" ]; then
        echo "Skipping Gradle build task invocation due to OVERRIDE_KOTLIN_BUILD_IDE_SUPPORTED environment variable set to \"YES\""
        exit 0
    fi
    cd "$SRCROOT/.."
    ./gradlew :umbrella:embedAndSignAppleFrameworkForXcode
    ```

After that, Swift code can use the API of all exported libraries through `import Shared`.

## Manage changes and versions

When libraries are updated independently, clear versioning rules help library teams and iOS teams avoid surprises.

### Version libraries semantically

Use [semantic versioning](https://semver.org/) for each library, and treat any change to the public API that is
visible from Swift as part of the library's API contract:

* Increase the major version when you remove or change public API in an incompatible way.
* Before removing an API, deprecate it in a minor release and give consuming teams time to migrate.
* Describe changes that affect iOS consumers in the release notes of the library.

To detect accidental changes to the Kotlin API before a release, you can enable
[binary compatibility validation](https://kotlinlang.org/docs/gradle-binary-compatibility-validation.html)
in the library projects.
It checks the Kotlin ABI of the library and helps avoid failures when the umbrella module resolves a newer version of a
library than another library was built against. It doesn't check the Objective-C and Swift API generated for the framework.

### Avoid name clashes between libraries

The Swift API of the framework is generated from all exported libraries together.
Objective-C doesn't support packages, so if two libraries contain classes with the same name, the compiler renames
them, and the renaming algorithm can change between Kotlin releases.
As a result, adding a class to one library can change the Swift names of classes from another library.
See [Name translation](https://kotlinlang.org/docs/native-objc-interop.html#name-translation).

To avoid this, agree on naming rules across library teams, for example, a prefix for each library, or set stable Swift names
with the [`@ObjCName` annotation](https://kotlinlang.org/docs/native-objc-interop.html#change-declaration-names).
It also helps to try a new library version in the iOS app before releasing it, as described in
[Develop against unpublished library changes](#develop-against-unpublished-library-changes).

### Update one library at a time

Update library versions in the iOS repository in separate changes.
This keeps each change small, makes it clear which library introduced a problem, and lets you revert a single update.
Dependency update tools that support Gradle version catalogs can propose these changes automatically when a new
library version is published.

### Align shared dependencies and the Kotlin version

Libraries often depend on common libraries, such as a shared platform module, kotlinx.coroutines, or Ktor.
The umbrella module resolves a single version of each such dependency for the whole framework.
By default, Gradle picks the highest requested version, so an update of one library can also update a dependency used by
other libraries.
To avoid unexpected changes, agree on versions of common dependencies across teams, for example, by publishing a shared
version catalog that all library projects use.

Also keep the Kotlin version in mind.
The Kotlin compiler can consume klib artifacts produced by the same or an earlier Kotlin version, but it isn't guaranteed
to read klibs produced by a later version. For cinterop klibs, there are no compatibility guarantees between
Kotlin versions, so libraries with cinterop dependencies should use the same Kotlin version as the umbrella module.
Make sure that the Kotlin version in the umbrella module is the same as or later than the Kotlin version of all
libraries, and update it in the umbrella module first.
For details, see [Kotlin klib binaries](https://kotlinlang.org/docs/kotlin-evolution-principles.html#kotlin-klib-binaries).

## Multiple iOS apps with different sets of libraries

When several iOS apps consume Kotlin Multiplatform libraries, each app repository contains its own umbrella module.
Each umbrella module declares only the libraries the app needs and pins their versions independently of other apps:

<!-- TODO: diagram of several iOS app repositories with their own umbrella modules, see the KEDOC-337 report -->

* An app doesn't include code from libraries it doesn't use.
* Each app adopts a new library version on its own schedule.
* Library teams don't need to know which apps use their libraries in order to publish a new version.

Don't add several umbrella frameworks to one app. If an app consists of several Swift modules or packages that use
Kotlin code, connect a single umbrella framework to the app and make it available to these modules.
For local Swift packages, see [](multiplatform-spm-local-integration.md).

## Workflows that reduce coordination between teams

With the umbrella module in the iOS app repository, a typical update looks like this:

1. A library team makes a change, increases the library version, and publishes the new version.
2. The library team or an automated tool opens a change in the iOS app repository that updates the library version in the
   version catalog.
3. The iOS team reviews the change, adapts Swift code if the API changed, and merges it when it suits their schedule.

Library teams and iOS teams don't have to synchronize their releases: each library is released on its own schedule,
and each iOS app adopts it on its own schedule.

### Develop against unpublished library changes

When a change spans a library and the iOS app, you can test it in the iOS app before publishing a new library version:

* Publish the library to your local Maven repository with the `publishToMavenLocal` task, add `mavenLocal()`
  to the repositories in the iOS app's Gradle project, and set the local version in the version catalog.
* Alternatively, include the library project in the iOS app's Gradle project as a
  [composite build](https://docs.gradle.org/current/userguide/composite_builds.html) with `includeBuild()`
  in the `settings.gradle.kts` file. If the project in the included build has the same group and name as
  the published library, Gradle builds the library from source instead of downloading the published artifact.
  Gradle configures the whole included build, so your machine needs everything that build requires,
  for example, the Android SDK if the library targets Android.

Keep these changes local and don't commit them to the iOS app repository.

### Continuous integration

* In the library repositories, CI builds and tests the library for all of its targets, including iOS, and publishes
  new versions.
* In the iOS app repository, CI builds the app as usual. The umbrella framework is built as part of the Xcode build,
  so no separate step for building or downloading the framework is needed.

## What's next

* [Configure the export of dependencies to Apple binaries](multiplatform-build-native-binaries.md#export-dependencies-to-binaries)
* [Set up direct integration with Xcode](multiplatform-direct-integration.md)
* [Learn about other repository configurations](multiplatform-project-repository-configurations.md)
