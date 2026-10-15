[//]: # (title: Kotlin and Compose Multiplatform in production: real-world use cases)

<web-summary>Discover how Kotlin Multiplatform with Compose Multiplatform is used in production across real-world projects. 
Explore practical use cases with examples.</web-summary>

> With large and small companies around the globe adopting Kotlin Multiplatform (KMP) with Compose Multiplatform, 
> the technology has become a trusted solution for building and scaling modern cross-platform applications.
> 
{style="note"}

## TL;DR

Companies use Kotlin Multiplatform in production to share code, reduce duplicated work, and deliver features consistently across platforms. Depending on their needs, they share business logic, SDK functionality, or both logic and UI:

* [Cash App](https://kotlinlang.org/lp/multiplatform/case-studies/cash-app) shares business logic between Android and iOS while keeping the UI native.

* [Philips](https://www.youtube.com/watch?v=hZPL8QqiLi8) uses Kotlin Multiplatform to maintain a shared mobile SDK and accelerate feature development.

* [McDonald's](https://kotlinlang.org/case-studies/#mcdonalds-umain) adopted Kotlin Multiplatform incrementally without rewriting its existing applications.

* [Duolingo](https://youtu.be/RJtiFt5pbfs) shares core business logic while preserving native user interfaces.

* [Booking.com](https://medium.com/booking-com-development/kotlin-multiplatform-in-production-two-real-world-use-cases-from-booking-com-46ffe13a773d) shares experimentation logic between Android and iOS to keep feature rollouts consistent.

* [Sony](https://kotlinlang.org/case-studies/#sony) uses Kotlin Multiplatform with Compose Multiplatform to share logic and UI while retaining access to native device capabilities.

These examples show that teams can adopt Kotlin Multiplatform gradually and choose how much code to share based on their project requirements.

Kotlin Multiplatform is used in production by startups and global enterprises across fintech, healthcare, media, logistics, education, 
retail, and e-commerce. Companies such as [Duolingo](https://youtu.be/RJtiFt5pbfs), [Booking.com](https://medium.com/booking-com-development/kotlin-multiplatform-in-production-two-real-world-use-cases-from-booking-com-46ffe13a773d), 
[Sony](https://kotlinlang.org/case-studies/#sony), [McDonald's](https://kotlinlang.org/case-studies/#mcdonalds-umain), [Forbes](https://www.forbes.com/sites/forbes-engineering/2023/11/13/forbes-mobile-app-shifts-to-kotlin-multiplatform/), [Philips](https://www.youtube.com/watch?v=hZPL8QqiLi8), [Cash App](https://kotlinlang.org/lp/multiplatform/case-studies/cash-app), [Instabee](https://www.youtube.com/watch?v=YsQ-2lQYQ8M), and [9GAG](https://raymondctc.medium.com/adopting-kotlin-multiplatform-mobile-kmm-on-9gag-app-dfe526d9ce04) 
use it to share business logic, develop cross-platform SDKs, adopt code sharing incrementally, and reduce duplicated engineering work while preserving native user experiences.

From integrating into existing apps and sharing app logic to building new cross-platform applications, 
[Kotlin Multiplatform](https://www.jetbrains.com/kotlin-multiplatform/) has become the technology of choice for many companies. These teams are capitalizing on the 
advantages that KMP provides to roll out their products faster and reduce development costs.

A growing number of businesses are also adopting [Compose Multiplatform](https://www.jetbrains.com/compose-multiplatform/), a declarative UI framework powered by 
Kotlin Multiplatform and Google’s Jetpack Compose. With the [stable release for iOS](https://blog.jetbrains.com/kotlin/2025/05/compose-multiplatform-1-8-0-released-compose-multiplatform-for-ios-is-stable-and-production-ready/), Compose Multiplatform completes the picture, 
making KMP a complete solution for cross-platform mobile development.

As adoption grows, this article examines how Kotlin Multiplatform is used in production across different industries and team structures.

## Kotlin Multiplatform use cases by the type of business and team

Here are several ways different teams are applying Kotlin Multiplatform to meet various project needs:

### Startups starting a new greenfield project
Why should startups consider Kotlin Multiplatform for their next greenfield project?

Startups often operate with limited resources and tight deadlines. To maximize development efficiency and cost-effectiveness, 
they benefit from targeting multiple platforms using a shared codebase — especially in early-stage products or MVPs, 
where time-to-market is critical.

For companies that want to share both logic and UI, Kotlin Multiplatform with Compose Multiplatform offers an ideal solution. 
You can start with a shared UI, enabling quick prototyping. You can even mix and match native UIs with shared UIs. 
This makes KMP with Compose Multiplatform an ideal choice for greenfield projects, helping startups balance speed, 
flexibility, and a high-quality native experience.

**Case studies:**

* [Instabee](https://www.youtube.com/watch?v=YsQ-2lQYQ8M) migrated its Android application logic and UI to Kotlin Multiplatform with Compose Multiplatform. 
  By using the Android codebase effectively, the company was able to release its iOS application in a short period of time.
* [Respawn Pro](https://youtu.be/LB5a2FRrT94?si=vgcJI-XoCrWree3u) develops a habit-tracking and productivity app. Its iOS app is built with Compose Multiplatform, 
  sharing 96% of its code with Android.

> If you’re choosing between [Kotlin Multiplatform and Flutter](https://www.jetbrains.com/help/kotlin-multiplatform-dev/kotlin-multiplatform-flutter.html), 
> don’t miss our overview of both technologies.
> 
{style="tip"}

### Small and medium-sized businesses
How can Kotlin Multiplatform help a small mobile team maintain a mature product?

Small and medium-sized businesses often have compact teams while maintaining mature, feature-rich products. 
Kotlin Multiplatform allows them to share core logic while keeping the native look-and-feel users expect. 
By relying on existing codebases, these teams can accelerate development without compromising on user experience.

KMP also supports a flexible approach to gradually introduce cross-platform capabilities. This makes it especially 
effective for teams evolving existing apps or launching new features, helping reduce development time, lower overhead, 
and maintain platform-specific customizations where needed.

**Case studies:**

* [Down Dog](https://kotlinlang.org/lp/multiplatform/case-studies/down-dog/?_gl=1*xdrptd*_gcl_au*ODIxNDk5NDA4LjE3MjEwNDg0OTY.*_ga*MTY1Nzk3NDc4MC4xNzA1NDc1NDcw*_ga_9J976DJZ68*MTcyNzg1MTIzNS4yMzcuMS4xNzI3ODUxNDM0LjU2LjAuMA..) uses a "maximum shared Kotlin" strategy for their application, which brings a studio-like yoga 
  experience to mobile devices. The company shares various helpers between clients and servers, and most of the client code, 
  with Kotlin Multiplatform. The team managed to significantly increase the app's development speed by keeping native-only views.
* [Doist](https://www.youtube.com/watch?v=z-o9MqN86eE) utilized Kotlin Multiplatform in its award-winning to-do list app, Todoist. The team shared key logic between Android 
  and iOS to ensure consistent behavior and streamline development. It adopted KMP incrementally, starting with internal libraries.

### Enterprises that need consistent behavior across devices for their applications
Can enterprises adopt Kotlin Multiplatform without rewriting large existing applications?

Large applications usually have extensive codebases, with new features constantly being added, and complex business logic 
that must work the same way on all platforms. Kotlin Multiplatform provides gradual integration, 
allowing teams to adopt it incrementally. And since developers can reuse their existing Kotlin skills, 
using KMP also saves them from introducing new tech stacks.

**Case studies:** [Duolingo](https://youtu.be/RJtiFt5pbfs), [Booking.com](https://medium.com/booking-com-development/kotlin-multiplatform-in-production-two-real-world-use-cases-from-booking-com-46ffe13a773d), [Sony](https://kotlinlang.org/case-studies/#sony), [Bilibili](https://kotlinlang.org/case-studies/#bilibili), [Forbes](https://www.forbes.com/sites/forbes-engineering/2023/11/13/forbes-mobile-app-shifts-to-kotlin-multiplatform/), [McDonald's](https://medium.com/mcdonalds-technical-blog/mobile-multiplatform-development-at-mcdonalds-3b72c8d44ebc), [Google Docs](https://www.youtube.com/watch?v=5lkZj4v4-ks), [Philips](https://www.youtube.com/watch?v=hZPL8QqiLi8), [VMware](https://medium.com/vmware-end-user-computing/adopting-a-cross-platform-strategy-for-mobile-apps-59495ffa23b0), [Cash App](https://kotlinlang.org/lp/multiplatform/case-studies/cash-app), and [Wonder by Baidu](https://kotlinlang.org/lp/multiplatform/case-studies/baidu).

> “After a successful initial test with the payments feature, we expanded Kotlin Multiplatform to our entire McDonald's application.”
>
> — **Varsha Singh**, Project Manager for the McDonald's app at Umain  
> [Read the McDonald's story](https://kotlinlang.org/case-studies/#mcdonalds-umain)

> “One exciting trend for Duolingo is that the more that we use Kotlin Multiplatform internally, the more we find ourselves speeding up in terms of shipping.”
>
> — **John Rodriguez**, Client Platform team at Duolingo  
> [Watch the full Duolingo story](https://youtu.be/RJtiFt5pbfs)

[![Learn from KMP success stories](kmp-success-stories.svg){width="700"}{style="block"}](https://kotlinlang.org/case-studies/?type=multiplatform)

### Agencies

Working with diverse clients, agencies and consultancies must accommodate a wide range of platform requirements and business goals. 
The ability to reuse code with Kotlin Multiplatform is especially valuable for teams managing multiple projects under 
tight timelines and limited engineering teams. By adopting KMP, agencies can accelerate delivery and maintain consistent 
app behavior across platforms. 

**Case studies:**

* [Touchlab](https://touchlab.co/) specializes in cross-platform development and advisory work with Kotlin Multiplatform.
  Touchlab also creates tools that improve your iOS development experience, such as [SKIE](https://github.com/touchlab/SKIE)
 which enhances Swift API published from Kotlin, and the [Kotlin plugin for Xcode](https://github.com/touchlab/xcode-kotlin).
* [IceRock](https://icerockdev.com/) is an outsourcing company that uses Kotlin Multiplatform to develop apps for its clients. 
  Its app portfolio spans various business requirements, complemented by a substantial collection of open-source 
  Kotlin Multiplatform libraries that enhance the Kotlin Multiplatform development process.
* [Mirego](https://kotlinlang.org/lp/multiplatform/case-studies/mirego/), an end-to-end digital product team, uses Kotlin Multiplatform to run the same business logic on the web, 
  iOS, tvOS, Android, and Amazon Fire TV. KMP allows it to streamline development while still getting the most out of each platform.  

### Companies expanding to new markets

Some companies want to enter new markets by launching their apps on platforms they haven't previously targeted, 
for example, moving from iOS-only to include Android or vice versa.

KMP helps you utilize existing iOS code and development practices while maintaining native performance and UI flexibility on Android. 
If you want to maintain platform-specific user experiences and take advantage of existing knowledge and code, 
KMP could be the ideal long-term solution.

**Case study:** [Instabee](https://www.youtube.com/watch?v=YsQ-2lQYQ8M) used Kotlin Multiplatform with Compose Multiplatform to migrate its Android app logic and UI. 
This allowed the company to enter the iOS market quickly by reusing much of its existing Android codebase.

### Teams developing software development kits (SDK)
Can Kotlin Multiplatform be used to build an SDK for both Android and iOS?

Shared Kotlin code compiles to platform-specific binaries (JVM for Android, native for iOS) and integrates seamlessly into any project. 
It offers flexibility, in that you can use platform-specific APIs without limitations, while also giving you a choice 
between native and cross-platform UI. These features make Kotlin Multiplatform an excellent option for developing mobile SDKs. 
From a consumer's perspective, your Kotlin Multiplatform SDK will behave like any regular platform-specific dependency, 
while still providing the benefit of shared code.

**Case study:** [Philips](https://www.youtube.com/watch?v=hZPL8QqiLi8) uses Kotlin Multiplatform in its HealthSuite Digital Platform mobile SDK, enabling faster development of new features and improving collaboration between Android and iOS developers.

> “We consolidated all our business logic into shared code, which means we can now develop once and deploy more.”
>
> — **Jeroen Brosens**, Software Architect at Philips  
> [Explore the Philips story](https://www.youtube.com/watch?v=hZPL8QqiLi8)
## Kotlin Multiplatform use cases by industry

Kotlin Multiplatform’s versatility is evident from the wide range of industries where it’s used in production. 
From fintech to education, KMP with Compose Multiplatform has been adopted in many types of applications. 
Here are a few industry-specific examples:

### Financial technology
Why is Kotlin Multiplatform useful for fintech applications?

Fintech applications often involve complex business logic, secure workflows, and strict compliance requirements, 
all of which must be implemented consistently across platforms. Kotlin Multiplatform helps unify this core logic in one codebase, 
reducing the risk of platform-specific inconsistencies. It ensures faster feature parity between iOS and Android, 
which is crucial for apps like wallets and payments.

**Case studies:** [Cash App](https://kotlinlang.org/lp/multiplatform/case-studies/cash-app), [Bitkey by Block](https://engineering.block.xyz/blog/how-bitkey-uses-cross-platform-development), and [Worldline](https://blog.worldline.tech/2022/01/26/kotlin_multiplatform.html).

> “We loved the ‘shared business, native UI’ idea that Kotlin Multiplatform promoted.”
>
> — **Alec Strong**, Mobile Developer at Cash App  
> [Read the Cash App case study](https://kotlinlang.org/case-studies/cash-app/)
### Media and publishing

Media and content-driven apps depend on fast feature rollout, consistent user experiences, and the flexibility to 
customize UIs for each platform. Kotlin Multiplatform allows teams to share core logic for content feeds and discovery sections, 
while maintaining full control over native UI. This accelerates development, reduces costly duplication, 
and ensures parity across platforms.

**Case studies:** [Forbes](https://www.forbes.com/sites/forbes-engineering/2023/11/13/forbes-mobile-app-shifts-to-kotlin-multiplatform/), [9GAG](https://raymondctc.medium.com/adopting-kotlin-multiplatform-mobile-kmm-on-9gag-app-dfe526d9ce04), [Kuaishou](https://medium.com/@xiang.j9501/case-studies-kuaiying-kotlin-multiplatform-mobile-268e325f8610)

### Project management and productivity

From shared calendars to real-time collaboration, productivity apps demand feature-rich functionality that must work 
identically on all platforms. Kotlin Multiplatform helps teams centralize this complexity in one shared codebase, 
ensuring consistent functionality and behavior on every device. This flexibility means teams can ship updates 
faster and maintain a unified user experience across platforms.

**Case studies:** [Wrike](https://www.youtube.com/watch?v=jhBmom8z3Qg), [VMware](https://medium.com/vmware-end-user-computing/adopting-a-cross-platform-strategy-for-mobile-apps-59495ffa23b0)

### Transportation and mobility

Ride-hailing, delivery, and mobility platforms benefit from Kotlin Multiplatform by sharing common features in 
their driver, rider, and merchant apps. Core logic for services like real-time tracking, route optimization, 
or in-app chat can be written once and used on both Android and iOS, guaranteeing consistent behavior for all users.

**Case studies:** [Bolt](https://medium.com/vmware-end-user-computing/adopting-a-cross-platform-strategy-for-mobile-apps-59495ffa23b0), 
[Feres](https://kotlinlang.org/case-studies/#case-study-feres)

### Educational technology
How can educational applications keep learning behavior consistent across platforms?

Education apps must deliver a seamless and consistent learning experience on both mobile and web, especially when supporting large, 
distributed audiences. By centralizing study algorithms, quizzes, and other business logic with Kotlin Multiplatform, 
educational apps deliver a uniform learning experience on every device. 
This code sharing can significantly boost performance and consistency — for example, Quizlet migrated its shared code
from JavaScript to Kotlin and saw notable speed improvements in both its Android and iOS apps.

**Case studies:** [Duolingo](https://youtu.be/RJtiFt5pbfs?si=mFpiN9SNs8m-jpFL), [Quizlet](https://quizlet.com/blog/shared-code-kotlin-multiplatform), [Chalk](https://kotlinlang.org/lp/multiplatform/case-studies/chalk/?_gl=1*1wxmdrv*_gcl_au*MTE5NzY3MzgyLjE3NDk3MDk0NjI.*FPAU*MTE5NzY3MzgyLjE3NDk3MDk0NjI.*_ga*MTM4NjAyOTM0NS4xNzM2ODUwMzA5*_ga_9J976DJZ68*czE3NTEwMjI5ODAkbzYwJGcxJHQxNzUxMDIzMTU2JGo1OCRsMCRoMA..), [Memrise](https://engineering.memrise.com/kotlin-multiplatform-memrise-3764b5a4a0db), 
[Physics Wallah](https://kotlinlang.org/case-studies/#case-study-physics-wallah)

### E-commerce

Building cross-platform shopping experiences means balancing shared business logic with native features like payments, 
camera access, and maps. Kotlin Multiplatform with Compose Multiplatform enables teams to share both business logic 
and UIs across platforms, while still using platform-specific components where needed. 
This hybrid approach ensures faster development, a consistent user experience, and the flexibility to integrate critical native features.

**Case studies:** [Balary Market](https://kotlinlang.org/case-studies/#case-study-balary), [Markaz](https://kotlinlang.org/case-studies/#case-study-markaz)

### Social networking and community

On social platforms, timely feature delivery and consistent interactions are essential for keeping communities active 
and connected across devices. Key interaction logic might include messaging, notifications, or scheduling. 
For example, Meetup, which allows users to find local groups, events, and activities, has been able to release 
new features simultaneously thanks to KMP.

**Case study:** [Meetup](https://youtu.be/GtJBS7B3eyM?si=lNX3KMhSTCICFPxv)

### Health and wellness

Whether guiding a yoga session or syncing health data across devices, wellness apps depend on both responsiveness 
and reliable cross-platform behavior. These apps often need to share core functionality, such as workout logic and data handling, 
while maintaining fully native UI and platform-specific integrations like sensors, notifications, or health APIs.

**Case studies:** [Respawn Pro](https://youtu.be/LB5a2FRrT94?si=vgcJI-XoCrWree3u), [Fast&amp;Fit](https://kotlinlang.org/case-studies/#case-study-fast-and-fit), [Philips](https://www.youtube.com/watch?v=hZPL8QqiLi8), [Down Dog](https://kotlinlang.org/lp/multiplatform/case-studies/down-dog)

### Postal services

While not a common use case, Kotlin Multiplatform has even been adopted by a 377-year-old national postal service. 
Norway’s Posten Bring uses KMP to unify complex business logic across dozens of frontend and backend systems, 
helping them streamline workflows and drastically reduce the time required to roll out new services — from months to days.

**Case study:** [Posten Bring](https://2024.javazone.no/program/a1d9aeac-ffc3-4b1d-ba08-a0568f415a02)

These examples highlight how Kotlin Multiplatform can be used in practically any industry or type of app. 
Whether you are building a fintech app, a mobility solution, an education platform, or something else, 
Kotlin Multiplatform provides the flexibility to share as much code as makes sense for your project, 
without sacrificing the native experience. You can also check out an extensive list of [KMP case studies](https://kotlinlang.org/case-studies/?type=multiplatform), 
showcasing many other companies that use the technology in production.

## Frequently asked questions

### What is Kotlin Multiplatform used for in production?

Kotlin Multiplatform is used to share business logic, UI, and core functionality across Android, iOS, desktop, and web platforms. Companies across industries such as fintech, education, media, and e-commerce rely on it to reduce development costs and accelerate feature delivery.

### Is Kotlin Multiplatform suitable for startups?

Yes. Kotlin Multiplatform allows startups to target multiple platforms from a shared codebase, which is especially valuable for MVPs where time to market is critical. Pairing Kotlin Multiplatform with Compose Multiplatform also enables UI sharing for faster prototyping.

### Can large enterprises adopt Kotlin Multiplatform incrementally?

Yes. Kotlin Multiplatform supports gradual integration. Teams can start by sharing a single layer, such as business logic, networking, or data, while keeping their existing native code and UI in place. They can then expand the amount of shared code over time without rewriting the entire application.

Companies such as Duolingo, Booking.com, Sony, and Google Docs have successfully integrated Kotlin Multiplatform into large, complex applications.

### Can Kotlin Multiplatform be used to build SDKs?

Yes. Shared Kotlin code compiles to platform-specific binaries and integrates as a native dependency on each platform. Philips, for example, uses Kotlin Multiplatform to develop its HealthSuite Digital Platform mobile SDK.

### Does Kotlin Multiplatform require sharing all code across platforms?

No. Kotlin Multiplatform lets teams decide how much code to share—from core business logic to the entire UI. Teams can also combine shared and native UIs based on their project requirements.

### What is Compose Multiplatform, and how does it relate to Kotlin Multiplatform?

Compose Multiplatform is a declarative UI framework powered by Kotlin Multiplatform and based on Google's Jetpack Compose. It enables teams to share UI code across Android, iOS, desktop, and web platforms. Its iOS support reached Stable status in 2025, making Kotlin Multiplatform with Compose Multiplatform a complete solution for cross-platform mobile development.

### Can agencies use Kotlin Multiplatform across multiple client projects?

Yes. Agencies can reuse Kotlin Multiplatform code across projects while working under tight delivery timelines. Companies such as Touchlab, IceRock, and Mirego specialize in Kotlin Multiplatform development and use it across diverse client engagements.
