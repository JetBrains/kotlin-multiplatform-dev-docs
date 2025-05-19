[//]: # (title: Publish your library to Maven Central – tutorial)

In this tutorial, you'll learn how to publish your Kotlin Multiplatform library to the [Maven Central](https://central.sonatype.com/)
repository.

To publish your library, you'll need to:

1. Set up credentials, including an account on Maven Central and a PGP key for signing.
2. Configure the publishing plugin in your library's project.
3. Provide your credentials to the publishing plugin so it can sign and upload your artifacts.
4. Run the publication task, either locally or using continuous integration.

This tutorial assumes that you are:

* Creating an open-source library.
* Storing the code for your library in a GitHub repository.
* Using macOS or Linux. If you are a Windows user, use [GnuPG or Gpg4win](https://gnupg.org/download) to generate a key pair.
* Either not registered on Maven Central yet or have an existing account that's suitable for [publishing to the Central Portal](https://central.sonatype.org/publish-ea/publish-ea-guide/)
  (created after March 12th, 2024, or migrated to the Central Portal by their support).
* Using GitHub Actions for continuous integration.

> Most of the steps here are still applicable if you're using a different setup, but there may be some differences you
> need to take into account.
> 
> An [important limitation](multiplatform-publish-lib-setup.md#host-requirements)
> is that Apple targets must be built on a machine with macOS.
> 
{style="note"}

## Sample library

In this tutorial, you'll use the [fibonacci](https://github.com/Kotlin/multiplatform-library-template/) library as an example.
You can refer to the code of that repository to see how the publishing setup works.

If you'd like to reuse the code, you **must replace all example values** with those specific to your project.

## Prepare accounts and credentials

To get started with publishing to Maven Central, sign in (or create a new account) on the [Maven Central](https://central.sonatype.com/)
portal.

### Choose and verify a namespace

You'll need a verified namespace to uniquely identify your library's artifacts on Maven Central.

Maven artifacts are identified by their [coordinates](https://central.sonatype.org/publish/requirements/#correct-coordinates),
for example, `com.example:fibonacci-library:1.0.0`. These coordinates are made up of three parts, separated by colons:

* `groupId` in the reverse-DNS form, for example, `com.example`
* `artifactId`: the unique name of the library itself, for example, `fibonacci-library`
* `version`: the version string, for example, `1.0.0`. The version can be any string, but it cannot end in `-SNAPSHOT`

Your registered namespace allows you to set the format for your `groupId` on Maven Central. For example, if you register
the `com.example` namespace, you can publish artifacts with the `groupId` set to `com.example`,
`com.example.libraryname`, `com.example.module.feature`, and so on.

Once you are signed on Maven Central, navigate to the [Namespaces](https://central.sonatype.com/publishing/namespaces) page.
Then, click the **Add Namespace** button and register your namespace:

<tabs>
<tab id="github" title="With a GitHub repository">

Using your GitHub account to create a namespace is a good option if you don't own a domain name:

1. Enter `io.github.<your username>` as your namespace, for example, `io.github.kotlinhandson` and click **Submit**.
2. Copy the **Verification Key** displayed under the newly created namespace.
3. On GitHub, log in with the username that you used and create a new public repository with the verification key as
   the repository's name, for example, `http://github.com/kotlin-hands-on/ex4mpl3c0d`.
4. Navigate back to Maven Central and click the **Verify Namespace** button. When verification succeeds, you can delete
   the repository you've created.

</tab>
<tab id="domain" title="With a domain name">

To use a domain name that you own as your namespace:

1. Enter your domain as the namespace using the reverse-DNS form. If your domain is `example.com`, enter `com.example`.
2. Copy the **Verification Key** displayed.
3. Create a new TXT DNS-record with the verification key as its contents.

   See [Maven Central's FAQ](https://central.sonatype.org/faq/how-to-set-txt-record/) for more information on how to do
   this with various domain registrars.
4. Return to Maven Central and click the **Verify Namespace** button. When verification succeeds, you can delete
   the TXT record you've created.

</tab>
</tabs>

#### Generate a key pair

Before you publish something to Maven Central, you need to sign your artifacts with a [PGP signature](https://central.sonatype.org/publish/requirements/gpg/),
which allows users to validate the origin of artifacts.

To get started with signing, you'll need to generate a key pair:

* The _private key_ is used to sign your artifacts and should never be shared with others.
* The _public key_ can be shared with others so they can validate the signature of your artifacts.

The `gpg` tool that can manage signatures for you is available on the [GnuPG website](https://gnupg.org/download/index.html).
You can also install it using package managers such as [Homebrew](https://brew.sh/):

```bash 
brew install gpg
```

1. Start generating a key pair using the following command and provide the required details when prompted:

    ```bash
    gpg --full-generate-key
    ```

2. Choose the recommended defaults for the type of key to be created.
   You can leave the selections empty and press **Enter** to accept the default values.

    ```text
    Please select what kind of key you want:
        (1) RSA and RSA
        (2) DSA and Elgamal
        (3) DSA (sign only)
        (4) RSA (sign only)
        (9) ECC (sign and encrypt) *default*
        (10) ECC (sign only)
        (14) Existing key from card
    Your selection? 9
    
    Please select which elliptic curve you want:
        (1) Curve 25519 *default*
        (4) NIST P-384
        (6) Brainpool P-256
    Your selection? 1
    ```

    > At the time of writing, this is `ECC (sign and encrypt)` with `Curve 25519`.
    > Older versions of `gpg` might default to `RSA` with a `3072` bit key size.
    > 
    {style="note"}

3. When prompted to specify how long the key should be valid, you can choose the default option of no expiration date.

   If you choose to create a key that automatically expires after a set amount of time, you'll need to [extend its validity](https://central.sonatype.org/publish/requirements/gpg/#dealing-with-expired-keys)
   when it expires.

    ```text
    Please specify how long the key should be valid.
        0 = key does not expire
        <n>  = key expires in n days
        <n>w = key expires in n weeks
        <n>m = key expires in n months
        <n>y = key expires in n years
    Key is valid for? (0) 0
    Key does not expire at all
    
    Is this correct? (y/N) y
    ```

4. Enter your name, email, and an optional comment to associate the key with an identity (you can leave the comment field
   empty):

    ```text
    GnuPG needs to construct a user ID to identify your key.

    Real name: Jane Doe
    Email address: janedoe@example.com
    Comment:
    You selected this USER-ID:
        "Jane Doe <janedoe@example.com>"
    ```

5. Enter a passphrase to encrypt the key and repeat it when prompted.

   Keep this passphrase stored securely and privately. You'll need it later to access the private key when signing artifacts.

6. Take a look at the key you've created with the following command:

   ```bash
   gpg --list-keys
   ```

The output will look something like this:

```text
pub   ed25519 2024-10-06 [SC]
      F175482952A225BFD4A07A713EE6B5F76620B385CE
uid   [ultimate] Jane Doe <janedoe@example.com>
      sub   cv25519 2024-10-06 [E]
```

In the next steps, you'll need to use the long alphanumerical identifier of your key that appears in the output.

#### Upload the public key

You need to [upload the public key to a keyserver](https://central.sonatype.org/publish/requirements/gpg/#distributing-your-public-key)
for it to be accepted by Maven Central. There are multiple available keyservers, let's use `keyserver.ubuntu.com` as a
default choice.

Run the following command to upload your public key using `gpg`, **substituting your own key ID** in the parameters:

```bash
gpg --keyserver keyserver.ubuntu.com --send-keys F175482952A225BFC4A07A715EE6B5F76620B385CE
```

#### Export your private key

To let your Gradle project access your private key, you'll need to export it to a file.
You'll be prompted to enter the passphrase you've used when creating the key.

Use the following command, **passing in your own key ID** as a parameter:

```bash
gpg --armor --export-secret-keys F175482952A225BFC4A07A715EE6B5F76620B385CE > key.gpg
```

This command will create a `key.gpg` file which contains your private key.

> Never share your private key file with anyone – only you should have access to it
> since the private key enables signing files with your credentials.
> 
{style="warning"}

If you check the contents of the `.gpg` file, you should see text similar to this:

```text
-----BEGIN PGP PRIVATE KEY BLOCK-----
lQdGBGby2X4BEACvFj7cxScsaBpjty40ehgB6xRmt8ayt+zmgB8p+z8njF7m2XiN
...
bpD/h7ZI7FC0Db2uCU4CYdZoQVl0MNNC1Yr56Pa68qucadJhY0sFNiB23KrBUoiO 
-----END PGP PRIVATE KEY BLOCK-----
```

## Configure the project

### Prepare your library project

If you started developing your library from a template project, now is a good time to change any default names in the
project to match your own library's name. This includes the name of your library module and the name of the root project
in your top-level `build.gradle.kts` file.

If you have an Android target in your project, you should follow the [steps to prepare your Android library release](https://developer.android.com/build/publish-library/prep-lib-release).
At a minimum, this process requires you to [specify an appropriate namespace](https://developer.android.com/build/publish-library/prep-lib-release#choose-namespace)
for your library so that a unique `R` class will be generated when their resources are compiled.
Notice that the namespace is different from the Maven namespace [created earlier](#choose-and-verify-a-namespace).

```kotlin
// build.gradle.kts

android {
    namespace = "io.github.kotlinhandson.fibonacci"
}
```

### Set up the publishing plugin

This tutorial uses [vanniktech/gradle-maven-publish-plugin](https://github.com/vanniktech/gradle-maven-publish-plugin)
to help with publications to Maven Central.
You can read more about the advantages of the plugin [here](https://vanniktech.github.io/gradle-maven-publish-plugin/#advantages-over-maven-publish).
See the [plugin's documentation](https://vanniktech.github.io/gradle-maven-publish-plugin/central/)
to learn more about its usage and available configuration options.

To add the plugin to your project, add the following line to the `plugins {}` block of your library module's `build.gradle.kts` file:

```kotlin
// <module directory>/build.gradle.kts

plugins {
    id("com.vanniktech.maven.publish") version "0.30.0" 
}
```

> For the latest available version of the plugin, check its [Releases page](https://github.com/vanniktech/gradle-maven-publish-plugin/releases).
> 
{style="note"}

In the same file, add the following configuration, making sure to customize all the values for your library:

```kotlin
// <module directory>/build.gradle.kts

mavenPublishing {
    publishToMavenCentral(SonatypeHost.CENTRAL_PORTAL)
    
    signAllPublications()
    
    coordinates(group.toString(), "fibonacci", version.toString())
    
    pom { 
        name = "Fibonacci library"
        description = "A mathematics calculation library."
        inceptionYear = "2024"
        url = "https://github.com/kotlin-hands-on/fibonacci/"
        licenses {
            license {
                name = "The Apache License, Version 2.0"
                url = "https://www.apache.org/licenses/LICENSE-2.0.txt"
                distribution = "https://www.apache.org/licenses/LICENSE-2.0.txt"
            }
        }
        developers {
            developer {
                id = "kotlin-hands-on"
                name = "Kotlin Developer Advocate"
                url = "https://github.com/kotlin-hands-on/"
            }
        }
        scm {
            url = "https://github.com/kotlin-hands-on/fibonacci/"
            connection = "scm:git:git://github.com/kotlin-hands-on/fibonacci.git"
            developerConnection = "scm:git:ssh://git@github.com/kotlin-hands-on/fibonacci.git"
        }
    }
}
```

> To configure this, you can also use [Gradle properties](https://docs.gradle.org/current/userguide/build_environment.html).
> 
{style="tip"}

The most important settings here are:

* The `coordinates`, which specify the `groupId`, `artifactId`, and `version` of your library.
* The [license](https://central.sonatype.org/publish/requirements/#license-information), under which your library is published.
* The [developer information](https://central.sonatype.org/publish/requirements/#developer-information), which lists
  the authors of the library.
* [SCM (Source Code Management) information](https://central.sonatype.org/publish/requirements/#scm-information),
  which specifies where the library's source code is hosted.

## Publish to Maven Central using continuous integration

### Generate the user token

You need a Maven access token for Maven Central to authorize your publishing requests.
Open the [Setup Token-Based Authentication](https://central.sonatype.com/account) page and click the **Generate User Token** button.

The output looks like the example below, containing a username and a password.
If you lose these credentials, you'll need to generate new ones later, as they are not stored by Maven Central.

```xml
<server>
    <id>${server}</id>
    <username>l2nfaPmz</username>
    <password>gh9jT9XfnGtUngWTZwTu/8141keYdmQpipqLPRKeDLTh</password>
</server>
```

### Add secrets to GitHub

To use the keys and credentials required for publication in your GitHub Action workflow while keeping them private,
you need to store these values as secrets.

1. On your GitHub repository **Settings** page, click **Security** | **Secrets and variables** | **Actions**.
2. Click the `New repository secret` button and add the following secrets:

   * `MAVEN_CENTRAL_USERNAME` and `MAVEN_CENTRAL_PASSWORD` are the values [generated for the User Token](#generate-the-user-token)
     by the Central Portal website.
   * `SIGNING_KEY_ID` is **the last 8 characters** of your signing key's identifier, for example, `20B385CE` for
     `F175482952A225BFC4A07A715EE6B5F76620B385CE`.
   * `SIGNING_PASSWORD` is the passphrase you've provided when generating your GPG key.
   * `GPG_KEY_CONTENTS` should contain the entire contents of [your `key.gpg` file](#export-your-private-key).

   ![Add secrets to GitHub](github_secrets.png){width=700}

You'll use the names for these secrets in the CI configuration in the next step.

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
jobs:
  publish:
    name: Release build and publish
    runs-on: macOS-latest
    steps:
      - name: Check out code
        uses: actions/checkout@v4
      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          distribution: 'zulu'
          java-version: 21
      - name: Publish to MavenCentral
        run: ./gradlew publishToMavenCentral --no-configuration-cache
        env:
          ORG_GRADLE_PROJECT_mavenCentralUsername: ${{ secrets.MAVEN_CENTRAL_USERNAME }}
          ORG_GRADLE_PROJECT_mavenCentralPassword: ${{ secrets.MAVEN_CENTRAL_PASSWORD }}
          ORG_GRADLE_PROJECT_signingInMemoryKeyId: ${{ secrets.SIGNING_KEY_ID }}
          ORG_GRADLE_PROJECT_signingInMemoryKeyPassword: ${{ secrets.SIGNING_PASSWORD }}
          ORG_GRADLE_PROJECT_signingInMemoryKey: ${{ secrets.GPG_KEY_CONTENTS }}
```

Once you commit and push this file, the workflow will run automatically whenever you create a release (including a pre-release)
in the GitHub repository hosting your project. The workflow checks out the current version of your code, sets up a JDK,
and then runs the `publishToMavenCentral` Gradle task.

When using the `publishToMavenCentral` task, you'll still need to check and [release your deployment manually](#create-a-release-on-github)
on the Maven Central website. Alternatively, you can use the `publishAndReleaseToMavenCentral` task to fully automate
the release process.

You can also configure the workflow to [trigger when a tag is pushed](https://stackoverflow.com/a/61892639) to your repository.

> The script above disables Gradle [configuration cache](https://docs.gradle.org/current/userguide/configuration_cache.html)
> for the publication task by adding `--no-configuration-cache` to the Gradle command,
> as the publication plugin does not support it (see this [open issue](https://github.com/gradle/gradle/issues/22779)).
>
{style="tip"}

This action requires your signing details and your Maven Central credentials, which you created as [repository secrets](#add-secrets-to-github).

The workflow configuration automatically transfers these secrets into environment variables,
making them available to the Gradle build process.

### Create a release on GitHub

With the workflow and secrets set up, you're now ready to [create a release](https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository#creating-a-release)
that will trigger the publication of your library.

1. Make sure that the version number is specified in the `build.gradle.kts` file for your library is the one you would
   like to publish.
2. Go to your GitHub repository's main page.
3. In the right sidebar, click **Releases**.
4. Click the **Draft a new release** button (or the **Create a new release** button if you haven't created a release for
   this repository before).
5. Each release has a tag. Create a new tag in the tag dropdown, and set the release title (the tag name and
   the title can be identical).
   
   You probably want these to be the same as the version number of the library that you specified in the `build.gradle.kts` file.

   ![Create a release on GitHub](create_release_and_tag.png){width=700}

6. Double-check the branch you want to target with the release (especially if it's not the default branch) and add
   appropriate release notes for your new version.
7. Use the checkboxes below the description to mark a release as a pre-release (useful for early access versions such as
   alpha, beta, or RC).
   
   You can also mark the release as the latest one (if you already made a release for this repository before).
8. Click the **Publish release** button to create the new release.
9. Click the **Actions** tab at the top of your GitHub repository's page. Here, you'll see that the new release
   triggered your publishing workflow.
    
   You can click on the workflow to see the outputs of the publication task.
10. After the workflow run is complete, navigate to the [Deployments](https://central.sonatype.com/publishing/deployments)
    dashboard on Maven Central. You should see a new deployment here.

    This deployment may remain in the _pending_ or _validating_ states for some time while Maven Central performs checks.

11. Once your deployment is in the _validated_ state, check that it contains all the artifacts you've uploaded.
    If everything looks correct, click the **Publish** button to release these artifacts.

    ![Publishing settings](published_on_maven_central.png){width=700}

    > It will take some time (usually about 15–30 minutes) after the release for the artifacts to be available publicly in
    > the Maven Central repository. It may take longer for them to be indexed and become searchable on the [Maven Central website](https://central.sonatype.com/).
    >
    {style="tip"}

To release the artifacts automatically once the deployment is verified, replace the `publishToMavenCentral` task in your
workflow with `publishAndReleaseToMavenCentral`.

## What's next

* [Learn more about setting up multiplatform library publication and requirements](multiplatform-publish-lib-setup.md)
* [Add shield.io badges to your README](https://shields.io/badges/maven-central-version)
* [Share API documentation for your project using Dokka](https://kotl.in/dokka)
* [Add Renovate to automatically update dependencies](https://docs.renovatebot.com/)
* [Promote your library on the JetBrains' search platform](https://klibs.io/)
* [Share your library with the community in the `#feed` Kotlin Slack channel](https://kotlinlang.slack.com/)
  (to sign up, visit https://kotl.in/slack)
