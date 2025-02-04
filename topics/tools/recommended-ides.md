[//]: # (title: Recommended IDEs and code editors)

## Android Studio IDE

[Android Studio](https://developer.android.com/studio) is the more stable solution,
although its support for Kotlin Multiplatform is still limited.

With Android Studio, you can also install a [Kotlin Multiplatform plugin](https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform-mobile)
that provides basic launching and debugging capabilities for iOS apps.

## JetBrains Fleet code editor

[JetBrains Fleet](https://www.jetbrains.com/fleet/) is a code editor for any language that can transform into a more powerful tool for development.
It is designed to be smart and focused on core development workflows (the edit-build-run loop),
AI-capable, and ready for remote development and collaboration.

Fleet supports testing and debugging your code on all target platforms. You can also navigate between Kotlin
Multiplatform code and code written in other languages interoperable with Kotlin.

See the [Use Fleet for Multiplatform development](fleet.md) tutorial to get started.

> Kotlin Multiplatform support in Fleet is still in the early stages of development. You may face issues and
> limitations, but we're committed to fixing them based on [your feedback](fleet.md#leave-feedback).
>
{style="note"}

## Other IDEs and code editors

If basic Kotlin Multiplatform support is enough for you, you can use any IDE that supports Kotlin, including [IntelliJ IDEA](https://www.jetbrains.com/idea/).
It supports opening, browsing, and building multiplatform projects.

> If you're targeting iOS in your Kotlin Multiplatform project, you'll also need [Xcode](https://developer.apple.com/xcode/)
> installed on your machine to write iOS-specific code and run iOS applications.
>
{style="note"}
