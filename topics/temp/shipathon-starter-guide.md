[//]: # (title: Kotlin Multiplatform starter guide)

## Where to start

1. Learn about Kotlin Multiplatform (KMP) and Compose Multiplatform (CMP):
   What they are, their [advantages, and use cases](kmp-overview.md).
2. [Try KMP out on a sample project](quickstart.md) to see how it's organized and how it runs on different platforms.

## Learn KMP basics

The basics include:

* [Understand how a KMP / CMP project is organized](multiplatform-discover-project.md).
  This covers:
    * Common and platform-specific code in a shared module.
    * Targeted platform declarations.
* [Adding a dependency to a KMP project](multiplatform-add-dependencies.md).
    * For a practical example of multiplatform and platform-specific dependency organization, see our [sample](https://github.com/kotlin-hands-on/get-started-with-kmp/tree/main).
    * The tutorial that leads to the final state of that sample is [available in the documentation](multiplatform-upgrade-app.md).
* If you are already familiar with KMP, make sure you're up to date with the [recommended project structure](multiplatform-project-recommended-structure.md)
  for an average project.
  It takes into account how the release of Android Gradle plugin 9.0 affected the requirements for a KMP project
  and covers:
    * Module structure (independent app modules with shared code modules used as libraries).
    * Creating new app modules and transitioning from an older structure used with AGP 8.
* [Suggested video on project structure](https://www.youtube.com/watch?v=Atvl0l7fm1Y) recorded by a JetBrains developer advocate.

<!-- ## \[AI Agents scenario tools TODO\] -->

## Share code

There are different ways to share code in a KMP project, with some platform specifics:

* The basic examples of calling common code from app modules are covered in onboarding tutorials:
    * [for native UI and shared logic](multiplatform-create-first-app.md)
    * [for shared UI and logic](compose-multiplatform-create-first-app.md)
* [How to access platform-specific APIs](multiplatform-connect-to-apis.md):
    * Use multiplatform libraries when possible.
    * Use the `expect`/`actual` mechanism when no suitable multiplatform library is available.
* While calling shared Kotlin from Android Kotlin is relatively straightforward, iOS interoperability takes some getting to know it:
    * [Learn how to integrate your shared code with the iOS app](multiplatform-ios-integration-overview.md#local-integration) (all samples referenced in this doc have examples of iOS integration set up).
      > CocoaPods is generally being phased out in favor of Swift Package Manager and is not something we recommend using in new projects.
      >
      {style="note"}   
    * Check out the [sample and tutorial](multiplatform-upgrade-app.md#add-more-dependencies) that includes making Kotlin coroutines work with iOS.
    * See the guide on using existing [SPM packages in your KMP iOS app](multiplatform-spm-import.md).
    * Read the [in-depth explanation of calling Swift / ObjC from Kotlin](https://kotlinlang.org/docs/native-objc-interop.html) and vice versa.
    * Learn about the more straightforward [Swift export](https://kotlinlang.org/docs/native-swift-export.html) approach (currently in Alpha).
    * In general, the less interop, the better, so for a smoother experience we recommend relying on Compose Multiplatform to build the bulk of your UI for all platforms.

## Discover the ecosystem

A comprehensive catalog of multiplatform libraries is available at [klibs.io](https://klibs.io/):

* Most popular cases are already covered with robust solutions,
  usually with available alternatives:
  [SQLDelight](https://sqldelight.github.io/sqldelight/) and [Room](https://developer.android.com/kotlin/multiplatform/room) for databases, [Ktor](https://ktor.io/) and [OkHttp](https://square.github.io/okhttp/) for networking, [Coil](https://coil-kt.github.io/coil/) for image loading, and so on.
* App samples built to use multiplatform libraries for most popular use cases are available:
    * [SQLDelight / Ktor / kotlinx-serialization / Koin](https://github.com/kotlin-hands-on/kmp-networking-and-data-storage/tree/final)
      with the corresponding [tutorial](multiplatform-ktor-sqldelight.md).
    * [The multiplatform Jetcaster app](https://github.com/kotlin-hands-on/jetcaster-kmp-migration)
      converted from the [original Android sample](https://github.com/android/compose-samples/tree/main/Jetcaster).

## Create a KMP library

If you decide to create apps which share code by using a multiplatform library, check out these documentation pages:

* [Basic library tutorial](create-kotlin-multiplatform-library.md)
* [Publishing configuration for a KMP library](multiplatform-publish-lib-setup.md)
* Tutorials on publishing artifacts to [Maven Central](multiplatform-publish-libraries-to-maven.md) and [npm](multiplatform-publish-libraries-to-npm.md)

## Publish the artifacts

* Read the [general article on publishing KMP apps](multiplatform-publish-apps.md).
* Don't forget about the [privacy manifest](multiplatform-privacy-manifest.md) required by the Apple App Store.

## Using AI for KMP development

### Before you start

#### Use the free Junie access

Junie is a JetBrains AI agent.
For shipathon participants, JetBrains offers free access to the EAP version of the Junie CLI agent.
Your Junie agent can also be used through the [AI chat feature in IntelliJ IDEs](https://www.jetbrains.com/ai-ides/#getstarted).

<a as="button" href="https://surveys.jetbrains.com/s3/Build-with-Junie-at-Shipaton-2026-Application-Form" mode="classic" icon="arrow-right" icon-position="right">Claim your Junie access</a>

#### Set up and commit AGENTS.md

AI agents heavily rely on AGENTS.md files when exploring an unfamiliar codebase,
so accurate and comprehensive context can noticeably improve the quality of insights and generated code.
For example, simply noting that your project uses Kotlin Multiplatform can help avoid a lot of cross-platform issues.

To learn about the format and see examples, check out the [AGENTS.md](https://agents.md/) website.

#### Configure useful MCP servers

These MCP servers can be useful for an AI agent trying to build an app in the KMP context:

* The [klibs.io](https://github.com/JetBrains/klibs-io/blob/master/integrations/mcp/README.md) server
  helps to look for a suitable multiplatform library.
* The [Compose Hot Reload](compose-hot-reload.md#mcp-server-for-ai-agents) server
  allows the agent to quickly iterate on the UI

### Build features

#### Use planning mode

For larger tasks and distributed work, **planning mode** supported by most agents can help break down the task
and generate a clear step-by-step instruction that you can verify before the code generation starts in earnest.

Spending time to review and refine the results of the work done in planning mode usually produces significantly better results
for implementing:
* user-facing features from scratch,
* architectural changes,
* library integrations,
* large refactorings.

#### Validate AI-generated changes

On top of the general AI non-determinism, Kotlin Multiplatform introduces multifaceted context that is hard to cover comprehensively.
For example, it is common for changes to be implemented well and working for one platform and breaking another.

As a way to address this, it's a good idea to introduce explicit acceptance criteria:

* Run target-specific tests after whenever they are available after introducing any changes.
* Verify that all configured KMP targets successfully build before considering a task complete.
* Review the implementation for platform-specific APIs leaking into common code:
  this can lead to agents (and humans) trying to use these APIs in later stages.

#### Use Kotlin AI skills

The Kotlin team builds and maintains AI skills aimed at solving Kotlin-specific problems.
See the [skills repository](https://github.com/Kotlin/kotlin-agent-skills) and install the skills for your agent.

#### Use Swift Package Manager to integrate native iOS libraries

For iOS functionality that does not yet have a multiplatform library supporting it,
you may need to integrate native iOS libraries.
We recommend using SwiftPM packages and the [corresponding DSL](multiplatform-spm-import.md) for configuring such dependencies.

Kotlin team maintains an [AI skill aimed at CocoaPods to SwiftPM migration](https://github.com/Kotlin/kotlin-agent-skills/tree/main/skills/kotlin-tooling-cocoapods-spm-migration),
which can also be useful for setting up SwiftPM integration from scratch. 

#### Set up agent orchestration

JetBrains Air offers agent orchestration which can help speed up work by coordinating multiple agents
working on different parts of the project simultaneously.

<a as="button" href="https://air.dev/" mode="classic" icon="arrow-right" icon-position="right">Try out Air</a>

### Iterate on UI

#### Use Figma to generate UI designs and Compose code

The [Figma MCP server](https://help.figma.com/hc/en-us/articles/32132100833559-Guide-to-the-Figma-MCP-server)
can help with converting designs into Compose code.

For generating UI designs from scratch, consider [Google Stitch](https://stitch.withgoogle.com/) or [Figma Make](https://www.figma.com/make/).

#### Use Gemini CLI as the agent for Compose UI tasks

We have seen consistently good results with Compose code generation from the Google's model even in the [Flash family](https://ai.google.dev/gemini-api/docs/models#gemini-3-stable).
It offers a good balance of generation speed, token consumption, and UI quality.

#### Use Compose Hot Reload to iterate on UI

[Compose Hot Reload](compose-hot-reload.md) enables almost real-time UI updates that reflect changes
you — or your agent — make in the Compose code.

To help agents work with UI, you can add the [Compose Hot Reload MCP server](compose-hot-reload.md#mcp-server-for-ai-agents)
to your agent configuration.
It enables the agent to directly trigger the reload, take screenshots, and even interact with the UI.

## Learning resources catalog

All of the mentioned resources, along with more in-depth guides and third-party content, are catalogued
on the [Learning resources](kmp-learning-resources.md) page.