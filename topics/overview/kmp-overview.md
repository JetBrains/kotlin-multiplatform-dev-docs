[//]: # (title: What is Kotlin Multiplatform)

Kotlin Multiplatform (KMP) is a technology that allows reusing code across Android, iOS, desktop, web, and server-side
while integrating with native code where needed.

## Why you should adopt Kotlin Multiplatform

Kotlin Multiplatform is productive:

* **Start small, scale smoothly**. You can start sharing code for a single feature or a single screen and expand from there.
* **Native speed**. Apps made with Kotlin Multiplatform are compiled to native code, so Compose UIs feel native on every platform,
  without a custom engine or a VM.
* **Built-in tooling**. Kotlin Multiplatform and tooling around it are built by JetBrains, the team behind IntelliJ IDEA.
  Tools tailored for multiplatform development help with onboarding and development efficiency.
* **Open source**. Just like for Kotlin itself, source code for both Kotlin Multiplatform and Compose Multiplatform is open and free to use.

Kotlin Multiplatform is cost-efficient:

* **Faster time to market**. Ship updates to cross-platform functionality for several platforms simultaneously.
* **Real impact**. In addition to faster time to market, 65% of teams reported improved performance and quality,
  and 55% reported improved collaboration after adopting KMP (based on the KMP survey ran in 2024).
* **Smaller teams**. Kotlin Multiplatform allows one team to use one language for everything and own the entire project
  without splitting the code into, for example, iOS and Android silos.
* **Less code**. Sharing code means writing and maintaining less code to achieve the same result.

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