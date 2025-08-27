# Kotlin Multiplatform vs. React Native: A cross-platform comparison

<web-summary>Explore how Kotlin Multiplatform with Compose Multiplatform compares to React Native in code sharing, ecosystem, and UI rendering – and see which tool stack fits your team best.</web-summary>

<include-in-head>

<![CDATA[
    <script type="application/ld+json">
    {
      "@context": "https://schema.org",
      "@type": "FAQPage",
      "mainEntity": [{
        "@type": "Question",
        "name": "Is Kotlin Multiplatform production-ready?",
        "acceptedAnswer": {
          "@type": "Answer",
          "text": "<p>Kotlin Multiplatform is a stable technology that is ready for use in production. This means that you can use Kotlin Multiplatform for sharing code in production across Android, iOS, desktop (JVM), server-side (JVM), and web, even in the most conservative usage scenarios. Compose Multiplatform, a framework for building shared UIs across platforms (powered by Kotlin Multiplatform and Google’s Jetpack Compose), is stable for iOS, Android, and desktop. Web support is currently in Alpha.</p>"
        }
      }, {
        "@type": "Question",
        "name": "Is Kotlin Multiplatform better than React Native?",
        "acceptedAnswer": {
          "@type": "Answer",
          "text": "Both Kotlin Multiplatform and React Native have their own strengths, and the choice depends on your project’s specific goals, technical requirements, and team expertise. In the comparison above, we’ve outlined key differences in areas like code sharing, build tools, compilation, and ecosystem to help you decide which option is the best fit for your use case."
        } 
      }, {
        "@type": "Question",
        "name": "Does Google support Kotlin Multiplatform?",
        "acceptedAnswer": {
          "@type": "Answer",
          "text": "At Google I/O 2024, Google announced official support for using Kotlin Multiplatform on Android to share business logic between Android and iOS."
        }
      }, {
        "@type": "Question",
        "name": "Is it worth learning Kotlin Multiplatform?",
        "acceptedAnswer": {
          "@type": "Answer",
          "text": "If you're interested in sharing code across Android, iOS, desktop, and web while retaining native performance and flexibility, Kotlin Multiplatform is worth learning. It's backed by JetBrains and officially supported by Google on Android to share business logic between Android and iOS. What’s more, KMP with Compose Multiplatform is increasingly adopted in production by companies building multiplatform apps."
        }
      }]
    }
    </script>
]]>
</include-in-head>

> This comparison article highlights that Kotlin Multiplatform excels in delivering truly native experiences across Android and iOS with full access to platform APIs. KMP is particularly attractive for teams focused on performance, maintainability, and a native look and feel, especially when using Compose Multiplatform for shared UI code. Meanwhile, React Native may suit teams with JavaScript expertise, especially for rapid prototyping.
> 
{style="tip"}

Cross-platform development has significantly changed the way teams build applications, enabling them to deliver apps for multiple platforms from a shared codebase. This approach streamlines development and helps ensure a more consistent user experience across devices.

Previously, building for Android and iOS meant maintaining two separate codebases, often by different teams, which led to duplicated effort and noticeable differences between platforms. Cross-platform solutions have accelerated time to market and improved overall efficiency.

Among the available tools, Kotlin Multiplatform, React Native, and Flutter stand out as three of the most widely adopted options. In this article, we’ll take a closer look at both to help you choose the right fit for your product and team.

## Kotlin Multiplatform and Compose Multiplatform

