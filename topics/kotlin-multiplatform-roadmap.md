# Kotlin Multiplatform roadmap

The Kotlin Multiplatform roadmap is meant as an overview of priorities and general direction for the Kotlin Multiplatform project.

The latest [roadmap blog post](https://blog.jetbrains.com/kotlin/2024/10/kotlin-multiplatform-development-roadmap-for-2025/)
was published on October 28, 2024.
The page below summarizes it and is updated whenever we reach a declared milestone.

Kotlin Multiplatform goals are closely aligned with the [Kotlin roadmap](https://kotlinlang.org/docs/roadmap.html).
Be sure to check it out to have more context for the direction we're taking.

## Key priorities

* Stable Compose Multiplatform for iOS: driving the iOS target to a stable release involves both improving the underlying framework
    and iOS-specific integrations and benchmarks.
* Public release of a standalone Kotlin Multiplatform IDE, tailored specifically for multiplatform development in an optimized
    environment.
* Release the first public version of Kotlin-to-Swift export. With the initial release, we aim to provide experience comparable
    to the existing Objective-C export, and pave the way to fully leverage Swift export in the future.
* Improve the experience of creating multiplatform libraries by providing better tools and guidance. We shall improve
    the klib format to be more flexible and powerful, and provide better templates and instructions for creating multiplatform
    libraries.
* Make Amper suitable for multiplatform mobile development. In 2025, Amper should fully support multiplatform development
    for iOS and Android, including sharing UI code using Compose Multiplatform.

## Compose Multiplatform

Focus areas for Compose Multiplatform are:

* **Jetpack Compose feature parity**. Ensure that all core APIs and components are multiplatform.
* **iOS rendering performance**. Implement benchmarking infrastructure to catch regressions and make performance
    of the framework transparent to the users.
* **Feature completeness for core components**. Complete essential features, including:
    * navigation,
    * resource management,
    * accessibility,
    * internationalization.
* **General stabilization of the framework**. Improve overall stability (including interoperability between Compose and native views)
    while enhancing the user experience through Compose Multiplatform previews.
* **Documentation**. Provide users with all the resources needed to learn and use Compose Multiplatform in a single place.
* **Compose Multiplatform for web**. Reach feature parity with other supported platforms.

## Tooling

We aim to ensure that Kotlin Multiplatform integrates seamlessly with the IDEs already commonly used for KMP development,
such as IntelliJ IDEA and Android Studio, making it more straightforward to share code within or between projects.

We also believe that there is a place for a dedicated multiplatform IDE, and plan to release a standalone Kotlin Multiplatform IDE.
This IDE, based on [JetBrains Fleet](https://www.jetbrains.com/fleet/), will be tailored specifically for Kotlin Multiplatform development.
The initial release will focus on mobile scenarios and aim to:

* Ensure that developers no longer need to switch between different IDEs.
* Provide enhanced language support, delivering an equally great experience in both Kotlin and Swift.
* Create a unified workflow that helps developers with experience in either iOS or Android to feel productive when working with both platforms.

We are also exploring new areas to enhance the development experience:

* Investigating building iOS apps using cloud machines to help developers who don't have convenient access to Apple devices.
* Experimenting with deeper AI tools integration to assist not only in code generation but also in more complex development tasks.

## Kotlin-to-Swift export

Our goal for 2025 is to release the first public version of direct Kotlin-to-Swift export.
The initial release aims to provide a user experience comparable to the existing Objective-C export,
while overcoming the constraints of Objective-C.

This will enable broader support for Swift language and facilitate exporting APIs,
paving the way for future improvements to fully leverage Swift export.

## Library ecosystem

As the Kotlin Multiplatform ecosystem rapidly expands, ensuring the backward compatibility of libraries becomes crucial.
Here's what we plan to do:

* Improve the klib format to allow library creators to leverage their knowledge of building JVM libraries.
* Implement the same code-inlining behavior in Kotlin Multiplatform libraries as for the JVM.
* Provide a tool that ensures your multiplatform library public API remains backward compatible.

We’re also looking to improve the publishing process for Kotlin Multiplatform libraries. We want to:

* Provide templates and comprehensive guidelines for creating and publishing KMP libraries.
* Stabilize klib cross-compilation on different platforms.
* Launch a fully re-designed KMP library publication process.
* Significantly improve the libraries' documentation process. <!--TODO: does that mean automation? -->

While Kotlin Multiplatform will receive significant updates, libraries build with the current format will still work
with newer Kotlin versions.

### Improving search for multiplatform libraries

There are now more than 2500 Kotlin Multiplatform libraries available.
However, despite the extensive selection, it can be challenging for developers to find libraries that meet their specific
needs and support their chosen platforms.

Our goal is to introduce a solution that facilitates the discovery of these libraries and allows developers to try them out easily.

### Amper

Amper is an experimental project configuration and build tool by JetBrains. In 2025, we will focus on making Amper fully
suitable for multiplatform mobile app development for Android and iOS, with shared Compose Multiplatform UI.

We aim to support:

* Running and testing applications locally, on physical devices, and within CI.
* Signing applications and publishing them in the Play Store and App Store.
* IDE integration to ensure a smooth and enjoyable experience.

### Gradle and other build tools

As we look ahead to 2025, our work on Gradle enhancements is outlined in the [Kotlin roadmap](https://kotlinlang.org/docs/roadmap.html#tooling).

Here are the key areas we will be working regarding Kotlin Multiplatform in particular:

* Support declaring Kotlin Multiplatform dependencies at the project level. This will make it easier for developers to manage their project dependencies effectively.
* Improve integration of the Kotlin/Native toolchain into Gradle.
* Implement the next-generation distribution format for multiplatform libraries.
    This will simplify the dependencies model and publication layout for multiplatform libraries, making them easier
    to use with third-party build tools and reducing complexity for library authors.
* Provide full support for Kotlin Multiplatform in Declarative Gradle.
    Our work on the Experimental Kotlin Ecosystem Plugin, which supports Declarative Gradle, aims to help developers
    explore a declarative approach to their Gradle builds.

> * This roadmap is not an exhaustive list of all things the team is working on, only the biggest projects.
> * There's no commitment to delivering specific features or fixes in specific versions.
> * We will adjust our priorities as we go and update the roadmap accordingly.
>
{style="note"}