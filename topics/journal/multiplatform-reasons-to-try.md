[//]: # (title: Ten reasons to adopt Kotlin Multiplatform and supercharge your projects)

[//]: # (description: Discover ten reasons why you should use Kotlin Multiplatform in your projects. See real-life examples from companies and start using this technology in your multiplatform development.)

In today's diverse tech landscape,
developers face the challenge of building applications that can seamlessly operate across various platforms, 
while optimizing development time and increasing user productivity. 
Kotlin Multiplatform (KMP) offers a solution that allows you to create apps for multiple platforms, 
facilitating code reuse across them while maintaining the advantages of native programming.

In this article, we'll explore ten reasons why developers should consider using 
Kotlin Multiplatform in their existing or new projects and why KMP continues to gain a lot of traction.

## Why you should try Kotlin Multiplatform in your projects

Whether you're looking to make your development more efficient or explore new technologies,
you will find this article helpful.
It explains some of the practical benefits of Kotlin Multiplatform, 
such as streamlining development, supporting multiple platforms, and offering a robust tooling ecosystem.
You'll also find case studies from real companies.

1. [Kotlin Multiplatform allows you to avoid code duplication](#1-kotlin-multiplatform-allows-you-to-avoid-code-duplication)
2. [Kotlin Multiplatform supports an extensive list of platforms](#2-kotlin-multiplatform-supports-an-extensive-list-of-platforms)
3. [Kotlin provides simplified code-sharing mechanisms](#3-kotlin-provides-simplified-code-sharing-mechanisms)
4. [Kotlin Multiplatform allows for flexible multiplatform development](#4-kotlin-multiplatform-allows-for-flexible-multiplatform-development)
5. [With the Kotlin Multiplatform solution, you can share UI code](#5-with-the-kotlin-multiplatform-solution-you-can-share-ui-code)
6. [You can use Kotlin Multiplatform in existing and new projects](#6-you-can-use-kotlin-multiplatform-in-existing-and-new-projects)
7. [With Kotlin Multiplatform, you can start sharing your code gradually](#7-with-kotlin-multiplatform-you-can-start-sharing-your-code-gradually)
8. [Kotlin Multiplatform is already used by global companies](#8-kotlin-multiplatform-is-already-used-by-global-companies)
9. [Kotlin Multiplatform provides powerful tooling support](#9-kotlin-multiplatform-provides-powerful-tooling-support)
10. [Kotlin Multiplatform boasts a large and supportive community](#10-kotlin-multiplatform-boasts-a-large-and-supportive-community)

### 1. Kotlin Multiplatform allows you to avoid code duplication

Baidu, the largest Chinese language search engine, launched _Wonder App_, an application targeted at a younger audience. 
Here are some of the problems they faced with traditional app development:

* Inconsistencies in the app experience: The Android app worked differently from the iOS app.
* High cost of verifying the business logic: The work of the iOS and Android developers using the same business logic 
  needed to be independently checked, which led to high costs.
* High upgrade and maintenance costs: Duplicating the business logic was complicated and time-consuming, 
  which increased the app's upgrade and maintenance costs.

The Baidu team decided to experiment with Kotlin Multiplatform, starting out by unifying the data layer: 
data model, RESTful API requests, JSON data parsing, and caching logic.

Then they decided to adopt the Model-View-Intent (MVI) user interface pattern, 
which allows you to unify the interface logic with Kotlin Multiplatform. 
They also shared low-level data, the processing logic, and the UI processing logic. 

The experiment turned out to be very successful, resulting in the following:

* A consistent experience across their Android and iOS apps.
* A reduction in their maintenance and testing costs.
* Significantly improved productivity within the team.

[![Explore real-world Kotlin Multiplatform use cases](kmp-use-cases-1.svg){width="700"}](https://www.jetbrains.com/help/kotlin-multiplatform-dev/case-studies.html)

### 2. Kotlin Multiplatform supports an extensive list of platforms

One of the key advantages of Kotlin Multiplatform is its extensive support across various platforms, 
making it a versatile choice for developers.
These platforms include Android, iOS, desktop, web (JavaScript and WebAssembly), and server (Java Virtual Machine).

_Quizlet_, a popular educational platform that aids learning and practice through quizzes, 
serves as another case study highlighting the benefits of Kotlin Multiplatform.
The platform has approximately 50 million active users per month, with 10 million of them on Android. 
The app ranks inside the top 10 in the education category on Apple's App Store.

The Quizlet team experimented with technologies like JavaScript, React Native, C++, Rust, and Go, 
but faced various challenges, including performance, stability, and varying implementations across platforms. 
Finally, they opted for Kotlin Multiplatform for Android, iOS, and web. 
Here's how using KMP benefited the Quizlet team:

* More type-safe APIs when marshaling objects.
* 25% faster grading algorithm on iOS compared to JavaScript.
* Reduced Android app size from 18 MB to 10 MB.
* Enhanced developer experience.
* Increased interest from team members, including Android, iOS, backend, and web developers, in writing shared code.

> [Discover the full range of capabilities offered by Kotlin Multiplatform](https://www.jetbrains.com/kotlin-multiplatform/)
> 
{style="tip"}

### 3. Kotlin provides simplified code-sharing mechanisms

In the world of programming languages, Kotlin stands out for its pragmatic approach,
which means it prioritizes the following features:

* **Readability over brevity**. While concise code is appealing, Kotlin understands that clarity is paramount. 
  The goal isn't just to shorten code but to eliminate unnecessary boilerplate, which enhances readability and maintainability.

* **Code reuse over sheer expressiveness**. It's not just about solving many problems – it's about identifying
  patterns and creating reusable libraries. By leveraging existing solutions and extracting commonalities, 
  Kotlin enables developers to maximize the efficiency of their code.

* **Interoperability over originality**. Rather than reinventing the wheel, 
  Kotlin embraces compatibility with established languages like Java. 
  This interoperability not only allows seamless integration with the vast Java ecosystem but also facilitates 
  the adoption of proven practices and lessons learned from previous experiences.

* **Safety and tooling over soundness**. Kotlin enables developers to catch errors early, 
  ensuring that your program doesn't fall into an invalid state. 
  By detecting issues during compilation or while writing code in IDEs, 
  Kotlin enhances software reliability, minimizing the risk of runtime errors.

We conduct annual Kotlin surveys to help us learn about users' experiences with the language. 
This year, 92% of respondents reported having a positive experience, a notable increase from 86% a year earlier.

![Kotlin satisfaction rate for 2023 and 2024](kotlin-satisfaction-rate.png){width=700}

The key takeaway is that Kotlin’s emphasis on readability, reuse, interoperability, and safety 
makes the language a compelling choice for developers and increases their productivity.

### 4. Kotlin Multiplatform allows for flexible multiplatform development

With Kotlin Multiplatform, developers no longer need to decide between native and cross-platform development 
They can choose what to share and what to write natively.

Before Kotlin Multiplatform, developers had to write everything natively.

![Before Kotlin Multiplatform: writing all code natively](before-kotlin-multiplatform.svg){width=700}

Kotlin Multiplatform allows developers to share business logic, presentation logic, or even UI logic, 
thanks to [Compose Multiplatform](https://www.jetbrains.com/lp/compose-multiplatform/).

![With Kotlin Multiplatform and Compose Multiplatform:  developers can share business logic, presentation logic, or even UI logic](with-compose-multiplatform.svg){width=700}

Now, you can share almost anything, except for platform-specific code.

### 5. With the Kotlin Multiplatform solution, you can share UI code

JetBrains provides [Compose Multiplatform](https://www.jetbrains.com/lp/compose-multiplatform/), a declarative framework for sharing user interfaces across multiple platforms, 
including Android (via Jetpack Compose), iOS, desktop, and web (Alpha), based on Kotlin and Jetpack Compose.

_Instabee_, a last-mile logistics platform specialized for e-commerce businesses, 
started using Compose Multiplatform in their Android and iOS applications, 
sharing the UI logic, while the technology was still in its alpha state.

There is an official sample of Compose Multiplatform called [ImageViewer App](https://github.com/JetBrains/compose-multiplatform/tree/master/examples/imageviewer),
which runs on Android, iOS, desktop, and web and has integration with native components like maps and the camera.
There is also a community sample, the [New York Times App](https://github.com/xxfast/NYTimes-KMP) clone, 
which runs even on Wear OS for watches, the smartwatch operating system. 
Check this list of [Kotlin Multiplatform and Compose Multiplatform samples](multiplatform-samples.md) to see more examples.

[![Explore Compose Multiplatform](explore-compose.svg){width="700"}](https://www.jetbrains.com/compose-multiplatform/)

### 6. You can use Kotlin Multiplatform in existing and new projects

Let's take a look at the following two scenarios:

* **Using KMP in an existing project**

  Once again, there is an example of Wonder App by Baidu. 
  The team already had the Android and iOS apps, and they just unified the logic. 
  They started gradually unifying more libraries and more logic, and then achieved a unified codebase shared across platforms.

* **Using KMP in a new project**

  _9GAG_, an online platform and social media website, tried different technologies, like Flutter and React Native,
  but finally opted for Kotlin Multiplatform, which allowed them to align the behavior of their app across both platforms.
  They started by creating an Android app first. Then, they consumed the Kotlin Multiplatform project as a dependency on iOS.

### 7. With Kotlin Multiplatform, you can start sharing your code gradually

You can begin incrementally, starting with simple elements like constants and gradually migrating common utilities like email validation.
You can also write or migrate your business logic, such as transaction processes or user authentication, for instance.

At JetBrains, we frequently conduct Kotlin Multiplatform surveys and ask our community about what parts of code they share between different platforms.
These surveys revealed that data models, data serialization, networking, analytics 
and internal utilities were among the key areas where this technology makes a significant impact. 

![Parts of code users are able to share between platforms with Kotlin Multiplatform: survey results](parts-of-code-share.png){width=700}

### 8. Kotlin Multiplatform is already used by global companies

KMP is already used by many large companies all around the world, including Forbes, Philips, Cash App, Meetup, Autodesk,
and many others. You can read all of their stories on the [case studies page](case-studies.topic).

In November 2023, JetBrains announced that Kotlin Multiplatform is now [Stable](https://blog.jetbrains.com/kotlin/2023/11/kotlin-multiplatform-stable/),
attracting the interest of more companies and teams in the technology.

### 9. Kotlin Multiplatform provides powerful tooling support

When working with Kotlin Multiplatform projects, you have powerful tooling at your fingertips.

* **Android Studio**. This integrated development (IDE) environment is built upon IntelliJ Community Edition
  and is widely regarded as the industry standard for Android development. 
  Android Studio provides a comprehensive suite of features for coding, debugging, and performance monitoring.
* **Xcode**. Apple's IDE can be used to create the iOS part of Kotlin Multiplatform apps.
  Xcode is the standard for iOS app development, offering a plethora of tools for coding, debugging, and configuration.
  However, Xcode is Mac only.

### 10. Kotlin Multiplatform boasts a large and supportive community

Kotlin and Kotlin Multiplatform have a very supportive community. Here are a few places where you can find answers to any questions you might have.

* [Kotlinlang Slack workspace](https://slack-chats.kotlinlang.org/).
  This workspace has about 60,000 members and a few relevant channels dedicated to cross-platform development,
  like [#multiplatform](https://slack-chats.kotlinlang.org/c/multiplatform),
  [#compose](https://slack-chats.kotlinlang.org/c/compose),
  and [#compose-ios](https://slack-chats.kotlinlang.org/c/compose-ios).
* [Kotlin X](https://twitter.com/kotlin). Here, you'll find quick expert insights and the latest news, including myriad multiplatform tips.
* [Kotlin YouTube](https://www.youtube.com/channel/UCP7uiEZIqci43m22KDl0sNw).
  Our YouTube channel provides practical tutorials, livestreams with experts, and other excellent educational content for visual learners.
* [Kotlin Roundup](https://lp.jetbrains.com/subscribe-to-kotlin-news/). 
  If you want to stay in the loop with the latest updates across the dynamic Kotlin and Kotlin Multiplatform ecosystems,
  subscribe to our regular newsletter!

The Kotlin Multiplatform ecosystem is thriving. It's enthusiastically nurtured by numerous Kotlin developers globally.
Here is a diagram showing the number of Kotlin Multiplatform libraries created per year:

![The number of Kotlin Multiplatform libraries over the years.](kmp-libs-over-years.png){width=700}

As you can see, there was a definite uptick in 2021, and the number of libraries hasn't stopped growing since.

## Why choose Kotlin Multiplatform over other cross-platform technologies?

When choosing between [different cross-platform solutions](cross-platform-frameworks.md), 
it's essential to weigh up both their advantages and disadvantages. 
Here's a breakdown of the key reasons why Kotlin Multiplatform might be the right choice for you:

* **Excellent tooling, easy to use**. Kotlin Multiplatform leverages Kotlin, offering excellent tooling and ease of use for developers.
* **Native programming**. It's easy to write things natively.
  Thanks to the [expected and actual declarations](multiplatform-expect-actual.md),
  you can enable your multiplatform app to access platform-specific APIs.
* **Great cross-platform performance**. Shared code written in Kotlin is compiled into different output formats for different targets:
  Java bytecode for Android and native binaries for iOS, ensuring good performance across all platforms.

If you've already decided to try Kotlin Multiplatform, here are a few tips that will help you get started:

* **Start small**. Begin with small shared components or constants to familiarize the team with the workflow and benefits of 
  Kotlin Multiplatform.
* **Create a plan**. Develop a clear experiment plan, hypothesizing the expected outcomes and methods for implementation and analysis.
  Define roles for contributing to shared code and establish workflows for distributing changes effectively.
* **Evaluate and run a retrospective**. Run a retrospective meeting with your team to assess the experiment's success 
  and identify any challenges or areas for improvement.
  If it worked for you, you might want to expand the scope and share even more code.
  If not, you need to understand the reasons why this experiment didn't work out.

[![See Kotlin Multiplatform in action! Get Started now](kmp-get-started-action.svg){width="700"}](https://www.jetbrains.com/help/kotlin-multiplatform-dev/get-started.html)

For those who want to help their team get started with Kotlin Multiplatform, we prepared a [detailed guide](multiplatform-introduce-your-team.md) with practical tips.

As you can see, Kotlin Multiplatform is already being successfully used by many massive companies to build 
high-performance cross-platform applications with native-looking UIs, effectively reusing code across them, 
while maintaining the benefits of native programming.