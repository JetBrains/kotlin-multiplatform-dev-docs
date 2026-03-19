[//]: # (title: Publish your library to NPM – tutorial)

In this tutorial, you'll learn how to publish your Kotlin Multiplatform library targeting JS or WasmJS to the [NPM](https://www.npmjs.com/)
repository.

To publish your library, you'll need to:

1. Set up credentials, including [an account on NPM](https://docs.npmjs.com/creating-a-new-npm-user-account) and [an access token](https://docs.npmjs.com/creating-and-viewing-access-tokens).
2. Configure the publishing plugin in your library's project.
3. Provide the access token (and one-time password if two-factor authentication is enabled) to the publishing plugin so it can upload your artifacts.
4. Run the publication task, either locally or using continuous integration.

This tutorial assumes that you are:

* Creating an open-source library.
* Storing the code for your library in a GitHub repository.
* Either not registered on NPM yet or have an existing account.
* Using GitHub Actions for continuous integration.

## Sample library

In this tutorial, you'll use the [simple web](https://github.com/Kotlin/kotlin-multiplatform-web-library) library as an example.
You can refer to the code of that repository to see how the publishing setup works.

If you'd like to reuse the code, you **must replace all example values** with those specific to your project.

## Prepare accounts and credentials

To get started with publishing to NPM, [sign in](https://www.npmjs.com/login) (or [create a new account](https://docs.npmjs.com/creating-a-new-npm-user-account)) on the [NPM](https://www.npmjs.com/)
portal.

### Create a simple organization

In this tutorial, to avoid spending much time on choosing a unique name for the library,
we'll create a new organization and publish the library under this organization with any name.

To create a new organization, follow the guide [here](https://docs.npmjs.com/creating-an-organization).

#### Generate an access token

To publish to NPM, you'll need an access token that allows publishing a package under your newly created organization.
To generate such a token, follow the steps [here](https://docs.npmjs.com/creating-and-viewing-access-tokens).

> For the sake of simplicity, set the scope to `read-write` for the newly created organization and check the `Bypass two-factor authentication (2FA)` checkbox.
>
{style="note"}

As soon as you have the organization name and the access token, you're ready to set up your library to be published on NPM.

## Configure the project

### Prepare your library project

If you started developing your library from a template project, now is a good time to change any default names in the
project to match your own library's name. This includes the name of your library module and the name of the root project
in your top-level `build.gradle.kts` file.


### Set up the publishing plugin

This tutorial uses the official [npm-publish plugin](https://github.com/Kotlin/npm-publish)
to help with publishing to NPM.
See the [plugin's documentation](https://npm-publish.petuska.dev)
to learn more about its usage and available configuration options.

To add the plugin to your project, add the following line to the `plugins {}` block of your library module's `build.gradle.kts` file:

```kotlin
// <module directory>/build.gradle.kts

plugins {
    kotlin("npm-publish") version "%npmPublishPlugin%" 
}
```

> For the latest available version of the plugin, check its [Releases page](https://github.com/Kotlin/npm-publish/releases).
> 
{style="note"}

In the same file, add the following configuration, making sure to customize all the values for your library:

```kotlin
// <module directory>/build.gradle.kts
npmPublish {
    organization = "YOUR_ORGANIZATION_NAME_WITHOUT_AT_SIGN"
    
    registries {
        npmjs {
            authToken = System.getenv("NPM_TOKEN")
        }
    }

    packages {
        named("js") {
            version = "0.0.1"
            packageName = "greetings"
            readme = file("../README.md")

            packageJson {
                license = "Apache 2.0"
                homepage = "https://github.com/Kotlin/kotlin-multiplatform-web-library#readme"
                description = "Shared Kotlin/JS Greetings library"
                keywords = listOf("kotlin", "kotlin-js", "greetings", "shared", "api")
                author {
                    name = "Kotlin Developer Advocate"
                    url = "https://github.com/kotlin-hands-on/"
                }
                repository {
                    type = "git"
                    url = "https://github.com/Kotlin/kotlin-multiplatform-web-library.git"
                }
            }
        }
    }
}
```

> To configure this, you can also use [Gradle properties](https://docs.gradle.org/current/userguide/build_environment.html).
> 
{style="tip"}

The most important settings here are:

* The `registries`, which specify to which registry the library is going to be published.
* The `organization`, `packageName` (by default, the name of the module is taken), and `version` (by default, the version specified in the module is taken) of your library.
* The `license`, under which your library is published.
* The `author`, which specifies the author of the library (you can also add `contributors`).
* The `repository`, which specifies where the library's source code is hosted.
* The `readme`, which specifies which file should be added as the library's README.

### Publish locally

Let's try to publish the library to NPM from the local machine.

To do so, run the following command:
```bash
NPM_TOKEN=YOUR_ACCESS_TOKEN ./gradlew :shared:publishJsPackageToNpmjsRegistry
```

After the library is published, you can verify its presence on the NPM registry by visiting the package's page on [npmjs.com](https://www.npmjs.com/).
Your package URL should look like `https://www.npmjs.com/package/YOUR_ORGANIZATION_NAME/greetings`.

![Published library on NPM](published_on_npm.png){width=700}

## Publish to NPM using continuous integration

### Add a GitHub Actions workflow to your project

You can set up continuous integration to build and publish your library automatically.
We'll use [GitHub Actions](https://docs.github.com/en/actions) as an example.

To get started, add the following workflow to your repository in the `.github/workflows/publish.yml` file:

```yaml
# .github/workflows/publish.yml

name: Publish

on:
  release:
    types: [released, prereleased]

permissions:
  id-token: write  # Required for OIDC
  contents: read

jobs:
  publish:
    name: Release build and publish
    runs-on: ubuntu-latest
    steps:
      - name: Check out code
        uses: actions/checkout@v4

      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          distribution: 'zulu'
          java-version: 21

      - name: Publish to NPM
        run: ./gradlew :shared:publishJsPackageToNpmjsRegistry
```

Once you commit and push this file, the workflow will run automatically whenever you create a release (including a pre-release)
in the GitHub repository hosting your project. The workflow checks out the current version of your code, sets up a JDK,
and then runs the `publishJsPackageToNpmjsRegistry` Gradle task.

You can also configure the workflow to [trigger when a tag is pushed](https://stackoverflow.com/a/61892639) to your repository.

### Add a Trusted Publisher for your NPM package

As you can see, we haven't provided any credentials to the workflow.
This is because NPM has a special integration with popular CI/CD services called [Trusted Publisher](https://docs.npmjs.com/trusted-publishers).

So let's add our newly created workflow as a trusted publisher. 

1. Navigate to the published package from the ["Publish locally" step](#publish-locally): `https://www.npmjs.com/package/YOUR_ORGANIZATION_NAME/greetings`.
2. Navigate to the "Settings" tab.
3. Click the button "GitHub Actions" under "Select your publisher".
4. Set your GitHub name (or organization) and repository name.
5. Enter the name of the workflow file (in this tutorial, we've created [publish.yml](https://github.com/Kotlin/kotlin-multiplatform-web-library/blob/main/.github/workflows/publish.yml)).
6. Click the "Setup connection" button.

After that, you should see your repository listed in the "Trusted Publishers" section, which means that the workflow is now authorized to publish to NPM.

### Create a release on GitHub

With the workflow and secrets set up, you're now ready to [create a release](https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository#creating-a-release)
that will trigger the publication of your library.

1. Make sure that the version number specified in the `build.gradle.kts` file for your library is the one you would
   like to publish (if you've already published a version of your library, please update the version number).
2. Go to your GitHub repository's main page.
3. In the right sidebar, click **Releases**.
4. Click the **Draft a new release** button (or the **Create a new release** button if you haven't created a release for
   this repository before).
5. Each release has a tag. Create a new tag in the tag dropdown, and set the release title (the tag name and
   the title can be identical).
   
   You probably want these to be the same as the version number of the library that you specified in the `build.gradle.kts` file.

   ![Create a release on GitHub](create_release_and_tag_for_npm.png){width=700}

6. Double-check the branch you want to target with the release (especially if it's not the default branch) and add
   appropriate release notes for your new version.
7. Use the checkboxes below the description to mark a release as a pre-release (useful for early access versions such as
   alpha, beta, or RC).
   
   You can also mark the release as the latest one (if you already made a release for this repository before).
8. Click the **Publish release** button to create the new release.
9. Click the **Actions** tab at the top of your GitHub repository's page. Here, you'll see that the new release
   triggered your publishing workflow.
    
   You can click on the workflow to see the outputs of the publication task.
10. After the workflow run is complete, your new version should be available on the NPM registry (under the `https://www.npmjs.com/package/YOUR_ORGANIZATION_NAME/greetings` URL).

![Published library on NPM from CI/CD](published_second_version_on_npm.png){width=700}

## What's next

* [Add shield.io badges to your README](https://shields.io/badges/npm-version)
* [Share API documentation for your project using Dokka](https://kotl.in/dokka)
* [Add Renovate to automatically update dependencies](https://docs.renovatebot.com/)
* [Share your library with the community in the `#feed` Kotlin Slack channel](https://kotlinlang.slack.com/)
  (to sign up, visit https://kotl.in/slack)
