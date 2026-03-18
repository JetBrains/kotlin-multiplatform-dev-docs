# Configure TeamCity for a Kotlin Multiplatform application

<web-summary>Learn how to configure a TeamCity Cloud or On-Premises project for Kotlin Multiplatform (KMP). 
This tutorial uses TeamCity pipelines that support on-the-fly YAML configuration editing and an intuitive visual editor.</web-summary>

This article explains how to configure [TeamCity](https://www.jetbrains.com/teamcity/) to build, test, and deploy your KMP applications. 
TeamCity supports all major VCS providers (GitHub, GitLab, Bitbucket, Azure DevOps, Perforce, and more), 
enables highly scalable hybrid workflows with both local and cloud agents, and includes powerful features such as 
multi-node setups for high availability, advanced user management, issue tracker integrations, and an AI assistant.

Grab your free TeamCity trial [here](https://www.jetbrains.com/teamcity/download/): 
choose the Cloud version with JetBrains-hosted agents preconfigured with major build tools and SDKs, 
or opt for TeamCity On-Premises for maximum control and a free lifetime Professional license.

This tutorial is based on the [JetCaster KMP sample](https://github.com/kotlin-hands-on/jetcaster-kmp-migration/).

## Create a new project

Every TeamCity workflow starts with a project. Projects own entities such as build configurations and pipelines, 
which run actual CI/CD routines, store cloud profiles used to spin up cloud agents, share parameters with child objects, and more.

> Refer to these TeamCity documentation articles for more information:
> * [Project administrator guide](https://www.jetbrains.com/help/teamcity/project-administrator-guide.html#Steps%2C+Configurations+and+Projects)
> * [Create and edit projects](https://www.jetbrains.com/help/teamcity/creating-and-editing-projects.html#Create+New+Projects+in+Kotlin+DSL)
>
{style="tip"}

1. Click the plus button in the side navigation bar to start a new project.
2. Specify the project name and, optionally, provide a description.
3. After you click **Create**, TeamCity will ask you to choose the type of object that will perform the actual building tasks: 
   a build configuration or a pipeline.

   <img src="teamcity-kmp-projectselector.png" width="500" alt="Choose configurations or pipelines"/>

   <deflist type="medium">
   <def title="Build configuration">
   Supports the full range of TeamCity features, allows you to store configuration settings as Kotlin DSL code, 
   and offers unmatched customization. However, it may require more experience and manual setup.

   Learn more: [Create and edit build configurations](https://www.jetbrains.com/help/teamcity/creating-and-editing-build-configurations.html).
   </def>
   <def title="Pipeline">
   Offers an intuitive design with a visual editor, editable YAML configurations, and easily accessible settings. 
   Pipelines are designed for less experienced users and simpler workflows.
   Introduced in TeamCity 2025.11, pipelines currently lack some of the functionality available in build configurations.

   Learn more: [Create and edit pipelines](https://www.jetbrains.com/help/teamcity/create-and-edit-pipelines.html).
   </def>
   </deflist>

   For this tutorial, choose pipelines, as they are easier to configure and support all functionality required to build and test our sample project.

4. Select **Connect new repository** and choose either **GitHub** to create a permanent connection to GitHub 
   that you can reuse for future projects, or **Any Git URL** for a limited connection to a specific repository 
   (the sample JetCaster application or your personal fork).

5. After TeamCity verifies it can access the required repository, it retrieves the information about branches and asks you to specify basic pipeline behaviors.

   <img src="teamcity-kmp-pipelinesettings.png" width="450" alt="Basic pipeline settings"/>

   Leave the default settings to allow the pipeline to track all repository branches, use `main` as the default branch, 
   and automatically trigger new runs whenever a change is committed to the repository.

## Add pipeline jobs

Once your pipeline is ready, TeamCity will navigate you to its settings page. 
You can use the toggle in the top left corner to switch between the visual and code editors.

<img src="teamcity-kmp-clientarea.png" width="450" alt="Main client area"/>

TeamCity pipelines consist of jobs, which are collections of consecutively executed build steps. 
A build step is the smallest unit of a TeamCity routine that encapsulates a specific set of actions.

In the TeamCity UI, click a job tile to edit its settings, or click the darker region beneath the jobs 
to modify the global pipeline settings.

### Common pipeline settings

This tutorial does not require setting up any global pipeline options. 
Refer to [this article](https://www.jetbrains.com/help/teamcity/pipeline-settings.html) for more information on settings that affect all jobs within a pipeline, such as:

* **Auto-run pipelines** — allows you to configure your pipeline to run automatically whenever a new change is 
  committed to a remote repository (enabled by default), when a pull request is opened for the repository, or on a set schedule.
* **Repository** — allows you to check out and process multiple repositories from different VCS hosting providers.
* **Integrations** — lets you connect external NPM and Docker registries. Note that if you plan to run build steps 
  inside public Docker Hub images, you do not need to configure a corresponding integration 
  unless your pipeline runs frequently enough to exceed Docker Hub's rate limits for anonymous pulls.

### Agent settings

Building tasks are processed by build agents installed on bare-metal or cloud machines. 
These machines must have all the tools required by the given build tasks installed. 
For example, Job 2 in this pipeline requires the Android SDK, while Job 3 uses Xcode to build an iOS version of the app.

* TeamCity Cloud uses JetBrains-hosted agents [equipped with a wide range of build tools](https://www.jetbrains.com/help/teamcity/cloud/jetbrains-hosted-agents.html#Agent+Software). 
  For this tutorial, you do not need to connect any additional agents.
* If you're using TeamCity On-Premises, you need to make sure that every job can run on at least one agent. 
  Refer to this article for more details: [Install and start TeamCity agents](https://www.jetbrains.com/help/teamcity/install-and-start-teamcity-agents.html).

In this tutorial, the jobs specify agent requirements to guarantee they are assigned only to agents with the necessary tools installed.

### Run shared tests

Switch to the YAML pipeline editor and paste the following markup to set up the first job:

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

This job runs the `jvmTest` Gradle task using Java 17. It collects all files whose paths match `.../build/reports/tests/...`, 
groups them under the `test-reports` folder, and publishes this folder as an artifact.

You can also enable the **Optimizations | Parallel tests** job option to split the test suite into smaller 
batches, processing each batch on a separate build agent. 
This can significantly reduce the overall run duration but will use more resources.
To enable parallel testing, modify the pipeline YAML to include the `parallelism` setting as shown below:

```yaml
    ...
    allow-reuse: false
    parallelism: 3
```

The **Allow reuse** optimization option specifies whether TeamCity should skip re-running tasks if no changes have 
been made to the pipeline configuration or source code.

For additional information, refer to [Job settings](https://www.jetbrains.com/help/teamcity/job-settings.html) 
and [Gradle build step](https://www.jetbrains.com/help/teamcity/gradle.html).

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

* The `requirement` block ensures this job will only be assigned to agents that have the Android SDK installed. 
* The `dependencies` section guarantees that this job starts only after `Job1` has successfully finished.

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

Unlike the first two jobs, **Build iOS** uses the universal [command-line build step](https://www.jetbrains.com/help/teamcity/command-line.html), 
which allows you to run commands or interact with any tool installed on the agent machine.

The `dependencies` section specifies a dependency on `Job1`, meaning that both **Build Android** and **Build iOS** jobs can run in parallel 
but will start only after the testing routine from `Job1` completes.

> When working with build configurations, you can replace the Script build step with the specialized [Xcode Project step](https://www.jetbrains.com/help/teamcity/xcode-project.html).
>
{style="tip"}

## Run the pipeline

Click **Save and Run** in the top-right corner to start your workflow. 
Once a job finishes, any artifacts it publishes will be available in the **Artifacts** tab next to the build log.

<img src="teamcity-kmp-artifacts.png" alt="TeamCity artifacts" width="450"/>

`Job1` will also display a **Tests** tab, allowing you to inspect the test results.

<img src="teamcity-kmp-tests.png" alt="TeamCity tests" width="450"/>

## What's next

You can continue modifying this sample to gain even more benefits:

* **Add a pipeline using a VCS connection**
 
  When [adding a new pipeline to a project](#create-a-new-project), choose **GitHub** instead of **Any Git URL**. 
  This approach not only allows you to skip configuring VCS access for future GitHub-based projects, 
  but also unlocks additional pipeline features:

    * TeamCity can [post run statuses](https://www.jetbrains.com/help/teamcity/create-and-edit-pipelines.html#Publish+Run+Statuses+to+VCS) (successful, failed, or running) directly to GitHub.
    * The [**On new changes** trigger](https://www.jetbrains.com/help/teamcity/pipeline-settings.html#On+New+Changes) and [**Repository** entries](https://www.jetbrains.com/help/teamcity/pipeline-settings.html#Repository) will include a **Pull requests** toggle,
      that allows you to track and build changes that were not yet committed to a stable branch.

* **Explore advanced build configurations**

  Switch from pipelines to [build configurations](https://www.jetbrains.com/help/teamcity/configuring-general-settings.html) for access to advanced features:

    * Use [build chains](https://www.jetbrains.com/help/teamcity/build-chain.html) and [composite configurations](https://www.jetbrains.com/help/teamcity/composite-build-configuration.html) to run specific workflow portions. 
      For example, run **Test &rarr; Build iOS** without triggering **Build Android**, or run the testing configuration alone.
    * Enjoy the full range of JetBrains-crafted build steps, community recipes, and unbundled steps like [GitHub releases](https://blog.jetbrains.com/teamcity/2025/09/teamcity-github-releases-plugin/).
    * Deploy your [agents in a Kubernetes cluster](https://www.jetbrains.com/help/teamcity/setting-up-teamcity-for-kubernetes.html), or use it as an [external executor](https://www.jetbrains.com/help/teamcity/kubernetes-executor.html).
    * Set up integrations with [issue trackers](https://www.jetbrains.com/help/teamcity/integrating-teamcity-with-issue-tracker.html) and [secret vaults](https://www.jetbrains.com/help/teamcity/hashicorp-vault.html).