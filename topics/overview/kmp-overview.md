[//]: # (title: What is Kotlin Multiplatform)

Kotlin Multiplatform (KMP) is a technology that allows reusing code across Android, iOS, desktop, web, and server-side
while integrating with native code where needed.

Kotlin Multiplatform (KMP) is an open-source technology by JetBrains that enables sharing code across Android, iOS, desktop,
web, and server while retaining the advantages of native development.

With Compose Multiplatform, you can also share UI code across multiple platforms for maximum code reuse.

![Illustration of gradual KMP adoption: share part of logic and none of the UI, share all logic without UI, share logic and UI](kmp-graphic.png)

KMP is used in production by organizations of different sizes, from startups to global enterprises,
including Google, Duolingo, Forbes, Philips, McDonald’s, Bolt, H&M, Baidu, Kuaishou, and Bilibili.
These companies have adopted KMP for its flexibility, native performance, ability to deliver native user experiences,
cost efficiency, and support for gradual adoption. [Learn more about companies who adopted KMP](case-studies.topic).

## Why companies choose KMP

### Cost efficiency and faster delivery 🚀

* Reduce duplication and maintenance costs through shared logic
* Release features simultaneously on multiple platforms
* Use Compose Multiplatform to build shared UIs without separate implementations

### Flexibility of code sharing 🔄

* Share isolated modules such as networking or storage
* Share business logic while keeping UIs native
* Share both logic and UI with Compose Multiplatform for maximum reuse

### Native feel on iOS 📱

* Use SwiftUI or UIKit for fully native interfaces
* Get native-feeling apps on Android and iOS with Compose Multiplatform 
* Combine native and shared approaches as needed

### ⚡ Native performance

* Code compiled to platform binaries for near-native performance
* Access platform APIs directly without extra layers


### ✨ AI-powered development

Let Junie, JetBrains’ AI coding agent, handle KMP tasks so your team can move faster

### 👩‍💻 Team collaboration

* Maintain consistency with a shared codebase
* Reduce duplication across platform teams
* Simplify knowledge transfer through unified logic


### 🪜 Gradual adoption

* Start with a single feature or module
* Expand progressively over time 
* Mix native and shared code in the same project

## Discover Kotlin Multiplatform use cases

Look at how companies and developers already enjoy the benefits of shared Kotlin code:

* Learn how companies have successfully adopted KMP in their codebases on our [case studies page](case-studies.topic).
* Check out a wide range of sample apps in our [curated sample list](multiplatform-samples.md) and the GitHub [kotlin-multiplatform-sample](https://github.com/topics/kotlin-multiplatform-sample) topic.
* Search for specific multiplatform libraries among the thousands already present on [klibs.io](https://klibs.io/).

## Learn the basics

You can learn KMP fundamentals in a way that suits you: 

* To quickly see Kotlin Multiplatform in action, try the [quickstart](quickstart.md).
  You'll set up your environment and run a sample application on different platforms.
  * To create an app that shares both UI and business logic code between platforms,
    follow the [shared logic and UI tutorial](compose-multiplatform-create-first-app.md).
  * To see how an Android app can be turned into a multiplatform app,
    check out our [migration tutorial](multiplatform-integrate-in-existing-app.md).
  * To see how you can share some code without sharing UI implementation,
    follow the [shared logic tutorial](multiplatform-create-first-app.md).
* If you want to dig into technical details:
  * Start with the [basic project structure](multiplatform-discover-project.md).
  * Learn about the available [sharing code mechanisms](multiplatform-share-on-platforms.md).
  * See [how dependencies work](multiplatform-add-dependencies.md) in a KMP project.
  * Consider different [iOS integration methods](multiplatform-ios-integration-overview.md).
  * Learn how KMP [compiles code](multiplatform-configure-compilations.md) and [builds binaries](multiplatform-build-native-binaries.md)
    for various targets.
  * Read about [publishing a multiplatform app](multiplatform-publish-apps.md)
    or a [multiplatform library](multiplatform-publish-lib-setup.md).

## Adopt Kotlin Multiplatform at scale

Adopting a cross-platform framework in a team can be a challenge.
To learn about benefits and solutions to potential problems, take a look at our high-level overviews
of cross-platform development:

* [](cross-platform-mobile-development.md): Provides an overview of the different approaches and implementations for cross-platform applications.
* [](multiplatform-introduce-your-team.md): Offers strategies to introduce cross-platform development in a team.
* [](multiplatform-reasons-to-try.md): Lists reasons to adopt Kotlin Multiplatform as your cross-platform solution.