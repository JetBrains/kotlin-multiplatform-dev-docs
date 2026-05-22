[//]: # (title: Project configuration with Kotlin Toolchain)

[Kotlin Toolchain](https://kotlin-toolchain.org/) is a tool created by JetBrains to help you configure projects
for building, packaging, publishing, and more. With Kotlin Toolchain, you can spend less time dealing with build systems and focus
on addressing real business challenges instead.

Kotlin Toolchain lets you create configuration files for Kotlin Multiplatform applications that work on the JVM, Android, iOS,
macOS, Windows, and Linux, as well as for multiplatform libraries that work with all of these supported targets.

> Kotlin Toolchain is in [Alpha](supported-platforms.md#general-kotlin-stability-levels).
> You're welcome to try it in your Kotlin Multiplatform projects.
> We would appreciate your feedback in [YouTrack](https://youtrack.jetbrains.com/issues/AMPER).
>
{style="warning"}

## How Kotlin Toolchain works

Kotlin Toolchain is a standalone CLI application, and allows configuring your project using YAML files.

With Kotlin Toolchain, you can set up platform-specific applications and shared Kotlin libraries.
They are declared as modules in a `module.yaml` manifest file using a special declarative DSL.

The core concept of this DSL is Kotlin Multiplatform. Kotlin Toolchain allows you to configure Kotlin Multiplatform projects
quickly and easily without having to dive deep into complex concepts. The Kotlin Toolchain DSL offers a special syntax
enabling you to work with multiplatform configurations, including dependencies, settings, and so on.

Here is an example of a Kotlin module file for a Kotlin Multiplatform shared library that can be used with JVM,
Android, and iOS applications:

```yaml
product:
  type: kmp/lib
  platforms: [ jvm, android, iosArm64, iosSimulatorArm64 ]

# Shared Compose Multiplatform dependencies:
dependencies:
  - $compose.foundation: exported
  - $compose.material3: exported

# Android-only dependencies  
dependencies@android:
  # Integrate compose with activities
  - androidx.activity:activity-compose:1.7.2: exported
  - androidx.appcompat:appcompat:1.6.1: exported

settings:
  # Enable Kotlin serialization
  kotlin:
    serialization: json

  # Enable Compose Multiplatform framework
  compose:
    enabled: true
```

* The `product` section defines the project type and the list of targeted platforms.
* The `dependencies` section adds Maven dependencies, and in the future may support platform-specific package managers,
  such as CocoaPods and Swift Package Manager.
* The `$compose` namespace is a built-in library catalog that provides access to all optional Compose modules.
* The `@platform` qualifier marks platform-specific sections, including dependencies and settings.

## Try Kotlin Toolchain

Check out Kotlin Toolchain's [Getting Started guide](https://kotlin-toolchain.org/dev/getting-started/) to try it out yourself.

Feel free to submit any feedback you might have to our [issue tracker](https://jb.gg/amper-issues).
Your input will help us shape the future of Kotlin Toolchain.

## What's next

<!---
* Check out the [JetBrains blog](https://blog.jetbrains.com/blog/2023/11/09/amper-improving-the-build-tooling-user-experience)
  to learn more about our motivation behind creating Kotlin Toolchain, 
  its use cases, the current state of the project, and its future.
-->
* Check out the [Kotlin Toolchain website](https://kotlin-toolchain.org) to read the guides and comprehensive docs.
