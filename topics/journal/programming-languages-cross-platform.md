# Popular programming languages for developing cross-platform applications

[//]: # (description: Explore key considerations in choosing a language for cross-platform development, comparisons of popular technologies, and real-life case studies.)

You’ve probably started to notice the term [cross-platform development](cross-platform-mobile-development.md) popping up more and more frequently these days. Indeed, cross-platform programming is becoming increasingly popular in the context of software development. It is particularly prevalent in the area of mobile apps, but its use is by no means limited to these types of applications. As businesses strive to reach broader audiences across multiple devices and operating systems, developers are turning to versatile languages and frameworks that erase platform barriers.

If you’re wondering which programming language would best equip you to get started with cross-platform development, this overview article will steer you in the right direction, offering insights and real use-case examples.

## Understanding cross-platform development

Cross-platform application development refers to the development approach where a single codebase can be used to create software that runs on multiple platforms, such as iOS, Android, Windows, macOS, and web browsers, among others. This approach has gained popularity in recent years, thanks in large part to the growing demand for mobile apps. Mobile engineers can share some or all of the source code between iOS and Android instead of developing separate applications for each platform.

We have a dedicated guide where you can read more about the benefits and limitations of both [native and cross-platform development](native-and-cross-platform.md) and how to choose between these two approaches. Some of the main advantages of cross-platform development include:

1. **Cost-effectiveness.** Building separate apps for each platform can be expensive in terms of both time and resources. With cross-platform development, developers can write code once and deploy it across multiple platforms, reducing development costs.

2. **Faster development.** This approach helps to accelerate the development process by making it so that developers only need to write and maintain a single codebase.

3. **Efficient and flexible code sharing.** Modern cross-platform technologies enable developers to reuse code across multiple platforms while maintaining the advantages of native programming.

4. **Consistent user experiences across platforms.** Cross-platform development ensures that key behaviors, such as calculations or workflows, deliver the same results on different platforms when needed. This helps maintain consistency, providing users with the same experience whether they are on iOS, Android, or other devices and operating systems.

In this article, we will discuss some of the most popular programming languages for cross-platform development.

## Popular cross-platform programming languages, frameworks, and technologies

This article focuses on well-established programming languages suited for cross-platform development. While there are many languages designed for various purposes, this section provides a concise overview of some of the most popular programming languages for cross-platform development along with relevant statistics and the frameworks that support them.

<table style="header-row">
    <tr>
        <td>Language</td>
        <td>First appeared</td>
        <td>Most popular technologies (<a href="https://survey.stackoverflow.co/2024/technology#most-popular-technologies">Stack Overflow, 2024</a>)</td>
        <td>Most popular technologies (<a href="https://www.jetbrains.com/lp/devecosystem-2024/">Developer Ecosystem Report 2024</a>)</td>
        <td>Ecosystem/tooling</td>
        <td>Technologies/frameworks</td>
    </tr>
    <tr>
        <td>JavaScript</td>
        <td>1995</td>
        <td>#1 (62.3%)</td>
        <td>#1 (61%)</td>
        <td>Rich ecosystem, many libraries, an active community</td>
        <td>React Native, Ionic</td>
    </tr>
    <tr>
        <td>Dart</td>
        <td>2011</td>
        <td>#17 (6%)</td>
        <td>#15 (8%)</td>
        <td>Growing ecosystem, supported by Google</td>
        <td>Flutter</td>
    </tr>
    <tr>
        <td>Kotlin</td>
        <td>2011</td>
        <td>#15 (9.04%)</td>
        <td>#13 (14%)</td>
        <td>Expanding ecosystem, first-class support for JetBrains tools</td>
        <td>Kotlin Multiplatform</td>
    </tr>
    <tr>
        <td>C#</td>
        <td>2000</td>
        <td>#8 (27.1%)</td>
        <td>9 (22%)</td>
        <td>Strong support from Microsoft, a large ecosystem</td>
        <td>.NET MAUI</td>
    </tr>
    <tr>
        <td>C++</td>
        <td>1985</td>
        <td>#9 (23%)</td>
        <td>8 (25%)</td>
        <td>Mature but fewer third-party libraries than other languages</td>
        <td>Qt</td>
    </tr>
</table>

**JavaScript**

JavaScript is a widely used programming language that allows users to implement complex functionalities on web pages. With the introduction of frameworks like React Native and Ionic, it has become a popular choice for cross-platform app development. According to the latest [Developer Ecosystem Survey](https://www.jetbrains.com/lp/devecosystem-2024/) conducted by JetBrains, 61% of developers use JavaScript, which makes it the most popular programming language.

**Dart**

Dart is an object-oriented, class-based programming language that was introduced by Google in 2011. Dart forms the foundation of Flutter, an open-source framework created by Google for building multiplatform applications from a single codebase. Dart provides the language and runtimes that power Flutter apps.

**Kotlin**

Kotlin is a modern, mature multiplatform programming language developed by JetBrains. According to the [Octoverse report](https://github.blog/news-insights/octoverse/octoverse-2024/#the-most-popular-programming-languages), it is the fifth-fastest-growing language in 2024. It's concise, safe, interoperable with Java and other languages, and is Google's preferred language for Android app development. 

[Kotlin Multiplatform (KMP)](https://www.jetbrains.com/kotlin-multiplatform/) is a technology by JetBrains that allows you to create applications for various platforms and reuse Kotlin code across them while retaining the benefits of native programming. Additionally, JetBrains provides Compose Multiplatform, a declarative framework for sharing UIs across multiple platforms that is based on KMP and Jetpack Compose. In May 2024, Google announced their official [support for Kotlin Multiplatform](https://android-developers.googleblog.com/2024/05/android-support-for-kotlin-multiplatform-to-share-business-logic-across-mobile-web-server-desktop.html) for sharing business logic across Android and iOS.

[![Discover Kotlin Multiplatform](discover-kmp.svg){width="500"}](https://www.jetbrains.com/kotlin-multiplatform/)

**C#**

C# is a cross-platform general purpose programming language developed by Microsoft. C# is the most popular language for the .NET Framework. .NET MAUI is a framework for building native, cross-platform desktop and mobile apps from a single C# codebase for Android, iOS, Mac, and Windows.

**C++**

C++ is a general purpose programming language that was first released in 1985 as an extension of the C programming language. Qt is a cross-platform software development framework that includes a set of modularized C++ library classes and provides a range of APIs for application development. Qt provides a C++ class library with application building blocks for C++ development.

## Key factors in selecting a cross-platform programming language

With all the languages, technologies, and tools available today, it can be overwhelming trying to choose the right one, especially if you are just stepping into the world of cross-platform development. Various cross-platform technologies have their own unique pros and cons, but ultimately, it all comes down to your goals and requirements for the software you want to build.

When choosing a language or framework for your project, you should keep several important factors in mind. These include the type of your application, its performance and UX requirements, associated tooling, and various other concerns that are described in detail below.

**1. The type of application**

Different programming languages and frameworks are better supported on different platforms such as Windows, macOS, Linux, iOS, Android, and web browsers. Certain languages are naturally more suitable for specific platforms and projects.

**2. Performance and UX requirements**

Certain types of applications have specific performance and user experience (UX) requirements, which can be measured by different criteria, like speed, responsiveness, memory usage, and their consumption of central processing units (CPUs) and graphics processing units (GPUs). Consider the functions that your future application will need to fulfill and your desired parameters for the above criteria.

> For example, a graphics-intensive game app might benefit from a language that can efficiently utilize the GPU. Meanwhile, a business app might prioritize ease of database integration and network communication.
>
{style="tip"}

**3. Existing skill set and learning curve**

When making a choice of a technology for their next project, a development team should take into account their previous experience. Introducing a new language or tool requires time for training, which sometimes can delay the project. The steeper the learning curve, the longer it will take for the team to become proficient.

> For example, if your team consists of highly proficient JavaScript developers and you lack the resources to adopt new technologies, it might be beneficial to select frameworks that utilize JavaScript, like React Native.
>
{style="tip"}

**4. Existing use cases**

Another important factor to consider is the real-world use of the technology. Reviewing case studies from companies that have successfully implemented specific cross-platform languages or frameworks can provide valuable insights into how these technologies perform in production. This can help you evaluate whether a particular technology is suitable for your project’s goals. For example, you can explore case studies of companies leveraging Kotlin Multiplatform to develop production-ready applications across various platforms.

![Kotlin Multiplatform Case Studies](kmp-case-studies.png){width="700"}

[![Explore Real-World Kotlin Multiplatform Use Cases](kmp-use-cases-1.svg){width="500"}](case-studies.topic)

**5. Language ecosystem**

Another important factor is the maturity of the language’s ecosystem. Pay attention to the availability and quality of tools and libraries that support multiplatform development. For example, JavaScript has a vast number of libraries, which support frontend frameworks (React, Angular, Vue.js), backend development (Express, NestJS), and a wide range of other functionalities.

Similarly, Flutter has a significant and rapidly growing number of libraries, also known as packages or plugins. Though Kotlin Multiplatform has fewer libraries currently, its ecosystem is growing rapidly and the language is being enhanced by many Kotlin developers across the world. The infographics below show how the number of Kotlin Multiplatform libraries has grown over the years.

![Kotlin Multiplatform Libraries Over Years](kmp-libs-over-years.png){width="700"}

**6. Popularity and community support**

It is worth looking closely at the popularity and community support of the programming language and associated technologies. It doesn’t come down only to the number of users and libraries. Pay attention to how active and supportive the language’s community is, including its users and contributors. Look for available blogs, podcasts, forums, and other resources.

## The future of cross-platform development

As cross-platform development continues to evolve, we can expect even greater efficiency, performance, and flexibility from the tools and languages supporting it. With growing demand for seamless user experiences across multiple devices, more companies are investing in frameworks that allow developers to share code without compromising on native performance. The future of cross-platform technologies looks promising, with advances likely to reduce limitations and further streamline the development process for a wide range of applications.

[![See Kotlin Multiplatform in Action](see-kmp-in-action.svg){width="500"}](https://www.jetbrains.com/kotlin-multiplatform/)
