[//]: # (title: What is Kotlin Multiplatform)

Kotlin Multiplatform (KMP) is an open-source technology by JetBrains that enables sharing code across Android, iOS, desktop,
web, and server while retaining the advantages of native development.

With Compose Multiplatform, you can also share UI code across multiple platforms for maximum code reuse.

KMP is used in production by organizations of different sizes, from startups to global enterprises,
including Google, Duolingo, Forbes, Philips, McDonald’s, Bolt, H&M, Baidu, Kuaishou, and Bilibili.
These companies have adopted KMP for its flexibility, native performance, ability to deliver native user experiences,
cost efficiency, and support for gradual adoption. [Learn more about companies who adopted KMP](case-studies.topic).

## Why companies choose KMP

### Cost efficiency and faster delivery

Kotlin Multiplatform helps streamline both technical and organizational processes:

* You can reduce duplication and maintenance costs by sharing logic and UI code across platforms.
This also allows to release features simultaneously on multiple platforms.
* You make team collaboration easier as the knowledge of unified logic reflected in shared code
is easier to transfer between team members, and there's less duplication of effort between dedicated platform teams.

In addition to faster time to market, <format style="bold," color="Blue">55%</format> users reported improved collaboration after adopting KMP
and <format style="bold," color="Purple">65%</format> of teams reported improved performance and quality, (in the KMP Survey Q2 2024).

### Flexibility of code sharing

You can share code on your terms: share isolated modules, such as networking or storage, and progressively
expand shared code over time.
You can also share the entirety of business logic while keeping the UI native, or gradually migrate the UI as well
using Compose Multiplatform.

![Illustration of gradual KMP adoption: share part of logic and none of the UI, share all logic without UI, share logic and UI](kmp-graphic.png){width="700"}

### Native feel on iOS

You can fully build your UI using SwiftUI or UIKit, get uniform experience on Android and iOS with Compose Multiplatform,
or mix and match combining native and shared UI code as needed.

Regardless of the approach, you can produce apps which feel native on each platform: 

<video src="https://www.youtube.com/watch?v=LB5a2FRrT94"/>

### Native performance

Kotlin Multiplatform leverages [Kotlin/Native](https://kotlinlang.org/docs/native-overview.html)
to produce native binaries and access platform APIs directly where virtual machines are undesirable or impossible,
for example, on iOS.

This helps achieve near-native performance while writing platform-agnostic code:

![Graphs showing comparable performance of Compose Multiplatform and SwiftUI on iOS on iPhone 13 and iPhone 16](cmp-ios-performance.png){width="700"}

### Seamless tooling

IntelliJ IDEA and Android Studio provide smart IDE support for KMP with the [Kotlin Multiplatform IDE plugin](https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform)
common UI previews, Compose Hot Reload, cross-language navigation, refactorings, and debugging across Kotlin and Swift code.

<video src="https://youtu.be/ACmerPEQAWA"/>

### AI-powered development

Let [Junie](https://jetbrains.com/junie), the JetBrains’ AI coding agent, handle KMP tasks so your team can move faster.

## Discover Kotlin Multiplatform use cases

Look at how companies and developers already enjoy the benefits of shared Kotlin code:

* Learn how companies have successfully adopted KMP in their codebases on our [case studies page](case-studies.topic).
* Check out a wide range of sample apps in our [curated sample list](multiplatform-samples.md) and the GitHub [kotlin-multiplatform-sample](https://github.com/topics/kotlin-multiplatform-sample) topic.
* Search for specific multiplatform libraries among the thousands already present on [klibs.io](https://klibs.io/).

## Learn the basics

To quickly see Kotlin Multiplatform in action, try the [quickstart](quickstart.md).
You'll set up your environment and run a sample application on different platforms.

Choose a use case
: * To create an app that shares both UI and business logic code between platforms,
follow the [shared logic and UI tutorial](compose-multiplatform-create-first-app.md).
  * To see how an Android app can be turned into a multiplatform app,
  check out our [migration tutorial](multiplatform-integrate-in-existing-app.md).
  * To see how you can share some code without sharing UI implementation,
  follow the [shared logic tutorial](multiplatform-create-first-app.md).

Dig into technical details
: * Start with the [basic project structure](multiplatform-discover-project.md).
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