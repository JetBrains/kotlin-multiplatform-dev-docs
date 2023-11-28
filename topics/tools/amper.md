[//]: # (title: Project configuration with Amper)

[Amper](https://github.com/JetBrains/amper/tree/HEAD) is a new tool created by JetBrains to help you configure projects
for building, packaging, publishing, and more. With Amper, you can spend less time dealing with build systems and focus
on addressing real business challenges instead.

Amper lets you create configuration files for Kotlin Multiplatform applications that work on the JVM, Android, iOS,
macOS, and Linux, as well as for multiplatform libraries that work with all of these supported targets.

> Amper is currently [Experimental](supported-platforms.md#core-kotlin-multiplatform-technology-stability-levels).
> You're welcome to try it in your Kotlin Multiplatform projects.
> We would appreciate your feedback in [YouTrack](https://youtrack.jetbrains.com/issues/AMPER).
>
{type="warning"}

## How Amper works

Amper currently uses Gradle as the backend and YAML as the frontend defining the project configuration. It
supports custom tasks, CocoaPods, library publishing to Maven, and packaging desktop apps through the Gradle interop.

With Amper, you can set up a configuration for platform-specific applications and shared Kotlin libraries.
They are declared as modules in a `.yaml` module manifest file using a special declarative DSL.

The core concept of this DSL is Kotlin Multiplatform. Amper allows you to configure Kotlin Multiplatform projects
quickly and easily without having to dive deep into complex Gradle concepts. The Amper DSL offers a special syntax
enabling you to work with multiplatform configurations, including dependencies, settings, and so on.

Here is an example of Amper's manifest file for a Kotlin Multiplatform shared library that can be used with JVM,
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
  such as CocoaPods and Swift Package Manager.
* The `@platform` qualifier marks platform-specific sections, including dependencies and settings.

## Try Amper

You can try Amper in one of the following ways:

* Use [IntelliJ IDEA](https://www.jetbrains.com/idea/nextversion/) 2023.3 and later for JVM and Android projects
  (starting with build 233.11555).
* Use [Fleet](https://www.jetbrains.com/fleet/download) for JVM, Android, and Kotlin Multiplatform projects
  (starting with build 1.26.104).
* Use [Gradle](https://docs.gradle.org/current/userguide/userguide.html) to build Amper projects from the command line
  or your CI/CD tool.

Follow [this tutorial](https://github.com/JetBrains/amper/tree/HEAD/docs/Tutorial.md) to create your first Kotlin
Multiplatform project with Amper. Explore the [documentation](https://github.com/JetBrains/amper/tree/HEAD/docs/Documentation.md)
to learn more about Amper's functionality and design.

Feel free to submit any feedback you might have to our [issue tracker](https://youtrack.jetbrains.com/issues/AMPER).
Your input will help us shape the future of Amper.

## What's next

* Check out the [JetBrains blog](https://blog.jetbrains.com/blog/2023/11/09/amper-improving-the-build-tooling-user-experience)
  to learn more about our motivation behind creating Amper, its use cases, the current state of the project, and its future.
* See the [Amper FAQ](https://github.com/JetBrains/amper/tree/HEAD/docs/FAQ.md) to find answers to the most popular
  questions.
* Read the [Amper documentation](https://github.com/JetBrains/amper/tree/HEAD/docs/Documentation.md), which covers different
  aspects of Amper's functionality and design.
