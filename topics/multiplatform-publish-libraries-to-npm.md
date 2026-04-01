[//]: # (title: Publish your library to npm – tutorial)

<tldr>
Publish your Kotlin Multiplatform library to npm using the <a href="https://npm-publish.petuska.dev/latest/">npm-publish Gradle plugin</a>,
manually or through GitHub Actions.
</tldr>

To publish your library, you'll need to:

1. Prepare credentials, including [an account on npm](https://docs.npmjs.com/creating-a-new-npm-user-account) and [an access token](https://docs.npmjs.com/creating-and-viewing-access-tokens).
2. Configure the publishing plugin in your Kotlin Multiplatform project.
3. Provide the credentials to the publishing plugin or set up a Trusted Publisher for continuous integration.
4. Run the publication task, either manually or using CI.

In this tutorial, we use GitHub to host the project and run CI via GitHub Actions.

## Sample library

You can use a [sample library project](https://github.com/Kotlin/kotlin-multiplatform-web-library)
to follow along and see a working configuration.

If you reuse the code, make sure to **replace all example values** with values specific to your project.

## Prepare accounts and credentials

To publish to npm, you need to be [signed in at the npm portal](https://www.npmjs.com/login).

In this tutorial, you will need an organization and an access token to configure the manual publishing.

### Create a simple organization

In this tutorial, we publish the library under an npm organization to avoid naming conflicts.

To create a new organization, follow the [npm documentation](https://docs.npmjs.com/creating-an-organization).

### Generate an access token

To publish to npm manually, you need an access token that allows publishing a package under your newly created organization.
To generate such a token, follow the [npm guide](https://docs.npmjs.com/creating-and-viewing-access-tokens).

For this tutorial, use a simplified security configuration:
* Enable the **Bypass two-factor authentication (2FA)** option.
* Set both the general permissions and organization permissions for the token to **Read and write**.

## Configure the library project

If you use the [sample project](https://github.com/Kotlin/kotlin-multiplatform-web-library),
update the default names before publishing.
This includes:

* Name of the library module.
* Name of the project set in the `settings.gradle.kts` file. 

When the names are set, follow the next steps to set up publishing.

### Set up the publishing plugin

This tutorial uses the official [npm-publish plugin](https://github.com/Kotlin/npm-publish)
to help with publishing to npm.
To learn more about the plugin and available configuration options, see the [plugin's documentation](https://npm-publish.petuska.dev).

Add the plugin to your Kotlin Multiplatform project:

1. Open your library module's `build.gradle.kts` file.

2. Add the following line to the `plugins {}` block: 

    ```kotlin
    // <module directory>/build.gradle.kts
    
    plugins {
        kotlin("npm-publish") version "%npmPublishPlugin%"
    }
    ```
    
    > For the latest available version of the plugin, check the [Releases](https://github.com/Kotlin/npm-publish/releases) page.
    > 
    {style="note"}

3. Add the following configuration.
   Make sure to customize the values for your library.
   The only required parameters are `organization`, `authToken`, `packageName`, and `version`.
   The rest is given as an extended example:

    ```kotlin
    // <module directory>/build.gradle.kts
    npmPublish {
        organization = "organization_name_without_the_@_sign"
        
        registries {
            npmjs {
                // You'll pass your npm token as this environment variable
                // when you run the command to publish the package
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

* The `organization` parameter and the `registries {}` block specify authentication details.
  In this case, we use the main npm registry
  and the name of the `NPM_TOKEN` variable that should hold the token when the publishing task is run.
* The `packageName` and `version` parameters define the mandatory package options:
  * The `version` parameter can be omitted to use your module's version as the default value.
  * The `packageName` parameter can be omitted to use the name of your module as the default value.
* The `packageJson {}` block holds various metadata.

## Publish manually

Publishing manually can be useful when you're still experimenting with the project structure,
or want to implement the publishing automation yourself.

Now you can publish the library to npm from the local machine.
To do so, run the following command, pasting the access token you generated earlier in place of `YOUR_ACCESS_TOKEN`:

```bash
NPM_TOKEN=YOUR_ACCESS_TOKEN ./gradlew :shared:publishJsPackageToNpmjsRegistry
```

When the library is published, you should be able to see it in the npm registry.
Open your npm organization page and check the **Packages** tab
(but not on your personal **Packages** page).

![Published library on npm](published_on_npm.png){width=700}

### Troubleshooting

A couple of things that can often go wrong with manual publication:

* Keep track of the `version` field in the `build.gradle.kts` configuration:
  npm fails publication if the package was already published with the same or earlier version.
* When generating a token for an organization-scoped package,
  make sure to set both general **and** organization permissions.

## Publish using continuous integration (CI)

The npm mechanism of Trusted Publishers allows you to quickly set up a CI using OpenID Connect.
This approach avoids generating and maintaining tokens altogether.

In this example, we'll set up a workflow using [GitHub Actions](https://docs.github.com/en/actions).

### Create a GitHub Actions workflow file

Create a `.github/workflows/publish.yml` file that configures the GitHub action:

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
      # Check out the triggering branch
      - name: Check out code
        uses: actions/checkout@v4

      # Set up the JDK to run the Gradle task
      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          distribution: 'zulu'
          java-version: 21

      # Run the publishing Gradle task for the library module
      - name: Publish to npm
        run: ./gradlew :shared:publishJsPackageToNpmjsRegistry
```

Once you commit and push this file into the GitHub repository hosting your project,
the workflow runs whenever you create a GitHub release in that repository.

> You can also configure the workflow to [trigger when a tag is pushed](https://stackoverflow.com/a/61892639) to your repository.
> 
{style="tip"}

### Set up GitHub Actions as your Trusted Publisher

Now that you have a workflow published, you can use the GitHub Action to add a [Trusted Publisher](https://docs.npmjs.com/trusted-publishers)
to your npm package:

1. Open the [published package](#publish-manually) page.
2. Open the **Settings** tab and find the **Trusted Publisher** section.
3. Under **Select your publisher**, click the **GitHub Actions** button.
4. Fill out the form:
   * your GitHub name (or organization)
   * the repository name
   * the name of the workflow file (in this tutorial, we've used [publish.yml](#create-a-github-actions-workflow-file)).
5. Click the **Setup connection** button.

![npm Trusted Publisher setup for GitHub Actions](npm-trusted-publisher-github.png)

> [npm does not verify the provided coordinates](https://docs.npmjs.com/trusted-publishers#troubleshooting),
> so make sure you enter the details correctly.
> 
{style="warning"}

The created connection is then listed in the **Trusted Publishers** section of your package's settings,
which means that the workflow with the specified coordinates is now authorized to publish to npm.

### Create a release on GitHub

With the workflow and the Trusted Publisher connection set up, you're now ready to trigger publishing by [creating a GitHub release](https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository#creating-a-release):

1. Set the package version in the `build.gradle.kts` configuration to the one you want to publish.

   > npm will not allow publishing if the version number is already in use or lower than an already published version.
   > 
   > {style="note"}

2. Go to your GitHub repository.
3. In the right sidebar, click **Releases**.
4. Click the **Draft a new release** button (or the **Create a new release** button if you haven't created a release for
   this repository before).
5. Create or select a Git tag (match the module's version if possible to keep the numbering consistent across systems).
6. Set the release title (it's handy to name the release same as the tag).
   
   To keep track of everything, you may want for the version in the tag to be the same as the version number of the library
   that you specified in the `build.gradle.kts` file.

   ![Create a release on GitHub](create_release_and_tag_for_npm.png){width=700}

7. Click the **Publish release** button.

To check whether the Action was triggered, click the **Actions** tab at the top of your GitHub repository's page.
You should see that the newly published release triggered a run of the publishing workflow.
Click on the workflow to see the logs of the publication task.

When the workflow run is complete, the new version of your package should be listed on your package's page in the npm registry.

![Published library on npm from CI/CD](published_second_version_on_npm.png){width=700}

## What's next

* [Add shield.io badges to your README](https://shields.io/badges/npm-version)
* [Generate API documentation with Dokka](https://kotl.in/dokka)
* [Automate dependency updates with Renovate](https://docs.renovatebot.com/)
* [Share your library with the community in Kotlin Slack](https://kotlinlang.slack.com/)
  (to sign up, visit https://kotl.in/slack)
