[//]: # (title: FAQ)

## Kotlin Multiplatform

### What is Kotlin Multiplatform?

[Kotlin Multiplatform](https://www.jetbrains.com/kotlin-multiplatform/) (KMP) is an open-source technology by JetBrains for flexible cross-platform development. It allows you to
create applications for various platforms and efficiently reuse code across them while retaining the benefits of native
programming. With Kotlin Multiplatform, you can develop apps for Android, iOS, desktop, web, server-side, and other
platforms.

### Can I share UIs with Kotlin Multiplatform?

Yes, you can share UIs using [Compose Multiplatform](https://www.jetbrains.com/lp/compose-multiplatform/), JetBrains' declarative UI framework based on Kotlin
and [Jetpack Compose](https://developer.android.com/jetpack/compose). This framework allows you to create shared UI
components for platforms like iOS, Android, desktop, and web, helping you to maintain a consistent user interface across
different devices and platforms.

To learn more, see the [Compose Multiplatform](#compose-multiplatform) section.

### What platforms does Kotlin Multiplatform support?

Kotlin Multiplatform supports Android, iOS, desktop, web, server-side, and other platforms. Learn more
about [supported platforms](supported-platforms.md).

### In which IDE should I work on my cross-platform app?

We recommend using JetBrains Fleet code editor or Android Studio IDE, depending on your project needs and expectations.
Learn more about which one to choose in [Recommended IDEs and code editors](recommended-ides.md).

### How do I create a new Kotlin Multiplatform project?

The [Create a Kotlin Multiplatform app with shared logic and native UI](multiplatform-create-first-app.md) tutorial provides step-by-step
instructions for creating Kotlin Multiplatform projects. You can decide what to share – only logic or both logic and a
UI.

### I have an existing Android application. How can I migrate it to Kotlin Multiplatform?

The [Make your Android application work on iOS](multiplatform-integrate-in-existing-app.md) step-by-step tutorial
explains how to make your Android application work on iOS with a native UI.
If you also want to share the UI with Compose Multiplatform, see the [corresponding answer](#i-have-an-existing-android-application-that-uses-jetpack-compose-what-should-i-do-to-migrate-it-to-other-platforms).

### Where can I get complete examples to play with?

Here's a [list of real-life examples](multiplatform-samples.md).

### Where can I find a list of real-life Kotlin Multiplatform applications? What companies use KMP in production?

Check out our [list of case studies](case-studies.md) to learn from other companies that have already adopted Kotlin
Multiplatform in production.

### Which operating systems can work with Kotlin Multiplatform?

If you are going to work with shared code or platform-specific code, except for iOS, you can work on any operating
system supported by your IDE.

Learn more about [recommended IDEs](recommended-ides.md).

If you want to write iOS-specific code and run an iOS application on a simulator or real device, use a Mac with a macOS.
This is because iOS simulators can run only on macOS, per Apple requirements, but cannot run on other operating
systems, such as Microsoft Windows or Linux.

### How can I write concurrent code in Kotlin Multiplatform projects?

You can still use coroutines and flows to write asynchronous code in your Kotlin Multiplatform projects. How you call
this code depends on where you call the code from. Calling suspending functions and flows from Kotlin code is widely
documented, especially for Android. [Calling them from Swift code](https://kotlinlang.org/docs/native-ios-integration.html#calling-kotlin-suspending-functions)
requires a little more work, see [KT-47610](https://youtrack.jetbrains.com/issue/KT-47610) for more details.

The best current approach for calling suspending functions and flows from Swift is to use plugins and libraries like
[KMP-NativeCoroutines](https://github.com/rickclephas/KMP-NativeCoroutines) or [SKIE](https://skie.touchlab.co/) together
with Swift's `async`/`await` or libraries like Combine and RxSwift. At the moment, KMP-NativeCoroutines is the more
tried-and-tested solution, and it supports `async`/`await`, Combine, and RxSwift approaches to concurrency. SKIE is easier
to set up and less verbose. For instance, it maps Kotlin `Flow` to Swift `AsyncSequence` directly. Both of these libraries
support the proper cancellation of coroutines.

To learn how to use them, see [Share more logic between iOS and Android](multiplatform-upgrade-app.md).

### What is Kotlin/Native, and how does it relate to Kotlin Multiplatform?

[Kotlin/Native](https://kotlinlang.org/docs/native-overview.html) is a technology for compiling Kotlin code to native
binaries, which can run without a virtual machine. It includes an [LLVM-based](https://llvm.org/) backend for the Kotlin
compiler and a native implementation of the Kotlin standard library.

Kotlin/Native is primarily designed to allow compilation for platforms where virtual machines are not desirable or
possible, such as embedded devices and iOS. It is particularly suitable when you need to produce a self-contained
program that does not require an additional runtime or virtual machine.

For example, in mobile applications, shared code written in Kotlin is compiled to JVM bytecode for Android with
Kotlin/JVM and compiled to native binaries for iOS with Kotlin/Native. It makes the integration with Kotlin Multiplatform
seamless on both platforms.

![Kotlin/Native and Kotlin/JVM binaries](kotlin-native-and-jvm-binaries.png){width=350}

### How can I speed up my Kotlin Multiplatform module compilation for native platforms (iOS, macOS, Linux)?

See these [tips for improving Kotlin/Native compilation times](https://kotlinlang.org/docs/native-improving-compilation-time.html).

## Compose Multiplatform

### What is Compose Multiplatform?

[Compose Multiplatform](https://www.jetbrains.com/lp/compose-multiplatform/) is a modern declarative and reactive UI framework developed by JetBrains that provides a simple
way to build user interfaces with a small amount of Kotlin code. It also allows you to write your UI once and run it on
any of the supported platforms – iOS, Android, desktop (Windows, macOS, Linux), and web.

### How does it relate to Jetpack Compose for Android?

Compose Multiplatform shares most of its API with [Jetpack Compose](https://developer.android.com/jetpack/compose), the
Android UI framework developed by Google. In fact, when you are using Compose Multiplatform to target Android, your app
simply runs on Jetpack Compose.
Other platforms targeted by Compose Multiplatform may have implementation details under the hood that differ from those
of Jetpack Compose on Android, but they still provide you with the same APIs.

### Between which platforms can I share my UI?

We want you to have the option to share your UI between any combination of popular platforms – Android, iOS, desktop (
Linux, macOS, Windows), and web (based on Wasm). However, Compose Multiplatform is only Stable for Android
and desktop at the moment. For more details, see [Supported platforms](supported-platforms.md).

### Can I use Compose Multiplatform in production?

The Android and desktop targets of Compose Multiplatform are Stable. You can use them in production.

The iOS target is in Beta, which means that it's feature complete. You can use it in production
and expect minimal migration issues, but watch out for changes and deprecation warnings.

The version of Compose Multiplatform for Web that is based on WebAssembly is in Alpha, which means that it's in active development.
You can use it with caution and expect migration issues.
It has the same UI as Compose Multiplatform for iOS, Android, and Desktop.

### How do I create a new Compose Multiplatform project?

The [Create a Compose Multiplatform app with shared logic and UI](compose-multiplatform-create-first-app.md) tutorial provides step-by-step
instructions for creating a Kotlin Multiplatform project with Compose Multiplatform for Android, iOS, and desktop.
You can also watch a [video tutorial](https://www.youtube.com/watch?v=5_W5YKPShZ4) on YouTube created by Kotlin
Developer Advocate Sebastian Aigner.

### What IDE should I use for building apps with Compose Multiplatform?

We recommend using JetBrains Fleet code editor or Android Studio IDE, depending on your project needs and expectations. To try
a new multiplatform experience without juggling different IDEs and switching to Xcode for writing Swift code,
try out [JetBrains Fleet tutorial](fleet.md).

For more details on which one to choose, see [Recommended IDEs and code editors](recommended-ides.md).

### Can I play with a demo application? Where can I find it?

You can play with our [samples](multiplatform-samples.md).

### Does Compose Multiplatform come with widgets?

Yes, Compose Multiplatform provides full support for [Material 3](https://m3.material.io/) widgets.

### To what extent can I customize the appearance of Material widgets?

You can use Material's theming capabilities to customize colors, fonts, and paddings. If you want to create a unique
design, you can create custom widgets and layouts.

### Can I share the UI in my existing Kotlin Multiplatform app?

If your application uses a native API for its UI (which is the most common case), you can gradually rewrite some
parts to Compose Multiplatform, as it provides interoperability for that. You can replace native UIs
with a special interop view that wraps a common UI written with Compose.

### I have an existing Android application that uses Jetpack Compose. What should I do to migrate it to other platforms?

Migration of the app consists of two parts: migrating the UI and migrating the logic. The complexity of the migration
depends on the complexity of your application and the amount of Android-specific libraries you use.
You can migrate most of your screens to Compose Multiplatform without changes. All of the Jetpack Compose widgets are
supported. However, some APIs work only in the Android target – they might be Android-specific or have yet to be ported to
other platforms. For instance, resource handling is Android-specific, so you would need to migrate to the [Compose
Multiplatform resource library](compose-images-resources.md) or use a community solution. The
Android [Navigation library](https://developer.android.com/jetpack/androidx/releases/navigation) is also
Android-specific, but there are [community alternatives](compose-navigation-routing.md) available. For more information on components available only for Android, see the
current [list of Android-only APIs](compose-android-only-components.md).

You need to [migrate the business logic to Kotlin Multiplatform](multiplatform-integrate-in-existing-app.md). When you
try to move your code to shared modules, the parts that use Android dependencies stop compiling, and you need to rewrite
them.

* You can rewrite the code that uses Android-only dependencies to use multiplatform libraries instead. Some libraries
  might already support Kotlin Multiplatform, so no changes are needed. You can check
  the [KMP-awesome](https://github.com/terrakok/kmp-awesome) library list.
* Alternatively, you can separate common code from platform-specific logic
  and [provide common interfaces](multiplatform-connect-to-apis.md) that are implemented differently depending on the
  platform. On Android, the implementation can use your existing functionality, and on other platforms, such as iOS, you
  need to provide new implementations for the common interfaces.

### Can I integrate Compose screens into an existing iOS app?

Yes. Compose Multiplatform supports different integration scenarios. For more information on integration with iOS UI frameworks,
see [Integration with SwiftUI](compose-swiftui-integration.md) and [Integration with UIKit](compose-uikit-integration.md).

### Can I integrate UIKit or SwiftUI components into a Compose screen?

Yes, you can. See [Integration with SwiftUI](compose-swiftui-integration.md) and [Integration with UIKit](compose-uikit-integration.md).

<!-- Need to revise
### What happens when my mobile OS updates and introduces new platform capabilities?

You can use them in platform-specific parts of your codebase once Kotlin supports them. We do our best to support them
in the upcoming Kotlin version. All new Android capabilities provide Kotlin or Java APIs, and wrappers over iOS APIs are
generated automatically.
-->

### What happens when my mobile OS updates and changes the visual style of the system components or their behavior?

Your UI will stay the same after the OS updates because all of the components are drawn on a canvas. If you embed native
iOS components into your screen, updates may affect their appearance.

## Future plans

### What are the plans for the Kotlin Multiplatform evolution?

We at JetBrains are investing much to provide the best experience for multiplatform development and eliminate existing pains of multiplatform users.
We have plans for improving the core Kotlin Multiplatform technology, integration with the Apple ecosystem, tooling, and our Compose Multiplatform UI framework. 
[Check out our roadmap](https://blog.jetbrains.com/kotlin/2023/11/kotlin-multiplatform-development-roadmap-for-2024/).

### When will Compose Multiplatform become Stable?

Compose Multiplatform is Stable for Android and desktop, while the iOS platform support is in Beta, and web is in Alpha. We are working towards a stable release for both iOS and web platforms, with exact dates to be announced.

For more information on stability statuses, see [Supported platforms](supported-platforms.md).

### What about future support for web targets in Kotlin and Compose Multiplatform?

We're currently focusing resources on WebAssembly (Wasm), which shows great potential. You can experiment with our new
[Kotlin/Wasm backend](https://kotlinlang.org/docs/wasm-overview.html) and [Compose Multiplatform for Web](https://kotl.in/wasm-compose-example) powered by Wasm.

As for the JS target, the Kotlin/JS backend has already reached Stable status. In Compose Multiplatform, due to resource
constraints, we've shifted our focus from JS Canvas to Wasm, which we believe holds more promise.

We also offer Compose HTML, previously known as Compose Multiplatform for web. It's an additional library designed for
working with the DOM in Kotlin/JS, and it's not intended for sharing UIs across platforms.

### Are there any plans to improve tooling for multiplatform development?

Yes, we're acutely aware of the current challenges with multiplatform tooling and are actively working on enhancements
in several areas.

We've already launched previews of the [Kotlin Multiplatform support in Fleet](fleet.md).
Try it yourself and leave feedback!

### Are you going to provide a Swift interop?

Yes. We're currently investigating various approaches to provide direct interoperability with Swift, with a focus on
exporting Kotlin code to Swift.
