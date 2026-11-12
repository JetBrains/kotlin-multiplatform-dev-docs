[//]: # (title: Code sharing workflow)

This page describes how apps can consume shared code during development.
For an overview of all project configuration aspects, see [](multiplatform-project-configuration.md).
For the available ways to connect a framework to an Xcode project, see [](multiplatform-ios-integration-overview.md).

An app can consume the code of Kotlin Multiplatform shared modules as a _local_ or _remote_
dependency. You can use a local dependency by building the shared code from source as part of the app build. In this case,
you don't need to publish the shared code. Alternatively, you can publish an artifact with the shared code somewhere and
make the app consume it as a remote dependency, like any other third-party dependency.

How much each workflow costs can depend on the platform.
Android and desktop JVM apps, for example, consume the shared code directly through Kotlin build,
and published artifacts usually include sources, so it's simple to browse and debug a remote dependency.
iOS apps consume the shared code as a binary framework, which makes workflows more involved.
For details on iOS integration, see [](multiplatform-ios-integration-overview.md).

## Local: source distribution

Local distribution is where an app consumes the code of Kotlin Multiplatform modules without the
need for publishing.
This workflow is used, for example,
when both Android and iOS team members want to edit the shared Kotlin Multiplatform code.

Local distribution can be used in monorepo project configurations as the default
workflow, without the need to publish artifacts, or for local development,
in addition to the remote workflow.
See [Use a local dependency during development](#use-a-local-dependency-during-development) for more details.

This workflow is most effective when all the team members are ready to edit code in the whole project. It includes both
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
    <li>Managing changes becomes difficult as more code is shared and the team grows.</li>
  </list>
</td>
</tr>
</table>

## Remote: artifact distribution

Remote distribution means that the shared code is published as an artifact, for example, to a Maven repository, and consumed
by the apps like any other dependency. Apps can use different workflows: for example, the Android app may consume the
shared code locally and the iOS app remotely.

Remote distribution is often used to gradually introduce the technology to existing projects.
Teams with two or more repositories mainly use remote distribution to store project code.

<table>
  <tr>
     <th>Pros</th>
     <th>Cons</th>
  </tr>
  <tr>
  <td>App teams don't need to build or edit the shared code, and each app can stay on a version until it's ready to update.</td>
<td>
  <list>
    <li>Slower workflow for app developers, as the process of editing and building the shared code involves publishing and versioning.</li>
   <li>The maintenance of the shared code rests entirely on participating team members.</li>
  </list>
</td>
</tr>
</table>

## Use a local dependency during development

When apps consume the shared code remotely, you can still switch to a local dependency while you develop a change,
and switch back to the published version when the change is released.

For apps built with Gradle, such as Android and desktop JVM apps, include the project with the shared code as a
[composite build](https://docs.gradle.org/current/userguide/composite_builds.html) with `includeBuild()`.
If the included project has the same group and name as the published artifact, Gradle builds it from source instead
of downloading the published version.

For iOS apps, see [Setting up a local dependency for local development](multiplatform-ios-integration-overview.md#direct-dependency-for-local-development-remote-for-publishing).

## What's next

* [Learn about module configurations](multiplatform-project-module-configurations.md)
* [Learn about repository configurations](multiplatform-project-repository-configurations.md)
* [Choose an iOS integration method](multiplatform-ios-integration-overview.md)
