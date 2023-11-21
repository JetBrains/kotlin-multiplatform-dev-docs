[//]: # (title: Project configuration with Amper)

[Amper](https://github.com/JetBrains/amper/tree/main) is a new tool for project configuration created by JetBrains.
It helps configure projects for building, packaging, publishing, and more. The goal of Amper is to allow developers
to spend less time dealing with build systems and focus on solving actual business tasks instead.

With Amper, you can create configuration files for Kotlin Multiplatform applications that work on JVM, Android, iOS,
macOS, and Linux, as well as for multiplatform libraries that work with all supported targets.

> Amper is currently [Experimental](supported-platforms.md#core-kotlin-multiplatform-technology-stability-levels).
> You're welcome to try it in your Kotlin Multiplatform projects.
> We would appreciate your feedback in [YouTrack](https://youtrack.jetbrains.com/issues/AMPER).
>
{type="warning"}

## How Amper works

Amper currently uses Gradle as the backend and YAML as the frontend that defines the project configuration. It
supports custom tasks, library publishing to Maven, CocoaPods, and packaging desktop apps through the Gradle interop.

With Amper, you can set up a configuration for platform-specific applications and shared Kotlin libraries.
They are declared as modules in a `.yaml` module manifest file using a special declarative DSL.

The core concept of this DSL is Kotlin Multiplatform. Amper allows you to configure Kotlin Multiplatform projects
quickly and easily without having to dive deep into complex Gradle concepts. The Amper's DSL offers a special syntax to
work with multiplatform configuration: dependencies, settings, and so on.

Here is an example of Amper's manifest file for a Kotlin Multiplatform shared library which can be used with JVM,
Android, and iOS applications:

```yaml
product:
  type: lib
  platforms: [ jvm, android, iosArm64, iosSimulatorArm64, iosX64 ]

# Shared Compose Multiplatform dependencies:
dependencies:
  - org.jetbrains.compose.foundation:foundation:1.5.0-rc01: exported
  - org.jetbrains.compose.material3:material3:1.5.0-rc01: exported

# Android-only dependencies  
dependencies@android:
  # Integration compose with activities
  - androidx.activity:activity-compose:1.7.2: exported
  - androidx.appcompat:appcompat:1.6.1: exported

# iOS-only dependencies with a dependency on a CocoaPod
# Note that CocoaPods dependencies are not yet implemented in the prototype
dependencies@ios:
  - pod: 'FirebaseCore'
    version: '~> 6.6'

settings:
  # Enable Kotlin serialization
  kotlin:
    serialization: json

  # Enable Compose Multiplatform framework
  compose: enabled
```

* The `product` section defines the project type and the list of targeted platforms.
* The `dependencies` section adds not only Kotlin and Maven dependencies, but also platform-specific package managers,
  such as CocoaPods, Swift Package Manager.
* The `@platform` qualifier marks platform-specific sections, including dependencies and settings.

## Try Amper

There are multiple ways for you to try out Amper:

* In [IntelliJ IDEA](https://www.jetbrains.com/idea/nextversion/) 2023.3 and later for JVM and Android projects
  (starting with the build 233.11555).
* In [Fleet](https://www.jetbrains.com/fleet/download) for the JVM, Android, and Kotlin Multiplatform project
  (starting with the build 1.26.104).
* Using [Gradle](https://docs.gradle.org/current/userguide/userguide.html) to build Amper projects from the command line
  or CI/CD.

Follow [this tutorial](https://github.com/JetBrains/amper/tree/main/docs/Tutorial.md) to create your first Kotlin
Multiplatform project with Amper. Explore the [documentation](https://github.com/JetBrains/amper/tree/main/docs/Documentation.md)
to learn more about the Amper design and its functionality.

Feel free to submit any feedback in our [issue tracker](https://youtrack.jetbrains.com/issues/AMPER). It'll help us
shape the future of Amper.

## What's next

* Check out [JetBrains blog](https://blog.jetbrains.com/blog/2023/11/09/amper-improving-the-build-tooling-user-experience)
  to learn more about the motivation behind Amper, its use cases, the current state of the project, and its future.
* See [Amper FAQ](https://github.com/JetBrains/amper/tree/main/docs/FAQ.md) to find answers to the most popular
  questions.
* Read [Amper documentation](https://github.com/JetBrains/amper/tree/main/docs/Documentation.md) that covers different
  aspects of Amper's design and functionality.
