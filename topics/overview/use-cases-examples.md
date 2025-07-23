[//]: # (title: Kotlin and Compose Multiplatform in production: real-world use cases)

[//]: # (description: Discover how Kotlin Multiplatform with Compose Multiplatform is used in production across real-world projects. Explore practical use cases with examples.)

> With large and small companies around the globe adopting Kotlin Multiplatform (KMP) with Compose Multiplatform, the technology has become a trusted solution for building and scaling modern cross-platform applications.
> 
{style="note"}

From integrating into existing apps and sharing app logic to building new cross-platform applications, [Kotlin Multiplatform](https://www.jetbrains.com/kotlin-multiplatform/) has become the technology of choice for many companies. These teams are capitalizing on the advantages that KMP provides to roll out their products faster and reduce development costs.

A growing number of businesses are also adopting [Compose Multiplatform](https://www.jetbrains.com/compose-multiplatform/), a declarative UI framework powered by Kotlin Multiplatform and Google’s Jetpack Compose. With the [stable release for iOS](https://blog.jetbrains.com/kotlin/2025/05/compose-multiplatform-1-8-0-released-compose-multiplatform-for-ios-is-stable-and-production-ready/), Compose Multiplatform completes the picture, making KMP a complete solution for cross-platform mobile development.

Kotlin Multiplatform is now used in a growing number of real-world projects. This article outlines key scenarios – by team type and industry – with examples of how companies apply the technology in production. 

## Kotlin Multiplatform use cases by the type of business and team

Here are several ways different teams can leverage Kotlin Multiplatform to meet various project needs:

### Startups starting a new greenfield project

Startups often operate with limited resources and tight deadlines. To maximize development efficiency and cost-effectiveness, they benefit from targeting multiple platforms using a shared codebase – especially in early-stage products or MVPs, where time-to-market is critical.

For companies that want to share both logic and UI, Kotlin Multiplatform with Compose Multiplatform offers an ideal solution. You can start with a shared UI, enabling quick prototyping. You can even mix and match native UIs with shared UIs. This makes KMP with Compose Multiplatform an ideal choice for greenfield projects, helping startups balance speed, flexibility, and a high-quality native experience.

**Case studies:**

* [Instabee](https://www.youtube.com/watch?v=YsQ-2lQYQ8M) migrated their Android application logic and UI to Kotlin Multiplatform with Compose Multiplatform. Thanks to that, they were able to release their iOS application in a short period of time by leveraging the Android codebase.
* [Respawn Pro](https://youtu.be/LB5a2FRrT94?si=vgcJI-XoCrWree3u) develops a habit-tracking and productivity app. Their iOS app is built with Compose Multiplatform, sharing 96% of its code with Android.

> If you’re choosing between [Kotlin Multiplatform and Flutter](https://www.jetbrains.com/help/kotlin-multiplatform-dev/kotlin-multiplatform-flutter.html), you can check our overview of both technologies.
> 
{style="tip"}

### Small and medium-sized businesses

Small and medium-sized businesses often have compact teams while maintaining mature, feature-rich products. Kotlin Multiplatform allows them to share core logic while keeping the native look and feel users expect. By leveraging existing codebases, these teams can accelerate development without compromising on user experience.

KMP also supports a flexible approach to gradually introduce cross-platform capabilities. This makes it especially effective for teams evolving existing apps or launching new features, helping reduce development time, lower overhead, and maintain platform-specific customizations where needed.

**Case studies:**

* [Down Dog](https://kotlinlang.org/lp/multiplatform/case-studies/down-dog/?_gl=1*xdrptd*_gcl_au*ODIxNDk5NDA4LjE3MjEwNDg0OTY.*_ga*MTY1Nzk3NDc4MC4xNzA1NDc1NDcw*_ga_9J976DJZ68*MTcyNzg1MTIzNS4yMzcuMS4xNzI3ODUxNDM0LjU2LjAuMA..) uses a "maximum shared Kotlin" strategy for their application, which brings a studio-like yoga experience to mobile devices. They share various helpers between clients and servers, and most of the client code with Kotlin Multiplatform. Their team managed to significantly increase the app's development speed by keeping native-only views.
* [Doist](https://www.youtube.com/watch?v=z-o9MqN86eE) utilized Kotlin Multiplatform in their award-winning to-do list app, Todoist. The team shared key logic between Android and iOS to ensure consistent behavior and streamline development. They adopted KMP incrementally, starting with internal libraries.

### Enterprises that need consistent behavior across devices for their applications

Large applications usually have extensive codebases, with new features constantly being added, and complex business logic that must work the same way on all platforms. Kotlin Multiplatform provides gradual integration, allowing teams to adopt it incrementally. At the same time, it enables developers to reuse their existing Kotlin skills without introducing new tech stacks.

**Case studies:** [Forbes](https://www.forbes.com/sites/forbes-engineering/2023/11/13/forbes-mobile-app-shifts-to-kotlin-multiplatform/), [McDonald’s](https://medium.com/mcdonalds-technical-blog/mobile-multiplatform-development-at-mcdonalds-3b72c8d44ebc), [Google Docs](https://kotlinconf.com/talks/793515/), [Philips](https://www.youtube.com/watch?v=hZPL8QqiLi8), [VMware](https://medium.com/vmware-end-user-computing/adopting-a-cross-platform-strategy-for-mobile-apps-59495ffa23b0), [Cash App](https://kotlinlang.org/lp/multiplatform/case-studies/cash-app?_gl=1*1qc1ixl*_gcl_aw*R0NMLjE3NTEzNTcwMDguRUFJYUlRb2JDaE1JblBLRmc0cWJqZ01WZ0VnZENSM3pYQkVWRUFFWUFTQUFFZ0ltOVBEX0J3RQ..*_gcl_au*MTE5NzY3MzgyLjE3NDk3MDk0NjI.*FPAU*MTE5NzY3MzgyLjE3NDk3MDk0NjI.*_ga*MTM4NjAyOTM0NS4xNzM2ODUwMzA5*_ga_9J976DJZ68*czE3NTE1MjQ2MDUkbzcxJGcxJHQxNzUxNTI3Njc5JGozJGwwJGgw), [Wonder App by Baidu](https://kotlinlang.org/lp/multiplatform/case-studies/baidu)

[![Learn from KMP success stories](kmp-success-stories.svg){width="700"}](case-studies.topic)

### Agencies

Working with diverse clients, agencies and consultancies must accommodate a wide range of platform requirements and business goals. The ability to reuse code with Kotlin Multiplatform is especially valuable for teams managing multiple projects under tight timelines and limited engineering teams. By adopting KMP, agencies can accelerate delivery and maintain consistent app behavior across platforms. 

**Case studies:**

* [IceRock](https://icerockdev.com/) is an outsourcing company that leverages Kotlin Multiplatform to develop apps for its clients. Their app portfolio spans various business requirements, complemented by a substantial collection of open-source Kotlin Multiplatform libraries that enhance the Kotlin Multiplatform development process.
* [Mirego](https://kotlinlang.org/lp/multiplatform/case-studies/mirego/), an end-to-end digital product team, uses Kotlin Multiplatform to run the same business logic on the web, iOS, tvOS, Android, and Amazon Fire TV. KMP allows them to streamline development while still getting the most out of each platform.  

### Companies expanding to new markets

Some companies want to enter new markets by launching their apps on platforms they haven't previously targeted, for example, moving from iOS-only to include Android or vice-versa.

KMP helps you leverage existing iOS code and development practices while maintaining native performance and UI flexibility on Android. If you want to maintain platform-specific user experiences and leverage existing knowledge and code, KMP could be the ideal long-term solution.

**Case study:** [Instabee](https://www.youtube.com/watch?v=YsQ-2lQYQ8M) used Kotlin Multiplatform with Compose Multiplatform to migrate their Android app logic and UI. This allowed them to enter the iOS market quickly by reusing much of their existing Android codebase.

### Teams developing software development kits (SDK)

Shared Kotlin code compiles to platform-specific binaries (JVM for Android, native for iOS) and integrates seamlessly into any project. It offers flexibility, in that you can use platform-specific APIs without limitations, while also giving you a choice between native and cross-platform UI. These features make Kotlin Multiplatform an excellent option for developing mobile SDKs. From a consumer's perspective, your Kotlin Multiplatform SDK will behave like any regular platform-specific dependency, while still providing the benefit of shared code.

**Case study:** [Philips](https://www.youtube.com/watch?v=hZPL8QqiLi8) uses Kotlin Multiplatform in its HealthSuite Digital Platform mobile SDK, enabling faster development of new features and enhancing collaboration between Android and iOS developers.

## Kotlin Multiplatform use cases by industry

Kotlin Multiplatform’s versatility is evident from the wide range of industries where it’s used in production. From fintech to education, KMP with Compose Multiplatform has been adopted in many types of applications. Here are a few industry-specific examples:

### Financial technology

Fintech applications often involve complex business logic, secure workflows, and strict compliance requirements that must behave consistently across platforms. Kotlin Multiplatform helps unify this core logic in one codebase, reducing the risk of platform-specific inconsistencies. It ensures faster feature parity between iOS and Android, which is crucial for apps like wallets and payments.

**Case studies:** [Cash App](https://kotlinlang.org/lp/multiplatform/case-studies/cash-app), [Bitkey by Block](https://engineering.block.xyz/blog/how-bitkey-uses-cross-platform-development), [Worldline](https://blog.worldline.tech/2022/01/26/kotlin_multiplatform.html)

### Media and publishing

Media and content-driven apps depend on fast feature rollout, consistent user experiences, and the flexibility to customize UI for each platform. Kotlin Multiplatform allows teams to share core logic for content feeds and discovery sections, while maintaining full control over native UI. This accelerates development, reduces costly duplication, and ensures parity across platforms.

**Case studies:** [Forbes](https://www.forbes.com/sites/forbes-engineering/2023/11/13/forbes-mobile-app-shifts-to-kotlin-multiplatform/), [9GAG](https://raymondctc.medium.com/adopting-kotlin-multiplatform-mobile-kmm-on-9gag-app-dfe526d9ce04), [Kuaishou](https://medium.com/@xiang.j9501/case-studies-kuaiying-kotlin-multiplatform-mobile-268e325f8610)

### Project management and productivity

From shared calendars to real-time collaboration, productivity apps demand feature-rich functionality that must work identically on all platforms. Kotlin Multiplatform helps teams centralize this complexity in one shared codebase, ensuring consistent functionality and behavior on every device. This flexibility means teams ship updates faster and maintain a unified user experience across platforms.

**Case studies:** [Wrike](https://www.youtube.com/watch?v=jhBmom8z3Qg), [VMware](https://medium.com/vmware-end-user-computing/adopting-a-cross-platform-strategy-for-mobile-apps-59495ffa23b0)

### Transportation and mobility

Ride-hailing, delivery, and mobility platforms benefit from Kotlin Multiplatform by sharing common features in their driver, rider, and merchant apps. Core logic for services like real-time tracking, route optimization, or in-app chat can be written once and used on both Android and iOS, guaranteeing consistent behavior for all users.

**Case studies:** [Bolt](https://medium.com/vmware-end-user-computing/adopting-a-cross-platform-strategy-for-mobile-apps-59495ffa23b0), Feres

### Educational technology

Education apps must deliver a seamless and consistent learning experience on both mobile and web, especially when supporting large, distributed audiences. By centralizing study algorithms, quizzes, and other business logic with Kotlin Multiplatform, educational apps deliver a uniform learning experience on every device. This code sharing can significantly boost performance and consistency – for example, Quizlet migrated its shared code from JavaScript to Kotlin and saw notable speed improvements in both its Android and iOS apps.

**Case studies:** [Duolingo](https://youtu.be/RJtiFt5pbfs?si=mFpiN9SNs8m-jpFL), [Quizlet](https://quizlet.com/blog/shared-code-kotlin-multiplatform), [Chalk](https://kotlinlang.org/lp/multiplatform/case-studies/chalk/?_gl=1*1wxmdrv*_gcl_au*MTE5NzY3MzgyLjE3NDk3MDk0NjI.*FPAU*MTE5NzY3MzgyLjE3NDk3MDk0NjI.*_ga*MTM4NjAyOTM0NS4xNzM2ODUwMzA5*_ga_9J976DJZ68*czE3NTEwMjI5ODAkbzYwJGcxJHQxNzUxMDIzMTU2JGo1OCRsMCRoMA..), [Memrise](https://engineering.memrise.com/kotlin-multiplatform-memrise-3764b5a4a0db), Physics Wallah

### E-commerce

Building cross-platform shopping experiences means balancing shared business logic with native features like payments, camera access, and maps. Kotlin Multiplatform with Compose Multiplatform enables teams to share both business logic and UIs across platforms, while still using platform-specific components where needed. This hybrid approach ensures faster development, a consistent user experience, and the flexibility to integrate critical native features.

**Case studies:** Balary Market, Markaz

### Social networking and community

On social platforms, timely feature delivery and consistent interactions are essential for keeping communities active and connected across devices. Key interaction logic might include messaging, notifications, or scheduling. For example, Meetup, which allows users to find local groups, events, and activities, achieved the simultaneous release of new features by leveraging KMP.

**Case study:** [Meetup](https://youtu.be/GtJBS7B3eyM?si=lNX3KMhSTCICFPxv)

### Health and wellness

Whether guiding a yoga session or syncing health data across devices, wellness apps depend on both responsiveness and reliable cross-platform behavior. These apps often need to share core functionality, such as workout logic and data handling, while maintaining fully native UI and platform-specific integrations like sensors, notifications, or health APIs.

**Case studies:** [Respawn Pro](https://youtu.be/LB5a2FRrT94?si=vgcJI-XoCrWree3u), Fasting App, [Philips](https://www.youtube.com/watch?v=hZPL8QqiLi8), [Down Dog](https://kotlinlang.org/lp/multiplatform/case-studies/down-dog/?_gl=1*1ryf8m7*_gcl_au*MTE5NzY3MzgyLjE3NDk3MDk0NjI.*FPAU*MTE5NzY3MzgyLjE3NDk3MDk0NjI.*_ga*MTM4NjAyOTM0NS4xNzM2ODUwMzA5*_ga_9J976DJZ68*czE3NTEyNzEzNzckbzYyJGcxJHQxNzUxMjcxMzgzJGo1NCRsMCRoMA..)

### Postal services

While not a common use case, Kotlin Multiplatform has even been adopted by a 377-year-old national postal service. Posten Bring uses KMP to unify complex business logic across dozens of frontend and backend systems, helping them streamline workflows and drastically reduce the time required to roll out new services – from months to days.

**Case study:** [Posten Bring](https://2024.javazone.no/program/a1d9aeac-ffc3-4b1d-ba08-a0568f415a02)

These examples highlight how Kotlin Multiplatform can be used in practically any industry or application category. Whether you are building a fintech app, a mobility solution, an education platform, or something else, Kotlin Multiplatform provides the flexibility to share as much code as makes sense for your project without sacrificing the native experience. You can also check an extensive list of [KMP case studies](case-studies.topic) on its website, showcasing many other companies that use the technology in production.