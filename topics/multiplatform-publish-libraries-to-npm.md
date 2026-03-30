[//]: # (title: Publish your library to npm – tutorial)

<tldr>
This tutorial explains how to publish your Kotlin Multiplatform library targeting JS or WasmJS as an npm package.
</tldr>

To publish your library, you'll need to:

1. Prepare credentials, including [an account on npm](https://docs.npmjs.com/creating-a-new-npm-user-account) and [an access token](https://docs.npmjs.com/creating-and-viewing-access-tokens).
2. Configure the publishing plugin in your Kotlin Multiplatform project.
3. Provide the credentials to the publishing plugin or set up a Trusted Publisher for continuous integration. (and one-time password if two-factor authentication is enabled) to the publishing plugin so it can upload your artifacts.
4. Run the publication task, either locally or using CI.

In this tutorial, we use GitHub to host the project and run CI via GitHub Actions.

## Sample library

You can use a [sample library project](https://github.com/Kotlin/kotlin-multiplatform-web-library)
to follow along and see a working configuration.

If you reuse the code, make sure to **replace all example values** with values specific to your project.

## Prepare accounts and credentials

To publish to npm, you need to be [signed in at the npm portal](https://www.npmjs.com/login).

### Create a simple organization

In this tutorial, we publish the library under an npm organization to avoid naming conflicts.

To create a new organization, follow the [npm documentation](https://docs.npmjs.com/creating-an-organization).

#### Generate an access token

To publish to npm manually, you need an access token that allows publishing a package under your newly created organization.
To generate such a token, follow the [npm guide](https://docs.npmjs.com/creating-and-viewing-access-tokens).

For this tutorial, use a simplified security configuration:
* Enable the **Bypass two-factor authentication (2FA)** option.
* Set both the general permissions and organization permissions for the token to **Read and write**.

As soon as you have the organization name and the access token, you're ready to publish your library as an npm package.

## Configure the library project

If you started developing your library from the [sample project](https://github.com/Kotlin/kotlin-multiplatform-web-library),
now is a good time to change any default names if you'd like.
This includes the name of your library module and the name of the root project in your top-level `settings.gradle.kts` file.

### Set up the publishing plugin

This tutorial uses the official [npm-publish plugin](https://github.com/Kotlin/npm-publish)
to help with publishing to npm.
If you'd like to learn more about the plugin and available configuration options, see the [plugin's documentation](https://npm-publish.petuska.dev).

Add the plugin to your Kotlin Multiplatform project:

1. Add the following line to the `plugins {}` block of your library module's `build.gradle.kts` file:

    ```kotlin
    // <module directory>/build.gradle.kts
    
    plugins {
        kotlin("npm-publish") version "%npmPublishPlugin%"
    }
    ```
    
    > For the latest available version of the plugin, check the [Releases](https://github.com/Kotlin/npm-publish/releases) page.
    > 
    {style="note"}

2. In the same file, add the following configuration, making sure to customize all the values for your library:

    ```kotlin
    // <module directory>/build.gradle.kts
    npmPublish {
        organization = "organization_name_without_the_@_sign"
        
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
                    contributors = listOf(
                        Person {
                            name = "John Smith"
                            email = "john.smith@example.com"
                            url = "https://github.com/johnsmith"
                        },
                    )
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

The important settings in the `npmPublish {}` block are:

* The `registries {}` block specifies the registry where the library should be published.
  For manual publishing, you'll specify the value of the `NPM_TOKEN` variable when running the publishing task.
* The `organization`, `packageName` and `version` parameters define the corresponding values for the package:
  * The `organization` parameter is optional unless you'd like to publish an organization-scoped package.
  * The `packageName` parameter can be omitted to use the name of your module as the default value.
  * The `version` parameter can be omitted to use the module version as the default value.
* The `license` parameter specifies the license under which your library should be published.
* The `author {}` block describes the author of the library, and the `contributors` collection holds the contributors' contact info.
* The `repository {}` block specifies where the library's source code is hosted.
* The `readme` parameter specifies which file should be used as the library's README.

### Publish manually

Now you can publish the library to npm from the local machine.
To do so, run the following command, pasting the access token you generated earlier in place of `YOUR_ACCESS_TOKEN`:

```bash
NPM_TOKEN=YOUR_ACCESS_TOKEN ./gradlew :shared:publishJsPackageToNpmjsRegistry
```

When the library is published, you should be able to see it in the npm registry.
Open your nmp organization page — it should be present in the **Packages** tab
(but not on your personal **Packages** page).

![Published library on npm](published_on_npm.png){width=700}

## Publish to npm using continuous integration

You can also set up continuous integration to build and publish your library automatically.
We'll use [GitHub Actions](https://docs.github.com/en/actions) as an example.

This requires both setting up a GitHub workflow and authorizing GitHub to publish packages with your npm account.

### Add a GitHub Actions workflow to your project

To get started, you need a workflow that configures the GitHub action, for example:

```yaml
# .github/workflows/publish.yml

name: Publish

on:
  release:
    types: [released, prereleased]

permissions:
  id-token: write  # Required for GitHub Actions
                   # to integrate with npm trusted publishing
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

      - name: Publish to npm
        run: ./gradlew :shared:publishJsPackageToNpmjsRegistry
```

Once you commit and push this file into the GitHub repository hosting your project,
the workflow will run automatically whenever you create a release (including a pre-release)
in that repository.
The workflow checks out the current version of your code, sets up a JDK,
and then runs the `publishJsPackageToNpmjsRegistry` Gradle task.

You can also configure the workflow to [trigger when a tag is pushed](https://stackoverflow.com/a/61892639) to your repository.

### Add a Trusted Publisher for your npm package

Now that you have a workflow published, you can use the GitHub Action as a [Trusted Publisher](https://docs.npmjs.com/trusted-publishers) on npm:

1. Open the page of the package you [published manually](#publish-manually).
2. Navigate to the **Settings** tab.
3. Under "Select your publisher", click the **GitHub Actions** button.
4. Enter your GitHub name (or organization) and the repository name.
5. Enter the name of the workflow file (in this tutorial, we've used [publish.yml](#add-a-github-actions-workflow-to-your-project)).
6. Click the **Setup connection** button.

> [npm does not verify the provided coordinates](https://docs.npmjs.com/trusted-publishers#troubleshooting),
> so make sure you enter the details correctly.
> 
{style="warning"}

The created connection should be listed in the **Trusted Publishers** section of your package's settings,
which means that the workflow with the specified coordinates is now authorized to publish to npm.

### Create a release on GitHub

With the workflow and secrets set up, you're now ready to trigger publishing by [creating a GitHub release](https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository#creating-a-release):

1. Make sure that the version number specified in the `build.gradle.kts` file for your library is the one you would
   like to publish (if you've already published a version of your library, please update the version number).

   > npm will not allow publishing if the version number is already in use, or lower than an already published version.
   > 
   > {style="note"}

2. Go to your GitHub repository's main page.
3. In the right sidebar, click **Releases**.
4. Click the **Draft a new release** button (or the **Create a new release** button if you haven't created a release for
   this repository before).
5. Each release needs to have a corresponding Git tag: you can create a new tag immediately in the tag dropdown
6. and set the release title (the tag name and the title can be identical).
   
   To keep track of everything, you may want for the version in the tag to be the same as the version number of the library
   that you specified in the `build.gradle.kts` file.

   ![Create a release on GitHub](create_release_and_tag_for_npm.png){width=700}

7. Click the **Publish release** button to create the new release.
8. Click the **Actions** tab at the top of your GitHub repository's page.
   You should see that the newly published release triggered a run of the publishing workflow.
    
   Click on the workflow to see the logs of the publication task.
9. After the workflow run is complete, the new version of your package should be listed on your package's page in the npm registry.

![Published library on npm from CI/CD](published_second_version_on_npm.png){width=700}

## What's next

* [Add shield.io badges to your README](https://shields.io/badges/npm-version)
* [Share API documentation for your project using Dokka](https://kotl.in/dokka)
* [Add Renovate to automatically update dependencies](https://docs.renovatebot.com/)
* [Share your library with the community in the `#feed` Kotlin Slack channel](https://kotlinlang.slack.com/)
  (to sign up, visit https://kotl.in/slack)