[Kotlin Multiplatform (KMP)](https://www.jetbrains.com/kotlin-multiplatform/) is an open-source technology developed by JetBrains that enables code sharing across Android, iOS, desktop (Windows, macOS, Linux), web, and backend. It allows developers to reuse Kotlin across multiple environments while keeping native capabilities and performance.

Adoption is rising steadily: Kotlin Multiplatform usage more than doubled in just a year among respondents to the last two [Developer Ecosystem surveys](https://www.jetbrains.com/lp/devecosystem-2024/) – increasing from 7% in 2024 to 18% in 2025 – a clear sign of its growing momentum.

![KMP usage increased from 7% in 2024 to 18% in 2025 among respondents to the last two Developer Ecosystem surveys](kmp-growth-deveco.svg){width="700"}

[![Discover Kotlin Multiplatform](discover-kmp.svg){width="500"}](https://www.jetbrains.com/kotlin-multiplatform/)

With KMP, you can choose your sharing strategy: from sharing all code except for app entry points, to sharing a single piece of logic (like a network or a database module), or sharing business logic while keeping the UI native.

To share UI code across platforms, you can use [Compose Multiplatform](https://blog.jetbrains.com/kotlin/2023/11/kotlin-multiplatform-stable/) – JetBrains’ modern declarative framework built on Kotlin Multiplatform and Google’s Jetpack Compose. It is [stable for iOS](https://blog.jetbrains.com/kotlin/2023/11/kotlin-multiplatform-stable/?_gl=1*dcswc7*_gcl_au*MTE5NzY3MzgyLjE3NDk3MDk0NjI.*FPAU*MTE5NzY3MzgyLjE3NDk3MDk0NjI.*_ga*MTM4NjAyOTM0NS4xNzM2ODUwMzA5*_ga_9J976DJZ68*czE3NTA2NzU0MzQkbzM2JGcxJHQxNzUwNjc1NjEwJGo2MCRsMCRoMA..), Android, and desktop, with web support currently in Alpha.

[![Explore Compose Multiplatform](explore-compose.svg){width="500"}](https://www.jetbrains.com/compose-multiplatform/)

Originally introduced in Kotlin 1.2 (2017), Kotlin Multiplatform reached [stable status](https://blog.jetbrains.com/kotlin/2023/11/kotlin-multiplatform-stable/) in November 2023. At Google I/O 2024, Google announced [official support for using Kotlin Multiplatform on Android](https://android-developers.googleblog.com/2024/05/android-support-for-kotlin-multiplatform-to-share-business-logic-across-mobile-web-server-desktop.html) to share business logic between Android and iOS.

## React Native

React Native is an open-source framework for building Android and iOS applications using [React](https://reactjs.org/), a library for web and native user interfaces, and the app platform’s native capabilities. React Native allows developers to use JavaScript to access their platform’s APIs and describe the appearance and behavior of the UI using React components – bundles of reusable, nestable code.

React Native was first announced in January 2015 at React.js Conf. Later that year, Meta released React Native at F8 2015 and has been maintaining it ever since.

While Meta oversees the React Native product, the [React Native ecosystem](https://github.com/facebook/react-native/blob/HEAD/ECOSYSTEM.md) consists of partners, core contributors, and an active community. Today, the framework is supported by contributions from individuals and companies around the globe.

## Kotlin Multiplatform vs. React Native: Side-by-side comparison

<table style="both">
<tr>
        <td></td>
        <td><b>Kotlin Multiplatform</b></td>
        <td><b>React Native</b></td>
    </tr>
    <tr>
        <td><b>Created by</b></td>
        <td>JetBrains</td>
        <td>Meta</td>
    </tr>
    <tr>
        <td><b>Language</b></td>
        <td>Kotlin</td>
        <td>JavaScript, TypeScript</td>
    </tr>
<tr>
        <td><b>Flexibility and code reuse</b></td>
        <td>Share whichever part of your codebase you want, including business logic and/or UI, from 1% to 100%. Adopt it incrementally or use it from the ground up to build native-feeling apps across platforms.</td>
        <td>Reuse business logic and UI components across platforms, from individual features to full apps. Add React Native to existing native applications to build new screens or user flows.</td>
    </tr>
    <tr>
        <td><b>Packages, dependencies, and ecosystem</b></td>
        <td>Packages are available from <a href="https://central.sonatype.com/">Maven Central</a> and other repositories, including
            <p><a href="http://klibs.io">klibs.io</a> (Alpha version), which is designed to simplify the search for KMP libraries.</p>
            <p>This <a href="https://github.com/terrakok/kmp-awesome">list</a> contains some of the most popular KMP libraries and tools.</p></td>
        <td><a href="https://reactnative.dev/docs/libraries">React Native libraries</a> are typically installed from the <a href="https://www.npmjs.com/">npm registry</a> using a Node.js package manager, such as <a href="https://docs.npmjs.com/cli/npm">npm CLI</a> or <a href="https://classic.yarnpkg.com/en/">Yarn Classic</a>.</td>
    </tr>
<tr>
        <td><b>Build tool</b></td>
        <td>Gradle (plus Xcode for applications targeting Apple devices).</td>
        <td>React Native command-line tool and <a href="https://metrobundler.dev/">Metro bundler</a>, which invoke Gradle for Android and the Xcode build system for iOS under the hood.</td>
    </tr>
    <tr>
        <td><b>Target Environments</b></td>
        <td>Android, iOS, web, desktop, and server-side.</td>
        <td>Android, iOS, web, and desktop.
            <p>Support for web and desktop is provided through community and partner-led projects such as <a href="https://github.com/necolas/react-native-web">React Native Web</a>, <a href="https://github.com/microsoft/react-native-windows">React Native Windows</a>, and <a href="https://github.com/microsoft/react-native-macos">React Native macOS</a>.</p></td>
    </tr>
    <tr>
        <td><b>Compilation</b></td>
        <td>Compiles to JVM bytecode for desktop and Android, JavaScript or Wasm on the web, and platform-specific binaries for native platforms</td>
        <td>React Native uses Metro to build JavaScript code and assets. 
            <p>React Native comes with a bundled version of <a href="https://reactnative.dev/docs/hermes">Hermes</a>, which compiles JavaScript to Hermes bytecode during build. React Native also supports using JavaScriptCore as the <a href="https://reactnative.dev/docs/javascript-environment">JavaScript engine</a>.</p>
            <p>Native code is compiled by Gradle on Android and Xcode on iOS.</p></td>
    </tr>
    <tr>
        <td><b>Communication with native APIs</b></td>
        <td>Native APIs are accessible directly from Kotlin code thanks to Kotlin’s interoperability with both Swift/Objective-C and JavaScript.</td>
        <td>React Native exposes a set of APIs to connect your native code to your JavaScript application code: Native Modules and Native Components. The New Architecture uses <a href="https://github.com/reactwg/react-native-new-architecture/blob/main/docs/turbo-modules.md">Turbo Native Module</a> and <a href="https://github.com/reactwg/react-native-new-architecture/blob/main/docs/fabric-native-components.md">Fabric Native Components</a> to achieve similar results.</td>
    </tr>
<tr>
        <td><b>UI rendering</b></td>
        <td><a href="https://www.jetbrains.com/compose-multiplatform/">Compose Multiplatform</a> can be used to share UI across platforms, based on Jetpack Compose by Google, using the Skia engine that is compatible with OpenGL, ANGLE (which translates OpenGL ES 2 or 3 calls to native APIs), Vulkan, and Metal.</td>
        <td>React Native includes a core set of platform-agnostic native components, such as `View`, `Text`, and `Image`, that map directly to the platform's native UI building blocks, like `UIView` on iOS and `android.view` on Android.</td>
    </tr>
    <tr>
        <td><b>Iteration on UI development</b></td>
        <td>UI previews are available even from common code.
            <p>With <a href="compose-hot-reload.md">Compose Hot Reload</a>, you can instantly see UI changes without restarting your app or losing its state.</p></td>
        <td><a href="https://reactnative.dev/docs/fast-refresh">Fast Refresh</a> is a React Native feature that allows you to get near-instant feedback for changes in your React components.</td>
    </tr>
    <tr>
        <td><b>Companies using the technology</b></td>
        <td><a href="https://www.forbes.com/sites/forbes-engineering/2023/11/13/forbes-mobile-app-shifts-to-kotlin-multiplatform/">Forbes</a>, <a href="https://www.youtube.com/watch?v=z-o9MqN86eE">Todoist</a>, <a href="https://medium.com/mcdonalds-technical-blog/mobile-multiplatform-development-at-mcdonalds-3b72c8d44ebc">McDonald’s</a>, <a href="https://youtu.be/5lkZj4v4-ks?si=DoW00DU7CYkaMmKc">Google Workspace</a>, <a href="https://www.youtube.com/watch?v=hZPL8QqiLi8">Philips</a>, <a href="https://raymondctc.medium.com/adopting-kotlin-multiplatform-mobile-kmm-on-9gag-app-dfe526d9ce04">9gag</a>, <a href="https://kotlinlang.org/lp/multiplatform/case-studies/baidu">Baidu</a>, <a href="https://kotlinlang.org/lp/multiplatform/case-studies/autodesk/">Autodesk</a>, <a href="https://touchlab.co/">TouchLab</a>, <a href="https://www.youtube.com/watch?v=YsQ-2lQYQ8M">Instabee</a>, and more are listed in our <a href="case-studies.topic">KMP case studies.</a></td>
        <td>Facebook, <a href="https://engineering.fb.com/2024/10/02/android/react-at-meta-connect-2024/">Instagram</a>, <a href="https://devblogs.microsoft.com/react-native/">Microsoft Office</a>, <a href="https://devblogs.microsoft.com/react-native/">Microsoft Outlook</a>, Amazon Shopping, <a href="https://medium.com/mercari-engineering/why-we-decided-to-rewrite-our-ios-android-apps-from-scratch-in-react-native-9f1737558299">Mercari</a>, Tableau, <a href="https://github.com/wordpress-mobile/gutenberg-mobile">WordPress</a>, <a href="https://nearform.com/work/puma-scaling-across-the-globe/">Puma</a>, the PlayStation App, and more are listed on the <a href="https://reactnative.dev/showcase">React Native Showcase</a>.</td>
    </tr>
</table>

You can also take a look at the comparison of [Kotlin Multiplatform and Flutter](kotlin-multiplatform-flutter.md).

## Choosing the right cross-platform technology for your project

Deciding on a cross-platform framework isn’t about finding a one-size-fits-all solution – it’s about choosing the best fit for your project’s goals, technical requirements, and team expertise. Whether you're building a feature-rich product with a complex UI or aiming to launch quickly with existing skills, the right choice will depend on your specific priorities. Consider how much control you need over UI customization, how important long-term stability is, and what platforms you plan to support.

A team experienced with JavaScript may find React Native to be a practical choice, especially for rapid prototyping. On the other hand, Kotlin Multiplatform offers a different level of integration: It produces fully native Android apps and compiles to native binaries on iOS, with seamless access to native APIs. UIs can be fully native or shared via Compose Multiplatform, which renders beautifully using a high-performance graphics engine. This makes KMP particularly appealing for teams who prioritize native look and feel, maintainability, and performance while still benefiting from code sharing.

You can find more guidance in our detailed article on how to choose the right [cross-platform development frameworks](cross-platform-frameworks.md) for your next project.

## Frequently Asked Questions

**Q: Is Kotlin Multiplatform production-ready?**

A: Kotlin Multiplatform is a [stable technology that is ready for use in production](https://blog.jetbrains.com/kotlin/2023/11/kotlin-multiplatform-stable/). This means that you can use Kotlin Multiplatform for sharing code in production across Android, iOS, desktop (JVM), server-side (JVM), and web, even in the most conservative usage scenarios.

Compose Multiplatform, a framework for building shared UIs across platforms (powered by Kotlin Multiplatform and Google’s Jetpack Compose), is stable for iOS, Android, and desktop. Web support is currently in Alpha.

**Q: Is Kotlin Multiplatform better than React Native?** 

A: Both Kotlin Multiplatform and React Native have their own strengths, and the choice depends on your project’s specific goals, technical requirements, and team expertise. In the comparison above, we’ve outlined key differences in areas like code sharing, build tools, compilation, and ecosystem to help you decide which option is the best fit for your use case.

**Q: Does Google support Kotlin Multiplatform?**

A: At Google I/O 2024, Google announced [official support for using Kotlin Multiplatform](https://android-developers.googleblog.com/2024/05/android-support-for-kotlin-multiplatform-to-share-business-logic-across-mobile-web-server-desktop.html) on Android to share business logic between Android and iOS.

**Q: Is it worth learning Kotlin Multiplatform?**

A: If you're interested in sharing code across Android, iOS, desktop, and web while retaining native performance and flexibility, Kotlin Multiplatform is worth learning. It's backed by JetBrains and officially supported by Google on Android to share business logic between Android and iOS. What’s more, KMP with Compose Multiplatform is increasingly adopted in production by companies building multiplatform apps.
