# Kotlin Multiplatform roadmap

The Kotlin Multiplatform roadmap is meant as an overview of priorities and general direction for the Kotlin Multiplatform project.

The latest roadmap blog post was published on (TODO: date and link). The page below summarizes it and is updated
whenever we reach a declared milestone.

Kotlin Multiplatform goals are closely aligned with the [Kotlin roadmap](https://kotlinlang.org/docs/roadmap.html).
Be sure to check it out to have more context for the direction we're taking.

## Key priorities

* Stable Compose Multiplatform for iOS: driving the iOS target to a stable release involves both improving the underlying framework
    and iOS-specific integrations and benchmarks.
* Release the first public version of Kotlin-to-Swift export. With the initial release, we aim to provide experience comparable
    to the existing Objective-C export, and pave the way to fully leverage Swift export in the future.
* Improve the experience of creating multiplatform libraries by providing better tools and guidance. We shall improve
    the klib format to be more flexible and powerful, and provide better templates and instructions for creating multiplatform
    libraries.
* Make Amper suitable for multiplatform mobile development. In 2025, Amper should fully support multiplatform development
    for iOS and Android, including sharing UI code using Compose Multiplatform.

## Compose Multiplatform

## Library ecosystem

## Build tools 

### Amper

Amper is an experimental project configuration and build tool by JetBrains. In 2025, we will focus on making Amper fully
suitable for multiplatform mobile app development for Android and iOS, with shared Compose Multiplatform UI.

This includes:

* Running and testing applications locally, on physical devices, and within CI.
* Signing applications and publishing them in the Play Store and App Store.
* IDE support to ensure a smooth and enjoyable experience.


### Gradle integration and feature support


> * This roadmap is not an exhaustive list of all things the team is working on, only the biggest projects.
> * There's no commitment to delivering specific features or fixes in specific versions.
> * We will adjust our priorities as we go and update the roadmap approximately every six months.
>
{style="note"}