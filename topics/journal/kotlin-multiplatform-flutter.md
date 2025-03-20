# Kotlin Multiplatform and Flutter: cross-platform development solutions

[//]: # (description: This article explores Kotlin Multiplatform and Flutter, helping you to understand their capabilities and choose the right fit for your cross-platform project.) 

In the rapidly evolving world of technology, developers are constantly seeking efficient frameworks and tools to help them build high-quality applications. However, when choosing between available possibilities, it’s important to avoid placing too much emphasis on finding the so-called best option, as this approach might not always lead to the most suitable choice.

Each project is unique and has specific requirements. This article aims to help you navigate your choices and better understand which technology, such as Kotlin Multiplatform or Flutter, best fits your project, so you can make informed decisions.

## Cross-platform development: a unified approach to modern application building

Cross-platform development offers a way to build applications that run across multiple platforms with a single codebase, eliminating the need to rewrite the same functionality for each system. While often associated with [mobile development](cross-platform-mobile-development.md) – targeting both Android and iOS – this approach extends far beyond mobile, covering web, desktop, and even server-side environments.

The core idea is to maximize code reuse while ensuring platform-specific features can still be implemented when necessary, streamlining the development process and reducing maintenance efforts. Teams can speed up development cycles, reduce costs, and ensure consistency across platforms, making cross-platform development a smart choice in today's increasingly diverse application landscape.

## Kotlin Multiplatform and Flutter: streamlining development across platforms

Flutter and Kotlin Multiplatform are two popular cross-platform technologies that simplify the development of applications across different platforms.

### Flutter

[Flutter](https://flutter.dev/) is an open-source framework for building natively compiled, multiplatform applications from a single codebase. It allows you to create rich app experiences across Android, iOS, web, desktop (Windows, macOS, Linux), and embedded systems – all from a single, shared app codebase. Flutter apps are written using the Dart programming language. Flutter is supported and used by Google.

First introduced in 2014 under the name Sky, [Flutter 1.0](https://developers.googleblog.com/en/flutter-10-googles-portable-ui-toolkit/) was officially announced in December 2018 during Flutter Live.

The Flutter developer community is both large and highly active, providing continuous improvements and support. Flutter allows the use of shared packages contributed by developers within the Flutter and Dart ecosystems.

### Kotlin Multiplatform

[Kotlin Multiplatform](https://www.jetbrains.com/kotlin-multiplatform/) (KMP) is an open-source technology built by JetBrains that allows developers to create applications for Android, iOS, web, desktop (Windows, macOS, Linux), and the server side, enabling them to efficiently reuse Kotlin code across these platforms while retaining the benefits of native programming.

With Kotlin Multiplatform, you have various options: you can share all code except for app entry points, share a single piece of logic (like a network or a database module), or share business logic while keeping the UI native.

![Kotlin Multiplatform is a technology for reusing up to 100% of your code](kmp-logic-and-ui.svg){ width="700" }

Kotlin Multiplatform was first introduced as part of Kotlin 1.2 in 2017. In November 2023, Kotlin Multiplatform became [stable](https://blog.jetbrains.com/kotlin/2023/11/kotlin-multiplatform-stable/). During Google I/O 2024, Google announced its [support for Kotlin Multiplatform](https://android-developers.googleblog.com/2024/05/android-support-for-kotlin-multiplatform-to-share-business-logic-across-mobile-web-server-desktop.html) on Android for sharing business logic across Android and iOS.

[![Discover Kotlin Multiplatform](discover-kmp.svg){width="500"}](https://www.jetbrains.com/kotlin-multiplatform/)

#### Compose Multiplatform

You can write shared UI code across multiple platforms using [Compose Multiplatform](https://www.jetbrains.com/compose-multiplatform/), a modern declarative framework by JetBrains, which is built on Kotlin Multiplatform and Google’s Jetpack Compose.

Compose Multiplatform is currently stable on Android and desktop, in Beta on iOS, and in Alpha on web.

[![Explore Compose Multiplatform](explore-compose.svg){width="500"}](https://www.jetbrains.com/compose-multiplatform/)

Our dedicated article outlines the relationship between [Compose Multiplatform and Jetpack Compose](compose-multiplatform-and-jetpack-compose.md), highlighting the key differences.

### Kotlin Multiplatform and Flutter: an overview

<table style="both">
    <tr>
        <td></td>
        <td><b>Kotlin Multiplatform</b></td>
        <td><b>Flutter</b></td>
    </tr>
    <tr>
        <td><b>Created by</b></td>
        <td>JetBrains</td>
        <td>Google</td>
    </tr>
    <tr>
        <td><b>Language</b></td>
        <td>Kotlin</td>
        <td>Dart</td>
    </tr>
    <tr>
        <td><b>Flexibility and code reuse</b></td>
        <td>Share whichever part of your codebase you want to, including business logic and/or UI, from 1% to 100%.</td>
        <td>Сontrol every pixel of an application to create customized and adaptive designs with 100% code sharing across all platforms.</td>
    </tr>
    <tr>
        <td><b>Packages, dependencies, and ecosystem</b></td>
        <td>Packages are available from <a href="https://central.sonatype.com/">Maven Central</a> and other repositories, including
            <p><a href="http://klibs.io">klibs.io</a> (Alpha version), which is designed to simplify the search for KMP libraries.</p>
            <p>This <a href="https://github.com/terrakok/kmp-awesome">list</a> contains some of the most popular KMP libraries and tools.</p> </td>
        <td>Packages are available from <a href="https://pub.dev/">Pub.dev.</a></td>
    </tr>
    <tr>
        <td><b>Build tool</b></td>
        <td>Gradle (plus Xcode for applications targeting Apple devices).</td>
        <td>The Flutter command line tool (which uses Gradle and Xcode under the hood).</td>
    </tr>
    <tr>
        <td><b>Code sharing</b></td>
        <td>Android, iOS, web, desktop, and server-side.</td>
        <td>Android, iOS, web, desktop, and embedded devices.</td>
    </tr>
    <tr>
        <td><b>Compilation</b></td>
        <td>Compiles to JVM bytecode for desktop and Android, JavaScript or Wasm on the web, and platform-specific binaries for native platforms</td>
        <td>Debug builds run Dart code in a virtual machine.
        <p>Release builds output platform-specific binaries for native platforms, and JavaScript/Wasm for the web.</p>
        </td>
    </tr>
    <tr>
        <td><b>Communication with native APIs</b></td>
        <td>Native APIs are accessible directly from Kotlin code using <a href="https://kotlinlang.org/docs/multiplatform-expect-actual.html">expect/actual declarations.</a></td>
        <td>Communication with the host platform is possible using <a href="https://docs.flutter.dev/platform-integration/platform-channels">platform channels.</a></td>
    </tr>
    <tr>
        <td><b>UI rendering</b></td>
        <td><a href="https://www.jetbrains.com/compose-multiplatform/">Compose Multiplatform</a> can be used to share UI across platforms, based on Jetpack Compose by Google, using the Skia engine that is compatible with OpenGL, ANGLE (which translates OpenGL ES 2 or 3 calls to native APIs), Vulkan, and Metal.</td>
        <td>Flutter widgets are rendered on the screen using the custom <a href="https://docs.flutter.dev/perf/impeller">Impeller engine</a>, which talks straight to the GPU using Metal, Vulkan, or OpenGL depending on the platform and device.</td>
    </tr>
    <tr>
        <td><b>Iteration on UI development</b></td>
        <td>IntelliJ IDEA supports previews for desktop applications.
        <p>The team is working on a <a href="https://www.youtube.com/shorts/xgtHmB2J2f8">hot reload</a> feature that lets you see visual changes as you code, now available in IntelliJ IDEA. If you want to try Hot Reload for Compose Multiplatform in your project, go to its <a href="https://github.com/JetBrains/compose-hot-reload">GitHub repository</a> and follow the steps in the README.</p>
        <p>Android Studio offers preliminary support for <a href="https://developer.android.com/develop/ui/compose/tooling/previews">previews for Android</a>, plus <a href="https://developer.android.com/develop/ui/compose/tooling/iterative-development">Live Edit</a> for quick iteration on Android UI.</p></td>
        <td>IDE plugins are available for VS Code and Android Studio.</td>
    </tr>
    <tr>
        <td><b>Companies using the technology</b></td>
        <td><a href="https://www.forbes.com/sites/forbes-engineering/2023/11/13/forbes-mobile-app-shifts-to-kotlin-multiplatform/">Forbes</a>, <a href="https://www.youtube.com/watch?v=z-o9MqN86eE">Todoist</a>, <a href="https://medium.com/mcdonalds-technical-blog/mobile-multiplatform-development-at-mcdonalds-3b72c8d44ebc">McDonald’s</a>, <a href="https://www.youtube.com/watch?v=5sOXv-X43vc">Google Workspace</a>, <a href="https://www.youtube.com/watch?v=hZPL8QqiLi8">Philips</a>, <a href="https://raymondctc.medium.com/adopting-kotlin-multiplatform-mobile-kmm-on-9gag-app-dfe526d9ce04">9gag</a>, <a href="https://kotlinlang.org/lp/multiplatform/case-studies/baidu">Baidu</a>, <a href="https://kotlinlang.org/lp/multiplatform/case-studies/autodesk/">Autodesk</a>, <a href="https://touchlab.co/">TouchLab</a>, <a href="https://www.youtube.com/watch?v=YsQ-2lQYQ8M">Instabee</a>, and more are listed in our <a href="case-studies.topic">KMP case studies.</a></td>
        <td><a href="https://flutter.dev/showcase/xiaomi">Xiaomi</a>, <a href="https://flutter.dev/showcase/wolt">Wolt</a>, <a href="https://flutter.dev/showcase/universal-studios">Universal Studios</a>, <a href="https://flutter.dev/showcase/alibaba-group">Alibaba Group</a>, <a href="https://flutter.dev/showcase/bytedance">ByteDance</a>, <a href="https://www.geico.com/techblog/flutter-as-the-multi-channel-ux-framework/">Geico</a>, <a href="https://flutter.dev/showcase/ebay">eBay Motors</a>, <a href="https://flutter.dev/showcase/google-pay">Google Pay</a>, <a href="https://flutter.dev/showcase/so-vegan">So Vegan</a>, and more are listed on the <a href="https://flutter.dev/showcase">Flutter Showcase.</a></td>
    </tr>
</table>

[![Explore real-life use cases from global companies that leverage Kotlin Multiplatform for cross-platform development.](kmp-use-cases-1.svg){width="500"}](case-studies.topic)

You can also check out Google’s blog post, [Making Development Across Platforms Easier for Developers](https://developers.googleblog.com/en/making-development-across-platforms-easier-for-developers/), which provides guidance on choosing the right tech stack for your project.

If you're looking for an additional comparison between Kotlin Multiplatform and Flutter, you can also watch the [KMP vs. Flutter video](https://www.youtube.com/watch?v=dzog64ENKG0) by Philipp Lackner. In this video, he shares some interesting observations about these technologies, in terms of code sharing, UI rendering, performance, and the future of both technologies.

By carefully evaluating your specific business needs, objectives, and tasks, you can identify the cross-platform solution that best meets your requirements.




