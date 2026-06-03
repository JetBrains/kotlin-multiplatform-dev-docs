[//]: # (title: Switch Kotlin Multiplatform project from CocoaPods to SwiftPM dependencies with Junie)
<primary-label ref="alpha"/>

If you have a KMP module with CocoaPods dependencies and want to switch to Swift packages using [SwiftPM import](multiplatform-spm-import.md),
you can use AI to help you.
This guide shows how you can use Junie and Kotlin AI skills to make this process easier.

> Although this guide uses Junie, you can use any AI tool with [Kotlin AI skills](https://kotlinlang.org/docs/kotlin-ai-skills.html)
> to complete the process.
> 
{style="tip"}

As with all AI tools, Junie can make mistakes.
If you prefer to migrate manually, see [Switch Kotlin Multiplatform project from CocoaPods to SwiftPM dependencies](multiplatform-cocoapods-spm-migration.md).

## Set up Junie CLI

In the terminal, install Junie CLI:

```bash
curl -fsSL https://junie.jetbrains.com/install.sh | bash
```

Start the Junie CLI for the first time to log in with your JetBrains account
or use an external LLM:

```bash
junie
```

![Junie CLI login prompt](cocoapods-spm-junie-login.png){width="500"}

See Junie documentation to learn more about [authentication options](https://junie.jetbrains.com/docs/junie-cli.html#step-3-authenticate).

## Install the AI skill

In the terminal, navigate to your project directory and install the corresponding Kotlin AI skill:
<!-- Stable Junie CLI will support extensions soon https://junie.jetbrains.com/docs/junie-cli-extensions.html -->

```shell
npx skills add Kotlin/kotlin-agent-skills
```

> You need npm no older than 5.2.0 for this command to work.
> 
{style="note"}

In the dialog, select the `kotlin-tooling-cocoapods-spm-migration` skill and Junie as the agent to install it for.
When asked about the scope, select `Project` to limit the scope of the skill to your current project.

## Start the migration

Before you begin, make sure that your project is using VCS, for example, Git.
This is important so that you can review the changes from the initial state and after each iteration.

1. Open the terminal and navigate to your project directory.
2. Enter the following command to start Junie in interactive mode:

    ```shell
    junie
    ```

3. Enter the following prompt:

    ```text
    Migrate <project-name> from CocoaPods to SwiftPM
    ```
   
Junie recognizes that the skill you installed is appropriate for the task and starts the migration process.

## Review and test the changes

Review all the changes made by Junie in the project git history.
Use your Git client's side-by-side diff viewer to easily review the changes made.
For example, in IntelliJ IDEA:

![Side-by-side diff of changes made to the CocoaPods-dependent code](cocoapods-spm-junie-diff.png)

A successful migration modifies:
* `build.gradle.kts` files in the modules that depend on CocoaPods: the `cocoapods {}` blocks should be replaced with
  `swiftPMDependencies {}` blocks.
* Kotlin files that contain import directives referring to CocoaPods APIs replacing them with SwiftPM API imports

Test whether your project runs as it did before.
If you encounter problems, inspect the error messages in the logs and ask Junie to resolve them.
If you can't resolve the issues on your own, reach out in [Slack](https://kotlinlang.slack.com/archives/C8CFFCVAB) for support.

> To monitor your quota consumption, run the `/usage` command in Junie's interactive mode.
> 
{style="tip"}

## What's next

* Check out these sample projects which use CocoaPods in the `main` branch and SwiftPM in the `spm-import` branch:
    * [Firebase sample](https://github.com/Kotlin/kmp-with-cocoapods-firebase-sample/)
    * [Compose Multiplatform sample](https://github.com/Kotlin/kmp-with-cocoapods-compose-sample/)
* Learn about:
    * [Swift package export setup](multiplatform-spm-export.md)
    * [Adding Swift packages as dependencies to KMP modules](multiplatform-spm-import.md)
* Explore the other [Kotlin AI skills](https://kotlinlang.org/docs/kotlin-ai-skills.html).
