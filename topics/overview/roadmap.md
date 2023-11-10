[//]: # (title: Kotlin Multiplatform roadmap)

INTRO

## Compose Multiplatform

We're dedicated to making Compose Multiplatform a framework that allows creating beautiful and performant applications that look the same way on all supported platforms.
Right now, our main focus is to get Compose for iOS to Stable, but we’re also working on other things. Here’s what we plan to do:
* Make all Jetpack Compose core APIs and components multiplatform.
* Improve rendering performance on iOS.
* Make scrolling and text editing in Compose for iOS apps work as they do in iOS native apps.
* Implement a common API for sharing all types of resources.
* Integrate with iOS and Desktop accessibility APIs.
* Provide a solution for multiplatform navigation.

Many of the aforementioned improvements benefit Compose for Desktop as well. 
Besides it, we're working on improving its stability and evolving it according to the feedback from folks who use it in production.

We also continue to explore what’s possible with Compose for Web, specifically with Wasm. Our nearest goal is to promote it to Alpha, which includes:
Allowing you to port your existing apps and reuse all of your common code.
* Supporting different screen sizes, orientations, and densities.
* Supporting input from a mouse, touchscreen, physical keyboard, or onscreen keyboard.
* Improving performance and binary size.

## Tooling

We’re committed to providing a great IDE experience for Kotlin Multiplatform. 
That means not only investing in the core platform and, for example, migrating Kotlin IDE plugin to K2 compiler frontend, 
but also providing a single tool (Fleet) for all Kotlin Multiplatform targets and codebases where you integrate it, 
eliminating the need to constantly switch between different IDEs.

We plan to quickly iterate over your feedback about using Fleet for Kotlin Multiplatform development, 
and make sure that we have everything you need for your development experience to be great. 
Among other things, we’re going to deliver in the following areas:
* Enhanced support for Compose Multiplatform, including live previews for common code and visual debugging tools.
* IDE help with project configuration.
* Unified and enhanced debugging experience for all parts of your Multiplatform project.

## Multiplatform Core

One of the popular Kotlin Multiplatform scenarios is sharing code with the iOS target. 
We want to focus on the development experience of iOS developers, who work with Kotlin Multiplatform frameworks in their codebases.

The main initiative in this area is a direct Kotlin to Swift export. It will eliminate the Objective-C bottleneck, 
allowing for broader Swift language support and more natural APIs export. Additionally, we are creating tools specifically for Kotlin library authors. 
These tools are designed to improve the compatibility and user-friendliness of Kotlin APIs when exported to Swift. 
We are also paying close attention to tooling. 
IDE and build systems are important parts of the developer experience, and our goal is to ensure that the Swift Export integrates smoothly.

Our other initiatives include Kotlin/Native compilation times improvements, enhancements in CocoaPods integration and introducing support for exporting your framework with SwiftPM.

We also plan to continue exploring ways to improve the build setup of Kotlin Multiplatform applications. 
With Kotlin 1.9.20, we released some huge improvements in the Gradle Multiplatform DSL, making it easier to read and write. 
We will continue to gradually improve it. Besides that, we’re experimenting with Amper, a build management tool for project configuration, 
build, packaging and publication. You can learn more about it in [this blogpost]().

## Library ecosystem

The Kotlin Multiplatform ecosystem is growing fast, and as it grows, the backwards compatibility of the libraries becomes crucial.
To do this well, both the JetBrains team and library creators must work together. Here's our plan:
* Improve the klib format, so library creators can use their knowledge of building JVM libraries.
* Implement the same code inlining behavior in Kotlin Multiplatform libraries as for JVM.
* Provide a tool that ensures that your multiplatform library public API hasn’t changed in an incompatible way.

We’re also going to improve the publishing process for Multiplatform libraries. Specifically, we plan to:
* Make it possible to build and publish a KMP library without having a Mac machine.
* Provide templates and extensive guidelines for creating and publishing a Multiplatform library.

Even though Kotlin Multiplatform is now stable, we're planning big updates.
But don't worry: libraries made with the current format will still work with newer Kotlin versions.

## Useful links

* [Kotlin language roadmap](http://kotlinlang.org/docs/roadmap.html)
