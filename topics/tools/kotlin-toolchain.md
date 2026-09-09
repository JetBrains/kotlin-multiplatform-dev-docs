[//]: # (title: Project configuration with Kotlin Toolchain)

[Kotlin Toolchain](https://kotlin-toolchain.org/) is a tool created by JetBrains to help you configure projects
for building, packaging, publishing, and more. With Kotlin Toolchain, you can spend less time dealing with build systems and focus
on addressing real business challenges instead.

Kotlin Toolchain is designed to support Kotlin Multiplatform applications as well, aware of all platforms supported by the framework.

> Kotlin Toolchain is in [Alpha](supported-platforms.md#general-kotlin-stability-levels).
> You're welcome to try it in your Kotlin Multiplatform projects.
> We would appreciate your feedback in [YouTrack](https://youtrack.jetbrains.com/issues/AMPER).
>
{style="note"}

## Installing Kotlin Toolchain

You can use Kotlin Toolchain both as a standalone CLI application and as an IDE plugin for IntelliJ IDEA or Android Studio.

While Kotlin Toolchain CLI can be used without an IDE,
the plugin provides additional tooling and diagnostics that make it easier to work with Kotlin Toolchain projects.

### CLI {id="toolchain-script-install"}

You can install the Kotlin Toolchain CLI using [SDKMAN!](https://sdkman.io/):

```shell
sdk install kotlintoolchain
```

Or via the installer script:

<tabs>
<tab title="macOS or Linux">

```shell
curl -fsSL https://kotl.in/install.sh | sh
```

</tab>

<tab title="Windows">

```shell
powershell -ExecutionPolicy ByPass -c "irm 'https://kotl.in/install.ps1' | iex"
```

</tab>
</tabs>

This adds `kotlin` as a command-line tool.

### IDE plugin

Install the [Kotlin Toolchain plugin](https://plugins.jetbrains.com/plugin/31850-kotlin-toolchain/)
to enable inspections, navigation, and completion for Kotlin Toolchain projects.

## How Kotlin Toolchain works

Kotlin Toolchain provides a declarative build configuration format based on YAML files.
A `project.yaml` file configures the project as a whole and catalogs modules,
and `module.yaml` files configure individual modules (libraries or applications produced by the project).

The Kotlin Toolchain DSL offers a rich syntax
enabling you to work with multiplatform configurations, including dependencies, settings, and so on.

Here is an example of a Kotlin module file for a Kotlin Multiplatform shared library that can be used with JVM,
Android, and iOS applications:

```yaml
# Defines the project type and the list of targeted platforms
product:
  type: kmp/lib
  platforms: [ android, iosArm64, iosSimulatorArm64, jvm ]

# Adds Maven or SwiftPM dependencies,
# with the potential to support other package managers
dependencies:
  - $compose.runtime
  - $compose.foundation
  - $compose.material3
  - $compose.ui
  - $compose.components.resources
  - $compose.preview
  - $libs.androidx.lifecycle.viewmodel.compose
  - $libs.androidx.lifecycle.runtime.compose

# Lists Android-specific dependencies
dependencies@android:
  - $compose.uiTooling

# Specifies project-specific configuration of Kotlin Toolchain
settings:
  # Settings specific to Compose Multiplatform
  compose:
    enabled: true
    resources:
      packageName: org.example.project.resources
      # Generate public accessors for Compose Multiplatform resources
      exposedAccessors: true
```

## What's next

* Check out the [get started guide](kotlin-toolchain-get-started.md)
  to learn how to set up a basic KMP project with Kotlin Toolchain.  
* For more on what Kotlin Toolchain is and what purpose it serves, check out the [product FAQ](https://kotlin-toolchain.org/dev/faq/).
* For a deep dive into Kotlin Toolchain, check out the [documentation on the official website](https://kotlin-toolchain.org/%kotlinToolchainVersion%/getting-started/).