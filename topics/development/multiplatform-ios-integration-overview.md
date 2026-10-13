[//]: # (title: iOS integration methods)

You can integrate a Kotlin Multiplatform shared module into your iOS app. For that, you generate an [iOS framework](https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPFrameworks/Concepts/WhatAreFrameworks.html)
from the shared module and then add it as a dependency to the iOS project:

![iOS integration scheme](ios-integration-scheme.svg)

It's possible to consume this framework as a local or remote dependency. Choose local integration if you want to have
full control over the entire codebase and get instant updates to final applications when the common code changes.

If you want to explicitly separate the code base of your final application from the common code base, set up remote
integration. In this case, the shared code will be integrated into final applications like a regular third-party
dependency.

For the trade-offs of each workflow for the whole team, see [](multiplatform-project-code-sharing-workflow.md).

## What local and remote integration mean for iOS developers

Local and remote integration have the following specifics for iOS developers.

### With local integration

In the local distribution scheme, the iOS app build triggers the generation of the iOS framework. This means that iOS
developers can observe their changes to Kotlin Multiplatform code right away:

![Local source distribution](local-source-distribution.svg){width=700}

The iOS developers need to install IntelliJ IDEA or Android Studio and have a basic knowledge of Kotlin and Gradle.

### With remote integration

Remote integration doesn't significantly change the workflow and build processes for iOS developers.

<table>
  <tr>
     <th>Pros</th>
     <th>Cons</th>
  </tr>
  <tr>
  <td>Non-participating iOS team members don't have to code in Kotlin or learn how to use tools like IntelliJ IDEA/Android Studio or Gradle.
      This lowers the barrier of entry for the team significantly.</td>
<td>
  <list>
   <li>Debugging shared Kotlin code is difficult on iOS.</li>
   <li>The likelihood of iOS team members contributing to shared code significantly decreases.</li>
   <li>When several modules are published as one prebuilt umbrella framework, every change to any module requires a new
       umbrella release. If releases are batched, iOS developers have to adopt changes from several modules at once.</li>
  </list>
</td>
</tr>
</table>

As a start, you may want to use [KMMBridge](https://touchlab.co/trykmmbridge) – a set of build tools that greatly simplifies the
remote distribution workflow. Alternatively, you can always set up a similar workflow on your own:

![Remote artifact distribution](remote-artifact-distribution.svg){width=700}

## Local integration

In a local setup, there are two main integration options. You can use direct integration through a special script, which
makes the Kotlin build a part of the iOS build. If you have Pod dependencies in your Kotlin Multiplatform project,
take the CocoaPods integration approach.

### Direct integration

You can connect the iOS framework directly from the Kotlin Multiplatform project by adding a special script to your Xcode
project. The script is integrated into the build phase of your project's build settings.

This integration method can work for you if you do **not** import CocoaPods dependencies in your Kotlin Multiplatform
project.

If you use the [Kotlin Multiplatform IDE plugin](https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform),
direct integration is applied by default.

For more information, see [Direct integration](multiplatform-direct-integration.md).

### SwiftPM integration with a local package

Your KMP iOS framework can depend on a local Swift package through the [Swift Package Manager](https://docs.swift.org/swiftpm/documentation/packagemanagerdocs/).

This integration method can work for you if:

* You have a mono repository setup with an iOS project that uses Swift packages.
* You don't have irreplaceable CocoaPods dependencies in your Kotlin Multiplatform project.

To learn how to add a local Swift package dependency to your project,
see [Adding Swift packages as dependencies](multiplatform-spm-import.md#importing-local-swift-packages).

### CocoaPods integration with a local podspec

You can connect the iOS framework from the Kotlin Multiplatform project through [CocoaPods](https://cocoapods.org/),
a popular dependency manager for Swift and Objective-C projects.

This integration method works for you if:

* You have a mono repository setup with an iOS project that uses CocoaPods.
* You import CocoaPods dependencies in your Kotlin Multiplatform project.

To learn how to set up a workflow with a local CocoaPods dependency, see [CocoaPods overview and setup](multiplatform-cocoapods-overview.md).

## Remote integration

For remote integration, your project might use the Swift Package Manager (SwiftPM) or the CocoaPods dependency manager to
connect the iOS framework from a Kotlin Multiplatform project.

### SwiftPM with XCFrameworks

You can both export your XCFramework from a Kotlin Multiplatform project and import remote Swift packages as dependencies
into a KMP project:
* For instructions on how to make and distribute a Swift package out of your XCFramework, see [Swift package export setup](multiplatform-spm-export.md).
* To learn how to add Swift packages as dependencies, see the [Swift PM import documentation](multiplatform-spm-import.md).

### CocoaPods integration with XCFrameworks

You can build XCFrameworks with the Kotlin CocoaPods Gradle plugin and then distribute shared parts of your project
separately from mobile apps through CocoaPods.

For more information, see [Build final native binaries](multiplatform-build-native-binaries.md#build-xcframeworks).

## Setting up a local dependency for local development

Many teams choose remote distribution workflows when adopting Kotlin Multiplatform technology to keep the development
process the same for iOS developers. However, it's hard for them to change Kotlin Multiplatform code in this workflow.
We recommend setting up an additional "local development" workflow with a local dependency on a framework
generated from the Kotlin Multiplatform module.

When developers add new functionality, they switch to consuming the Kotlin Multiplatform module as a local dependency.
That allows making changes to common Kotlin code, immediately observing the behavior from iOS, and debugging Kotlin
code. When the functionality is ready, they can switch back to the remote dependency and publish their changes
accordingly. First, they publish changes to the shared modules, and only after that do they make changes to the apps.

For remote distribution workflows, use Swift Package Manager. For local distribution workflow, integrate
the framework directly.

<!-- This tutorial [TODO] describes how to switch workflows by choosing the corresponding scheme in Xcode:
[TODO screenshot] -->

If you use CocoaPods, you can alternatively use CocoaPods for local distribution workflow. You switch between them by
changing the environmental variable as described in the [TouchLab documentation](https://touchlab.co/kmmbridgecocoapodslocal).

## Combining remote libraries with a local framework build

The local and remote workflows can also be combined differently: the Kotlin Multiplatform libraries are published
remotely as regular Kotlin artifacts, while the iOS framework is built locally in the iOS app repository by a small
umbrella module that depends on the published versions.
iOS developers don't edit Kotlin code in this setup, but the iOS build runs Gradle to produce the framework.

This approach is useful when several teams contribute shared libraries to one or more iOS apps.
To learn more, see [](multiplatform-ios-umbrella-module-placement.md).
