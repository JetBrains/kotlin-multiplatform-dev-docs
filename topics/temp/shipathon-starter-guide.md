[//]: # (title: Kotlin Multiplatform starter guide)

## Where to start

1. Learn about Kotlin Multiplatform (KMP) and Compose Multiplatform (CMP):
   What they are, their [advantages and use cases](kmp-overview.md).
2. [Try KMP out on a sample project](quickstart.md) to see how it's organized and how it runs on different platforms.

## Learn KMP basics

The basics include:

* [Understand how a KMP / CMP project is organized](multiplatform-discover-project.md).
  This covers:
    * Common and platform-specific code in a shared module.
    * Targeted platform declarations.
* [Adding a dependency to a KMP project](multiplatform-add-dependencies.md).
    * For a practical example of multiplatform and platform-specific dependency organization, see our [sample](https://github.com/kotlin-hands-on/get-started-with-kmp/tree/main).
    * The tutorial that leads to the final state of that sample is [available in the documentation](multiplatform-upgrade-app.md).
* If you are already familiar with KMP, make sure you're up to date with the [recommended project structure](multiplatform-project-recommended-structure.md)
  for an average project.
  It takes into account how the release of Android Gradle plugin 9.0 affected the requirements for a KMP project
  and covers:
    * Module structure (independent app modules with shared code modules used as libraries).
    * Creating new app modules and transitioning from an older structure used with AGP 8.
* [Suggested video on project structure](https://www.youtube.com/watch?v=Atvl0l7fm1Y) recorded by a JetBrains developer advocate.

<!-- ## \[AI Agents scenario tools TODO\] -->

## Share code

There are different ways to share code in a KMP project, with some platform specifics:

* The basic examples of calling common code from app modules are covered in onboarding tutorials:
    * [for native UI and shared logic](multiplatform-create-first-app.md)
    * [for shared UI and logic](compose-multiplatform-create-first-app.md)
* [How to access platform-specific APIs](multiplatform-connect-to-apis.md):
    * includes a video on the subject
    * use multiplatform libraries when possible
    * use the expect/actual mechanism when no multiplatform library is available.
* While calling shared Kotlin from Android Kotlin is relatively straightforward, iOS interoperability takes some getting to know it:
    * [Learn how to integrate your shared code with the iOS app](multiplatform-ios-integration-overview.md#local-integration) (all samples referenced in this doc have examples of iOS integration set up).
      > CocoaPods is generally being phased out in favor of Swift Package Manager and is not something we recommend using in new projects.
      >
      {style="note"}   
    * Check out the [sample and tutorial](multiplatform-upgrade-app.md#add-more-dependencies) that includes making Kotlin coroutines work with iOS.
    * See the guide on using existing [SPM packages in your KMP iOS app](multiplatform-spm-import.md).
    * Read the [in-depth explanation of calling Swift / ObjC from Kotlin](https://kotlinlang.org/docs/native-objc-interop.html) and vice versa.
    * Learn about the more straightforward [Swift export](https://kotlinlang.org/docs/native-swift-export.html) approach (currently in Alpha).
    * In general, the less interop, the better, so for a smoother experience we recommend relying on Compose Multiplatform to build the bulk of your UI for all platforms.

## Discover the ecosystem

A comprehensive catalog of multiplatform libraries is available at [klibs.io](https://klibs.io/):

* Most popular cases are already covered with robust solutions,
  usually with available alternatives:
  [SQLDelight](https://sqldelight.github.io/sqldelight/) and [Room](https://developer.android.com/kotlin/multiplatform/room) for databases, [Ktor](https://ktor.io/) and [OkHttp](https://square.github.io/okhttp/) for networking, [Coil](https://coil-kt.github.io/coil/) for image loading, and so on.
* App samples built to use multiplatform libraries for most popular use cases are available:
    * [SQLDelight / Ktor / kotlinx-serialization / Koin](https://github.com/kotlin-hands-on/kmp-networking-and-data-storage/tree/final)
      with the corresponding [tutorial](multiplatform-ktor-sqldelight.md).
    * [The multiplatform Jetcaster app](https://github.com/kotlin-hands-on/jetcaster-kmp-migration)
      converted from the [original Android sample](https://github.com/android/compose-samples/tree/main/Jetcaster).

## Create a KMP library

If you decide to create apps which share code by using a multiplatform library, check out these documentation pages:

* [Basic library tutorial](create-kotlin-multiplatform-library.md)
* [Publishing configuration for a KMP library](multiplatform-publish-lib-setup.md)
* Tutorials on publishing artifacts to [Maven Central](multiplatform-publish-libraries-to-maven.md) and [npm](multiplatform-publish-libraries-to-npm.md)

## Publish the artifacts

* Read the [general article on publishing KMP apps](multiplatform-publish-apps.md).
* Don't forget about the [privacy manifest](multiplatform-privacy-manifest.md) required by the Apple App Store.

## Learning resources catalog

All of the mentioned resources, along with more in-depth guides and third-party content, are catalogued
on the [Learning resources](kmp-learning-resources.md) page.