[//]: # (title: Project configuration with Amper)

[Amper](https://amper.org) is a new tool created by JetBrains to help you configure projects
for building, packaging, publishing, and more. With Amper, you can spend less time dealing with build systems and focus
on addressing real business challenges instead.

Amper lets you create configuration files for Kotlin Multiplatform applications that work on the JVM, Android, iOS,
macOS, Windows, and Linux, as well as for multiplatform libraries that work with all of these supported targets.

> Amper is currently [Experimental](supported-platforms.md#general-kotlin-stability-levels).
> You're welcome to try it in your Kotlin Multiplatform projects.
> We would appreciate your feedback in [YouTrack](https://youtrack.jetbrains.com/issues/AMPER).
>
{style="warning"}

## How Amper works

Amper is a standalone CLI application, and allows configuring your project using YAML files.

With Amper, you can set up a platform-specific applications and shared Kotlin libraries.
They are declared as modules in a `module.yaml` manifest file using a special declarative DSL.

The core concept of this DSL is Kotlin Multiplatform. Amper allows you to configure Kotlin Multiplatform projects
quickly and easily without having to dive deep into complex concepts. The Amper DSL offers a special syntax
enabling you to work with multiplatform configurations, including dependencies, settings, and so on.

Here is an example of Amper's module file for a Kotlin Multiplatform shared library that can be used with JVM,
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

settings:
  # Enable Kotlin serialization
  kotlin:
    serialization: json

  # Enable Compose Multiplatform framework
  compose: enabled
```

* The `product` section defines the project type and the list of targeted platforms.
* The `dependencies` section adds Maven dependencies, and in the future may support platform-specific package managers,
  such as CocoaPods and Swift Package Manager.
* The `@platform` qualifier marks platform-specific sections, including dependencies and settings.

## Try Amper

Check out Amper's [Getting Started guide](https://jb.gg/amper/get-started) to try it out yourself.

Feel free to submit any feedback you might have to our [issue tracker](https://jb.gg/amper-issues).
Your input will help us shape the future of Amper.

## What's next

* Check out the [JetBrains blog](https://blog.jetbrains.com/blog/2023/11/09/amper-improving-the-build-tooling-user-experience)
  to learn more about our motivation behind creating Amper, its use cases, the current state of the project, and its future.
* Check out the [Amper website](https://amper.org) to read the guides and comprehensive docs.
