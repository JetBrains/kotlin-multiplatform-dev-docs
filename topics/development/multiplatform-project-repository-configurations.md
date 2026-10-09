[//]: # (title: Repository configurations)

This page describes how to distribute the apps and the shared code between repositories.
For an overview of all project configuration aspects, see [](multiplatform-project-configuration.md).

There are a number of repository configuration options that you can use in new and existing Kotlin Multiplatform
projects, using one repository or a combination of several repositories.

## Monorepo: everything in one repository

A common repository configuration is called a monorepo configuration. This approach is used in Kotlin Multiplatform
samples and tutorials. In this case, the repository contains both Android and iOS apps, as well as the shared module or
several modules, including the umbrella module:

![Monorepo configuration](monorepo-configuration-1.svg){width=700}

![Monorepo configuration](monorepo-configuration-2.svg){width=700}

Typically, the iOS app consumes the Kotlin Multiplatform shared module as a regular framework by using direct or CocoaPods
integration. See [](multiplatform-ios-integration-overview.md) for more details and links to tutorials.

If the repository is under version control, the apps and the shared module have the same version.

<table>
  <tr>
     <th>Pros</th>
     <th>Cons</th>
  </tr>
  <tr>
  <td>
    <list>
       <li>Easy to set up with the help of wizards.</li>
       <li>iOS developers can easily work with Kotlin Multiplatform code since all of the code is located in the same repository.</li>
</list>
</td>
<td>
<list>
  <li>iOS developers need to set up and configure unfamiliar tools.</li>
<li>This approach often doesn't work for existing apps already stored in different repositories.</li>
</list>
</td>
</tr>
</table>

When the existing Android and iOS apps are already stored in different repositories, you can add the Kotlin
Multiplatform part to an Android repository or to a separate repository, instead of merging them.

## Two repositories: Android + shared | iOS

Another project configuration is having two repositories. In this case, the Kotlin Multiplatform repository contains
both the Android app and the shared module, including the umbrella module, while the Xcode project contains the iOS app:

![Two repository configuration](two-repositories.svg){width=700}

The Android and iOS apps can be versioned separately, and the shared module is versioned along with the Android app.

## Three repositories: Android | iOS | shared

One more option is having a separate repository for the Kotlin Multiplatform modules. In this case, the Android and iOS
apps are stored in separate repositories, and the project's shared code can contain multiple feature modules and the
umbrella module for iOS:

![Three repository configuration](three-repositories.svg){width=700}

Each project can be versioned separately. Kotlin Multiplatform modules must also be versioned and published for the
Android or JVM platforms. You can either publish feature modules independently or publish only the umbrella module and
make the Android app depend on it.

Publishing Android artifacts separately can present additional complexity for Android developers
compared to project scenarios where the Kotlin Multiplatform modules are part of the Android project.

When both Android and iOS teams consume the same versioned artifacts, they operate in version parity.
From a team perspective, this avoids the impression that the shared Kotlin Multiplatform code is "owned" by the
Android developers. For large projects that already publish versioned internal Kotlin and Swift packages for feature
development, publishing the shared Kotlin artifacts becomes a part of the existing workflow.

## Many repositories: Android | iOS | multiple libraries

When functionality should be shared between multiple apps on multiple platforms, you might prefer having many
repositories with Kotlin Multiplatform code. For example, you can store a logging library, which is common for the whole
product, in a separate repository with its own versioning.

In this case, you have multiple Kotlin Multiplatform library repositories, and each library is versioned and
published separately, including the iOS (klib) artifacts.
If several iOS apps use different subsets of these libraries, each app needs its own umbrella module that depends on
the libraries the app uses.

We recommend placing this umbrella module in the iOS app repository.
The umbrella module contains no feature code: it lists the libraries and their versions and builds the iOS framework
locally as part of the iOS build.
This way, each library can be updated in the iOS app separately, and each app chooses its own set of libraries:

<!-- TODO: replace with a diagram showing the umbrella module inside each iOS app repository -->

Alternatively, each app can have an additional repository containing the umbrella module, which publishes a prebuilt
umbrella framework for the app:

![Many repository configuration](many-repositories.svg){width=700}

A separate umbrella repository works, but every library update then requires changes in two repositories, which tends
to batch updates and increases coordination between teams.
For a detailed comparison and the recommended setup, see [](multiplatform-ios-umbrella-module-placement.md).

Each library must also be versioned and published for the Android or JVM platforms.
The apps and each library can be versioned separately.

## What's next

* [Learn about code sharing workflows](multiplatform-project-code-sharing-workflow.md)
* [Choose where to place the umbrella module](multiplatform-ios-umbrella-module-placement.md)
