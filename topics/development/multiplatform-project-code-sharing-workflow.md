[//]: # (title: Code sharing workflow)

This page describes how the iOS app can consume shared code during development.
For an overview of all project configuration aspects, see [](multiplatform-project-configuration.md).
For the available ways to connect a framework to an Xcode project, see [](multiplatform-ios-integration-overview.md).

The iOS app can consume a framework generated from the Kotlin Multiplatform shared modules as a _local_ or _remote_
dependency. You can use a local dependency by providing a local path to the framework in the iOS build. In this case,
you don't need to publish the framework. Alternatively, you can publish an artifact with the framework somewhere and
make the iOS app consume it as a remote dependency, like any other third-party dependency.

## Local: source distribution

Local distribution is where the iOS app consumes a Kotlin Multiplatform module framework without the
need for publishing. The iOS app can either integrate the framework directly or by using CocoaPods.

This workflow is typically used when both Android and iOS team members want to edit the shared Kotlin Multiplatform
code. The iOS developers need to install IntelliJ IDEA or Android Studio and have a basic knowledge of Kotlin and Gradle.

In the local distribution scheme, the iOS app build triggers the generation of the iOS framework. This means that iOS
developers can observe their changes to Kotlin Multiplatform code right away:

![Local source distribution](local-source-distribution.svg){width=700}

This scenario is typically used in two cases. First, it can be used in monorepo project configurations as the default
workflow, without the need to publish artifacts. Second, it can be used for local development, in addition to the remote
workflow. See [Setting up a local dependency for local development](#setting-up-a-local-dependency-for-local-development) for more
details.

This workflow is most effective when all of the team members are ready to edit code in the whole project. It includes both
Android and iOS parts after making changes to the common parts. Ideally, every team member can have IntelliJ IDEA/Android Studio and
Xcode installed to open and run both Android and iOS apps after making changes to the common code.

<table>
  <tr>
     <th>Pros</th>
     <th>Cons</th>
  </tr>
  <tr>
  <td>
    <list>
       <li>Both Android and iOS team members can easily edit Kotlin Multiplatform code, ensuring that creating and maintaining shared code is a shared responsibility. This helps prevent teams' isolation and encourages collaboration.</li>
       <li>This approach doesn't require separate versioning and publishing of the shared code.</li>
       <li>The development workflow is quicker, as iOS team members don't have to wait for artifacts to be created and published.</li>
   </list>
</td>
<td>
  <list>
    <li>Team members need to set up a full development environment on their machines.</li>
    <li>iOS developers have to learn how to use IntelliJ IDEA or Android Studio and Gradle.</li>
    <li>Managing changes becomes difficult as more code is shared and the team grows.</li>
  </list>
</td>
</tr>
</table>

## Remote: artifact distribution

Remote distribution means that the framework artifact is published using Swift Package Manager or as a CocoaPod and consumed
by the iOS app. The Android app may consume the binary dependency either locally or remotely.

Remote distribution is often used to gradually introduce the technology to existing projects. It doesn't significantly
change the workflow and build processes for iOS developers. Teams with two or more repositories mainly use remote
distribution to store project code.

As a start, you may want to use [KMMBridge](https://touchlab.co/trykmmbridge) – a set of build tools that greatly simplifies the
remote distribution workflow. Alternatively, you can always set up a similar workflow on your own:

![Remote artifact distribution](remote-artifact-distribution.svg){width=700}

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
    <li>Slower workflow for iOS developers, as the process of editing and building the shared code involves publishing and versioning.</li>
   <li>Debugging shared Kotlin code is difficult on iOS.</li>
   <li>The likelihood of iOS team members contributing to shared code significantly decreases.</li>
   <li>The maintenance of the shared code rests entirely on participating team members.</li>
   <li>When several modules are published as one prebuilt umbrella framework, every change to any module requires a new
       umbrella release. If releases are batched, iOS developers have to adopt changes from several modules at once.</li>
  </list>
</td>
</tr>
</table>

### Setting up a local dependency for local development

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

## What's next

* [Learn about module configurations](multiplatform-project-module-configurations.md)
* [Learn about repository configurations](multiplatform-project-repository-configurations.md)
* [Choose an iOS integration method](multiplatform-ios-integration-overview.md)
