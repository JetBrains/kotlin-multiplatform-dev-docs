[//]: # (title: Recommended IDEs and code editors)

## IntelliJ IDEA and Android Studio

[IntelliJ IDEA](https://www.jetbrains.com/idea/) provides full Kotlin Multiplatform support.
[Android Studio](https://developer.android.com/studio) is another stable solution for Kotlin Multiplatform.
Both are based on the IntelliJ platform, so generally they share the same features, although specific updates
may not arrive simultaneously.

Starting with IntelliJ IDEA 2025.2.2 or Android Studio Otter 2025.2.1,
you can install the [Kotlin Multiplatform IDE plugin](https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform)
that provides basic launching and debugging capabilities for iOS apps,
preflight environment checks, and other helpful KMP functionality.

Apart from base Kotlin Multiplatform features, the plugin also provides support for Compose Multiplatform
libraries, enabling more comfortable UI development:

* Quality-of-life automation for multiplatform resources.
* Support for the `@Preview` annotation which works in common Compose code.
* Support for [Compose Hot Reload](compose-hot-reload.md): automatic detection of hot reload run configurations,
  IDE integration with logs and settings for Compose Hot Reload,
  IDE actions and toolbars making the experience smoother in general.

## Xcode

If you're targeting iOS in your Kotlin Multiplatform project, you need [Xcode](https://developer.apple.com/xcode/)
installed on your machine to write iOS-specific code and run iOS applications.

To upload your apps to App Store Connect, build them with Xcode 16 or later.

## Other IDEs and code editors

If basic Kotlin Multiplatform support is enough for you, you can use any IDE that supports Kotlin.
