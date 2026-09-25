[//]: # (title: How to introduce multiplatform mobile development to your team)

<web-summary>Learn how to introduce multiplatform mobile app development to your team with these six recommendations for smooth and efficient adoption.</web-summary>

Implementing new technologies and tools into an organization comes with challenges. How do you help your team adopt a [multiplatform approach to mobile app development](cross-platform-mobile-development.topic) to optimize and streamline your workflow? Here are some recommendations and best practices to help you effectively introduce your team to [Kotlin Multiplatform (KMP)](https://www.jetbrains.com/kotlin-multiplatform/), an open-source technology built by JetBrains that allows developers to [share code across platforms](https://kotlinlang.org/docs/multiplatform/multiplatform-share-on-platforms.html) while retaining the benefits of native programming.

* [Start with empathy](#start-with-empathy)
* [Explain how Kotlin Multiplatform works](#explain-how-kotlin-multiplatform-works)
* [Use case studies to demonstrate the value of multiplatform development](#use-case-studies-to-demonstrate-the-value-of-multiplatform-development)
* [Offer proof by creating a sample project](#offer-proof-by-creating-a-sample-project)
* [Prepare for questions about multiplatform development from your team](#prepare-for-questions-about-multiplatform-development-from-your-team)
* [Support your team during the adaptation period](#support-your-team-during-the-adaptation-period)

## Start with empathy

Software development is a team game, with each critical decision needing the approval of all team members. Integrating any cross-platform technology will significantly affect the development process for your mobile application. So before you start integrating Kotlin Multiplatform into your project, you'll need to introduce your team to the technology and guide them gently to see that it's worth adopting.

Understanding the people who work on your project is the first step to successful integration. Your boss is responsible for delivering features with the best quality in the shortest time possible. To them, any new technology is a risk. Your colleagues have a different perspective, as well. They have experience building apps with the "native" technology stack. They know how to write the UI and business logic, work with dependencies, test, and debug code in the IDE, and they are already familiar with the language. Switching to a different ecosystem is always inconvenient, as it always means leaving your comfort zone.

Given all that, be ready to face lots of biases and answer a lot of questions when advocating for the move to Kotlin Multiplatform. As you do, never lose sight of what your team needs. Some of the advice below might be useful for preparing your pitch.
> **Takeaway:** Successful Kotlin Multiplatform adoption begins with understanding the concerns of Android developers, iOS developers, technical leaders, and product stakeholders before changing established workflows.
## Explain how Kotlin Multiplatform works

At this stage, you need to show that using Kotlin Multiplatform could bring value to your project and eliminate any biased opinions and doubts about cross-platform mobile applications that your team might have.

KMP has been widely used in production since its Alpha release. As a result, JetBrains has been able to collect extensive feedback and provide an even better development experience in the [Stable version](https://blog.jetbrains.com/kotlin/2023/11/kotlin-multiplatform-stable/).

* **Ability to use all iOS and Android features** – Whenever a task cannot be accomplished in the shared code or whenever you want to use specific native features, you can use the [expect/actual](multiplatform-expect-actual.md) pattern to seamlessly write platform-specific code.
* **Seamless performance** – Shared code written in Kotlin is compiled into different output formats for different targets: Java bytecode for Android and [native binaries](https://kotlinlang.org/docs/multiplatform/multiplatform-build-native-binaries.html) for iOS. Thus, there is no additional runtime overhead when it comes to executing this code on platforms, and the performance is comparable to [native apps](native-and-cross-platform.topic).
* **Compatibility with legacy code** – No matter how large your project is, your existing code will not prevent you from integrating Kotlin Multiplatform. You can start writing cross-platform code at any moment and connect it to your iOS and Android apps as a regular dependency, or you can use the code you've already written and modify it to be compatible with iOS.

Kotlin Multiplatform can also improve collaboration between platform teams. According to the KMP Survey Q2 2024, 55% of users reported improved collaboration after adopting Kotlin Multiplatform, while 65% reported improvements in performance and quality.

**Source:** [KMP Survey Q2 2024](https://kotlinlang.org/docs/multiplatform/kmp-overview.html)

Being able to explain _how_ a technology works is crucial, as nobody likes it when a discussion seems to rely on magic. People might think the worst if anything is unclear to them, so be careful not to make the mistake of thinking something is too obvious to warrant an explanation. Instead, try to explain all the basic concepts before moving on to the next stage. This document on [multiplatform programming](get-started.topic) could help you systemize your knowledge to prepare for this experience.
> **Takeaway:** Kotlin Multiplatform lets teams adopt code sharing gradually while retaining direct platform API access, native performance, and their existing Android and iOS codebases.
## Use case studies to demonstrate the value of multiplatform development

Understanding how the multiplatform technology works is necessary, but not enough. Your team needs to see the gains of using it, and the way you present these gains should be related to your product.

At this stage, you need to explain the main gains of using Kotlin Multiplatform in your product. One way is to share stories of other companies who already benefit from cross-platform mobile development. The successful experience of these teams, especially ones with similar product objectives, could become a key factor in the final decision.

Citing case studies of different companies who already use Kotlin Multiplatform in production could significantly help you make a compelling argument:
* **Booking.com** – By adopting Kotlin Multiplatform for its experimentation library, Booking.com keeps experiment behavior consistent across Android and iOS while reducing duplicated implementation work.  
  [Read more about the case study](https://kotlinlang.org/case-studies/#booking)

* **Duolingo** – Kotlin Multiplatform helps Duolingo accelerate development and ship features consistently across Android and iOS.  
  [Read more about the case study](https://kotlinlang.org/case-studies/#duolingo)

* **Sony** – Sony moved from separate native development plans to Kotlin Multiplatform with Compose Multiplatform to build a shared companion application.  
  [Read more about the case study](https://kotlinlang.org/case-studies/#sony)

* **McDonald's** – By leveraging Kotlin Multiplatform for the Global Mobile App, McDonald's built a codebase that can be shared across platforms, removing the need for codebase redundancies.  
  [Read more about the case study](https://kotlinlang.org/case-studies/#mcdonalds-umain)

* **Netflix** – With the help of Kotlin Multiplatform, Netflix optimizes product reliability and delivery speed, which is crucial for serving its customers' needs.  
  [Read more about the case study](https://netflixtechblog.com/netflix-android-and-ios-studio-apps-kotlin-multiplatform-d6d4d8d25d23)

* **Forbes** – By sharing over 80% of logic across iOS and Android, Forbes now rolls out new features simultaneously on both platforms while retaining flexibility for platform-specific customization.  
  [Read more about the case study](https://www.forbes.com/sites/forbes-engineering/2023/11/13/forbes-mobile-app-shifts-to-kotlin-multiplatform/)

* **9GAG** – After trying both Flutter and React Native, 9GAG gradually adopted Kotlin Multiplatform, which now helps the team ship features faster while providing a consistent experience to users.  
  [Read more about the case study](https://raymondctc.medium.com/adopting-kotlin-multiplatform-mobile-kmm-on-9gag-app-dfe526d9ce04)
[![Learn from Kotlin Multiplatform success stories](kmp-success-stories.svg){width="700"}](https://www.jetbrains.com/help/kotlin-multiplatform-dev/case-studies.html)
> **Takeaway:** Production examples from Booking.com, Duolingo, Sony, and 9GAG help teams evaluate Kotlin Multiplatform using real-world evidence rather than theoretical benefits alone.
## Offer proof by creating a sample project

The theory is good, but putting it into practice is ultimately most important. As one option to make your case more convincing and show the potential of multiplatform mobile app development, you can devote some of your time to [creating something with Kotlin Multiplatform](https://kotlinlang.org/docs/mpp-get-started.html) and then bringing in the results for your team to discuss. Your prototype could be some sort of test project, which you would write from scratch and which would demonstrate features that are needed in your application. 
The [Create a multiplatform app using Ktor and SQLDelight – tutorial](multiplatform-ktor-sqldelight.md) guides you well on this process. 

You may be able to produce more relevant examples by experimenting with your current project. 
You could take one existing feature implemented in Kotlin and make it cross-platform, 
or you could even create a new Multiplatform Module in your existing project, 
take a non-priority feature from the bottom of the backlog, and implement it in the shared module. 
The [Make your Android application work on iOS – tutorial](multiplatform-integrate-in-existing-app.md) provides a step-by-step guide based on a sample project.

## Prepare for questions about multiplatform development from your team

No matter how detailed your pitch is, your team will have a lot of questions. Listen carefully and try to answer them all patiently. You might expect the majority of the questions to come from the iOS part of the team, as they are the developers who aren't used to seeing Kotlin in their everyday developer routine. This list of some of the most common questions could help you here:

### Q: I heard applications based on cross-platform technologies can be rejected from the App Store. Is taking this risk worth it?

A: The Apple Store has strict guidelines for publishing applications. One of the limitations is that apps may not download, install, or execute code that introduces or changes any features or functionality of the app ([App Store Review Guideline 2.5.2](https://developer.apple.com/app-store/review/guidelines/#software-requirements)). This is relevant for some cross-platform technologies, but not for Kotlin Multiplatform. Shared Kotlin code compiles to native binaries with [Kotlin/Native](https://kotlinlang.org/docs/multiplatform/multiplatform-build-native-binaries.html), bundles a regular iOS framework into your app, and doesn't provide the ability for dynamic code execution.

### Q: Multiplatform projects are built with Gradle, and Gradle has an extremely steep learning curve. Does this mean that I now need to spend a lot of time trying to configure my project? {id="gradle-time-spent"}

A: There's actually no need. There are various ways to organize the work process around building Kotlin mobile applications. First, only Android developers could be responsible for the builds, in which case the iOS team would only write code or even only consume the resulting artifact. You can also organize some workshops or practice pair programming when dealing with tasks that require working with Gradle, which would increase your team's Gradle skills. You can explore different [ways of organizing teamwork for multiplatform projects](https://kotlinlang.org/docs/multiplatform/multiplatform-project-configuration.html) and choose the one that's most appropriate for your team.

When only the Android part of the team works with shared code, the iOS developers don't even need to learn Kotlin. But when you are ready for your team to move to the next stage, where everyone contributes to the shared code, making the transition won't take much time. The similarities between the syntax and functionality of Swift and Kotlin greatly reduce the work required to learn how to read and write shared Kotlin code. [Try it yourself with Kotlin Koans](https://play.kotlinlang.org/koans/overview), a series of exercises to familiarize yourself with Kotlin syntax and some idioms.

JetBrains is developing [Kotlin Toolchain](https://kotlinlang.org/docs/multiplatform/kotlin-toolchain.html), the official evolution of Amper and a unified entry point for Kotlin projects. It provides a single `kotlin` command for creating, building, running, testing, and publishing projects. Kotlin Toolchain is currently in Alpha and under active development, so Gradle remains the established option for configuring production Kotlin Multiplatform projects.
### Q: Is Kotlin Multiplatform production-ready?

A: Yes. [Kotlin Multiplatform](https://kotlinlang.org/docs/multiplatform/kmp-overview.html) has been [Stable](https://blog.jetbrains.com/kotlin/2023/11/kotlin-multiplatform-stable/) since November 2023 and is used in production by organizations ranging from startups to global enterprises. Companies including Google, Duolingo, McDonald's, Booking.com, Philips, Sony, Cash App, and Bolt are featured in [Kotlin Multiplatform case studies](https://kotlinlang.org/case-studies/).

Teams can [adopt Kotlin Multiplatform gradually](https://blog.jetbrains.com/kotlin/2026/04/helping-decision-makers-say-yes-to-kmp/) by sharing selected business logic or use it more extensively, including shared UI with Compose Multiplatform. Explore more [real-world Kotlin and Compose Multiplatform use cases](https://kotlinlang.org/docs/multiplatform/use-cases-examples.html).
### Q: Do iOS developers need to learn Kotlin to contribute to a Kotlin Multiplatform project?

A: Not necessarily. Your team can set up the project so that Android or Kotlin developers maintain the shared code, while iOS developers consume the resulting shared module as a standard iOS framework. If the team later decides that iOS developers should contribute directly to shared code, they can learn Kotlin gradually. Kotlin and Swift share many language concepts and syntax patterns, which can make that transition easier.

### Q: Can iOS developers continue using SwiftUI or UIKit with Kotlin Multiplatform?

A: Yes. [Kotlin Multiplatform does not require teams to replace native iOS UI frameworks](https://kotlinlang.org/docs/multiplatform/kmp-for-ios.html). You can share business and data logic with Kotlin while continuing to build the iOS interface entirely with SwiftUI or UIKit. Teams that want to share UI also have the option of using Compose Multiplatform.

### Q: How do iOS developers debug problems in shared Kotlin code?

A: [Kotlin Multiplatform tooling supports debugging shared Kotlin code used by iOS applications](https://kotlinlang.org/docs/multiplatform/kmp-for-ios.html). Teams should also decide how responsibility for shared modules is distributed. Some teams have all mobile developers contribute to shared code, while others initially assign ownership to developers with Kotlin experience and gradually expand the group of contributors.

### Q: Does adopting Kotlin Multiplatform mean combining the Android and iOS teams?

A: Not necessarily. Kotlin Multiplatform supports several collaboration models. Platform teams can remain independent while sharing selected modules, or developers can gradually move toward shared ownership of common code. The appropriate model depends on the existing team structure and how much code the project plans to share.

### Q: There are not enough multiplatform libraries to implement my app's business logic, and it's much easier to find native alternatives. Why should I choose Kotlin Multiplatform? {id="not-enough-libraries"}

A: The Kotlin Multiplatform ecosystem is thriving and is being cultivated by many Kotlin developers around the world. Just take a look at how fast the number of KMP libraries has been growing over the years.

![The number of Kotlin Multiplatform libraries over years](kmp-libraries-over-years.png){width=700}

You can also browse [Kotlin Multiplatform libraries](https://klibs.io/) to find the one that support the platforms and use case you need.

It's also a great time to be an iOS developer in the Kotlin Multiplatform open-source community because iOS experience is in demand and there are plenty of opportunities to gain recognition for iOS-specific contributions.

The more your team digs into multiplatform mobile development, the more interesting and complex their questions will be. Don't worry if you don't have the answers – Kotlin Multiplatform has a large and supportive community in the Kotlin Slack with a dedicated [#multiplatform](https://slack-chats.kotlinlang.org/c/multiplatform) channel where a lot of developers who already use it can help you. We would be very thankful if you could [share with us](mailto:kotlin.multiplatform.feedback@kotlinlang.org) the most popular questions asked by your team. This information will help us understand what topics need to be covered in the documentation. 
> **Takeaway:** Addressing concerns about iOS development, Gradle, tooling, libraries, and production readiness early can remove common barriers to Kotlin Multiplatform adoption.
## Support your team during the adaptation period

After you decide to use Kotlin Multiplatform, there will be an adaptation period as your team experiments with the technology. And your mission will not be over yet! By providing continuous support for your teammates, you will reduce the time it takes for your team to dive into the technology and achieve their first results.

Here are some tips on how you can support your team at this stage:

* Collect the questions you were asked during the previous stage on a "Kotlin Multiplatform: Frequently Asked Questions" wiki page and share them with your team.
* Create a _#kotlin-multiplatform-support_ Slack channel and become the most active user there.
* Organize an informal team-building event with popcorn and pizza where you watch educational or inspirational videos about Kotlin Multiplatform. Here are a few good choices for videos:
   * [Getting Started With KMP: Build Apps for iOS and Android With Shared Logic and Native UIs](https://www.youtube.com/live/zE2LIAUisRI?si=V1cn1Pr-0Sjmjzeu) 
   * [Build Apps for iOS, Android, and Desktop With Compose Multiplatform](https://www.youtube.com/live/IGuVIRZzVTk?si=WFI3GelN7UDjfP97) 
   * [iOS Development With Kotlin Multiplatform: Tips and Tricks](https://www.youtube.com/watch?v=eFzy1BRtHps) 
   * [Kotlin Multiplatform for Teams by Kevin Galligan](https://www.youtube.com/watch?v=-tJvCOfJesk)

The reality is that you probably will not change people's hearts and minds in a day or even a week. But patience and attentiveness to the needs of your colleagues will undoubtedly bring results. 

The JetBrains team looks forward to hearing [your story about your experience with Kotlin Multiplatform](mailto:kotlin.multiplatform.feedback@kotlinlang.org).

_We'd like to thank the [Touchlab team](https://touchlab.co) for helping with the creation of this article._
## Next steps

Once your team is ready to evaluate Kotlin Multiplatform, choose the path that best matches your project.

### Try Kotlin Multiplatform

Follow the [Kotlin Multiplatform quickstart](https://kotlinlang.org/docs/multiplatform/get-started.html) to create and run your first multiplatform application.

### Add Kotlin Multiplatform to an existing project

Start gradually by moving a suitable feature or part of your business logic into shared code. Follow the [migration and integration guide](https://kotlinlang.org/docs/multiplatform/multiplatform-integrate-in-existing-app.html) to learn how to introduce Kotlin Multiplatform without rewriting your existing application.

### Share both logic and UI

If your team wants to share user interface code as well as application logic, explore [Compose Multiplatform](https://kotlinlang.org/docs/multiplatform/compose-multiplatform.html).

### Evaluate whether Kotlin Multiplatform is right for your team

If your team is still considering adoption, explore the following resources:

* [What is Kotlin Multiplatform?](https://kotlinlang.org/docs/multiplatform/kmp-overview.html)
* [Ten reasons to adopt Kotlin Multiplatform and supercharge your projects](https://kotlinlang.org/docs/multiplatform/multiplatform-reasons-to-try.html)
* [Kotlin and Compose Multiplatform in production: real-world use cases](https://kotlinlang.org/docs/multiplatform/use-cases-examples.html)
* [Kotlin Multiplatform case studies](https://kotlinlang.org/case-studies/)

## Frequently asked questions

### Q: How do I introduce Kotlin Multiplatform to my development team?

**A:** Start by identifying the problems Kotlin Multiplatform could solve for your project. Explain how code sharing works, provide examples from similar companies, and validate the approach with a small proof of concept before proposing broader adoption.

### Q: How should a team start adopting Kotlin Multiplatform?

**A:** Start with a small, well-defined part of an existing project, such as networking, data handling, validation, or another piece of business logic shared between platforms. Kotlin Multiplatform supports gradual adoption, so teams do not need to rewrite their applications or share all their code from the beginning.

### Q: Which companies use Kotlin Multiplatform in production?

**A:** Companies using Kotlin Multiplatform include Google, Duolingo, McDonald's, Booking.com, Sony, Philips, Cash App, Bolt, 9GAG, and many others across a wide range of industries and project architectures. Explore more examples on the [Kotlin Multiplatform case studies](https://kotlinlang.org/case-studies/?type=multiplatform) page.

### Q: Where should I start after my team decides to try Kotlin Multiplatform?

**A:** Start with the [Kotlin Multiplatform quickstart](https://kotlinlang.org/docs/multiplatform/quickstart.html) or build a small proof-of-concept application. For an existing Android project, follow the [migration tutorial](https://kotlinlang.org/docs/multiplatform/multiplatform-integrate-in-existing-app.html) and gradually move suitable logic into shared modules.