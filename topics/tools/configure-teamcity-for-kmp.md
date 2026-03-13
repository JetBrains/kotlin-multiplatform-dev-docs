# Configure TeamCity for a Kotlin Multiplatform application

This article explains how to configure [TeamCity](https://www.jetbrains.com/teamcity/?source=google&medium=cpc&campaign=EMEA_en_DE_TeamCity_Branded&term=jetbrains%20teamcity&content=771411250243&gad_source=1&gad_campaignid=12704027475&gbraid=0AAAAADloJzi5LQxd_2GSPDer8jKk00xHY&gclid=CjwKCAjwyMnNBhBNEiwA-Kcgu9u9Gprgz8eDZs4p-aG14ZSEn3A3JARU_VXxZaEFPMrxGydCbvNJdxoCmToQAvD_BwE) to build, test, and deploy your KMP applications. TeamCity supports all major VCS providers (GitHub, GitLab, Bitbucket, Azure DevOps, Perforce, and more), enables highly scalable hybrid workflows with both local and cloud agents, and includes powerful features such as multi-node setups for high availability, advanced user management, issue tracker integrations, and an AI assistant.

Grab your free TeamCity trial [here](https://www.jetbrains.com/teamcity/download/): choose the Cloud version with JetBrains-hosted agents preconfigured with major build tools and SDKs, or opt for TeamCity On-Premises for maximum control and a free lifetime Professional license.

This tutorial is based on the [JetCaster KMP sample](https://github.com/kotlin-hands-on/jetcaster-kmp-migration/).


## Create a new project

Every TeamCity workflow starts with a project. Projects own entities (build configurations and pipelines) that run actual CI/CD routines, store cloud profiles used to spin up cloud agents, share parameters with child objects, and more.

> Refer to these TeamCity documentation articles for more information:
> * [Project administrator guide](https://www.jetbrains.com/help/teamcity/project-administrator-guide.html#Steps%2C+Configurations+and+Projects)
> * [Create and edit projects](https://www.jetbrains.com/help/teamcity/creating-and-editing-projects.html#Create+New+Projects+in+Kotlin+DSL)
>
{style="tip"}

1. Click a plus button in the side navigation bar to start a new project.
2. Specify the project name and (optionally) fill in its description.
3. After you click **Create**, TeamCity asks you to choose the type of an object that will perform actual building tasks: a build configuration or a pipeline.

   <img src="teamcity-kmp-projectselector.png" width="708" alt="Choose configurations or pipelines"/>

    <deflist type="medium">

    <def title="Build configuration">

   Supports the full range of TeamCity features, lets you store configuration settings as Kotlin DSL code, and offers unmatched customization. However, it may require more experience and manual setup.

   Learn more: [Create and edit build configurations](https://www.jetbrains.com/help/teamcity/creating-and-editing-build-configurations.html).

    </def>

    <def title="Pipeline">

   Geared toward less experienced users and simpler workflows, pipelines offer a more intuitive design with a visual editor, editable YAML configuration, and easily accessible settings. Introduced in version 2025.11, they currently lack some functionality available in build configurations.

   Learn more: [Create and edit pipelines](https://www.jetbrains.com/help/teamcity/create-and-edit-pipelines.html).

    </def>

    </deflist>

   For this tutorial, choose pipelines as they are easier to configure and support all functionality required to build and test our sample project.

4. Select **Connect new repository** and choose either **GitHub** to create a permanent connection to GitHub that you'll be able to reuse for future projects, or **Any Git URL** for a limited connection to a specific repository (the sample JetCaster application or your personal fork).

5. After TeamCity verifies it can access the required repository, it retrieves the information about branches and asks you to specify basic pipeline behavior.

   <img src="teamcity-kmp-pipelinesettings.png" width="708" alt="Basic pipeline settings"/>

   Leave default settings to allow the pipeline to track all repository branches, use `main` as the default branch, and automatically trigger new runs whenever a new change is committed to the repository.


## Add pipeline jobs

Once your pipeline is ready, TeamCity will navigate you to its settings. You can use the toggle in the top left corner to switch between the visual and code editors.

<img src="teamcity-kmp-clientarea.png" width="706" alt="Main client area"/>

TeamCity pipelines consist of jobs: collections of consecutively executed build steps. A build step is the smallest block of any TeamCity routine that encapsulates a specific set of actions.

In TeamCity UI, click a job tile to edit its settings, or the darker region beneath jobs to edit global pipeline settings.

### Common pipeline settings

Pipeline settings affect all jobs it owns.

* The **Auto-run pipelines** section allows you to set up your pipeline to run automatically whenever a new change is committed to a remote repository (enabled by default), when a new pull request is sent to this repo, or on a given schedule.
* The **Repository** section allows you to check out and process multiple different repositories from multiple VCS hosting providers.
* The **Integrations** section lets you connect external NPM and Docker registries. Note that if you plan to run certain build steps inside public Dockerhub images, you do not need to set up a corresponding integration (unless your pipeline will run frequently enough to violate Dockerhub rate limits for anonymous pulls).

This tutorial does not require setting any additional pipeline settings. For more information, refer to this article: [Pipeline settings](https://www.jetbrains.com/help/teamcity/pipeline-settings.html).

### Agent settings

Building tasks are processed by build agents installed on bare metal or cloud machines. As such, these machines should have all the tooling required by this build task installed. For example, Job 2 of this pipeline requires Android SDK installed, and Job 3 uses Xcode to build an iOS version of the app.

* TeamCity Cloud employs JetBrains-hosted agents [equipped with a wide range of build tools](https://www.jetbrains.com/help/teamcity/cloud/jetbrains-hosted-agents.html#Agent+Software). For this tutorial, you do not need to connect any additional agents.
* If you're using TeamCity On-Premises, you need to make sure every job can run on at least one agent. See this article for more information: [Install and start TeamCity agents](https://www.jetbrains.com/help/teamcity/install-and-start-teamcity-agents.html).

In this tutorial, jobs specify agent requirements to ensure these jobs are assigned only to agents that have required tools installed.

### Run shared tests

Switch to the YAML pipeline editor and paste the following markup to set up the first job.

```yaml
jobs:
  Job1:
    name: Run tests
    steps:
      - type: gradle
        use-gradle-wrapper: true
        name: Gradle test
        jdk-home: '%\env.JDK_17_0%'
        tasks: jvmTest
    files-publication:
      - path: '**/build/reports/tests/**/*'
        share-with-jobs: false
        publish-artifact: true
    allow-reuse: false
```

This job runs the `jvmTest` Gradle task using Java 17, groups all files whose paths include `.../build/reports/tests/...` to the "test-reports" folder, and publishes this folder as an artifact.

You can additionally enable the **Optimizations | Parallel tests** job option to break the test suite down into smaller batches and process each batch on a separate build agent. This can drastically speed up the overall run duration but employs more resources.

```yaml
    ...
    allow-reuse: false
    parallelism: 3
```

The **Allow reuse** optimization option specifies whether TeamCity should skip re-running tasks if there were neither pipelines were edited nor new commits were made to sources.

See these topics for more information: [Job settings](https://www.jetbrains.com/help/teamcity/job-settings.html) | [Gradle build step](https://www.jetbrains.com/help/teamcity/gradle.html).


### Build the Android debug package

Modify the pipeline YAML as follows:


```yaml
jobs:
  Job1:
    name: Run tests
    ...
    Job2:
      name: Build Android
      steps:
        - type: gradle
          jdk-home: '%\env.JDK_17_0%'
          tasks: ':mobile:assembleDebug'
          use-gradle-wrapper: true
      files-publication:
        - path: mobile/build/outputs/apk/debug/*.apk
          share-with-jobs: false
          publish-artifact: true
      runs-on:
        self-hosted:
          - requirement: exists
            name: Android home
            parameter: env.ANDROID_HOME
      dependencies:
        - Job1
```

The `requirement` block ensures this job is never assigned to agents that have Android SDK installed, and `dependencies` section specifies that this job should start after "Job 1" finishes.

### Build the iOS simulator application

For the final step, add the following markup to the pipeline YAML:

```yaml
jobs:
  Job1:
    ...
  Job2:
    ...
  Job3:
    name: Build iOS
    steps:
      - type: script
        script-content: |-
          xcodebuild build \
            -project JetcasterMigration/JetcasterMigration.xcodeproj \
            -configuration Debug \
            -scheme JetcasterMigration \
            -sdk iphonesimulator \
            -derivedDataPath ./build \
            -verbose
    files-publication:
      - path: build/Build/Products/Debug-iphonesimulator/**/*
        share-with-jobs: false
        publish-artifact: true
    dependencies:
      - Job1
```

Unlike the first two jobs, "Build iOS" utilizes the universal [command-line build step](https://www.jetbrains.com/help/teamcity/command-line.html) that you can use to interact with any tool installed on the agent machine.

The `dependencies` section also sets the dependency on "Job1", meaning both "Build ..." jobs can run in parallel but only after the testing routine finishes.

> When working with build configurations, you can replace the Script build step with the specialized [Xcode Project step](https://www.jetbrains.com/help/teamcity/xcode-project.html).
>
{style="tip"}


## Run the pipeline

Hit **Save and Run** in the top right corner to start your workflow. Once a job finishes, artifacts it publishes will be available in the corresponding tab next to the build log.

<img src="teamcity-kmp-artifacts.png" alt="TeamCity artifacts" width="706"/>

Job 1 also displays the **Tests** tab that allows you to inspect test results.

<img src="teamcity-kmp-tests.png" alt="TeamCity tests" width="706"/>


## What's next

You can continue modifying this sample to gain additional benefits:

* Try adding a pipeline using a VCS connection. To do so, choose **GitHub** instead of **Any Git URL** when [adding a pipeline to a project](#create-a-new-project). Not only following this approach allows you to skip configuring VCS access for any future GitHub-based project, but also unlocks multiple additional pipeline features:

    * TeamCity will be able to [post run statuses](https://www.jetbrains.com/help/teamcity/create-and-edit-pipelines.html#Publish+Run+Statuses+to+VCS) (successful, failed, running) back to GitHub.
    * The [**On new changes** trigger](https://www.jetbrains.com/help/teamcity/pipeline-settings.html#On+New+Changes) and [**Repository** entries](https://www.jetbrains.com/help/teamcity/pipeline-settings.html#Repository) will gain new **Pull requests** toggle that allows you to track and build changes that were not yet committed to a stable branch.

* Explore [build configurations](https://www.jetbrains.com/help/teamcity/configuring-general-settings.html) to use advanced features unavailable in pipelines.

    * Use [build chains](https://www.jetbrains.com/help/teamcity/build-chain.html) and [composite configurations](https://www.jetbrains.com/help/teamcity/composite-build-configuration.html) to be able to run limited workflow portions. For example, run "Test &rarr; Build iOS" without triggering "Build Android", or trigger the testing configuration alone.
    * Enjoy the full range of JetBrains-crafted build steps, community recipes, and unbundled steps like [GitHub releases](https://blog.jetbrains.com/teamcity/2025/09/teamcity-github-releases-plugin/).
    * Deploy your [agents in a Kubernetes cluster](https://www.jetbrains.com/help/teamcity/setting-up-teamcity-for-kubernetes.html) or use it as an [external executor](https://www.jetbrains.com/help/teamcity/kubernetes-executor.html) for TeamCity.
    * Set up integrations with [issue trackers](https://www.jetbrains.com/help/teamcity/integrating-teamcity-with-issue-tracker.html) and [secret vaults](https://www.jetbrains.com/help/teamcity/hashicorp-vault.html).