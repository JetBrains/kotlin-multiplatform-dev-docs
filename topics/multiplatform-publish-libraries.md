[//]: # (title: Publish your multiplatform library)

This guide helps to publish a Kotlin Multiplatform library to the [Maven Central repository](https://central.sonatype.com/).

To publish your library, you’ll need to:

1. Set up credentials, including an account on Maven Central and a PGP key for signing.
2. Configure the publishing plugin in your library’s project.
3. Provide your credentials to the publishing plugin so it can sign and upload your artifacts.
4. Run the publication task, either locally or using continuous integration.

This guide assumes that you are:

- Creating an open-source library.
- Storing the code for your library in a GitHub repository.
- Using macOS or Linux. If you are a Windows user, use [GnuPG or Gpg4win](https://gnupg.org/download) to generate a key pair.
- Either not registered on Maven Central yet, or have an existing account that’s suitable for [publishing to the Central Portal](https://central.sonatype.org/publish-ea/publish-ea-guide/) (created after March 12th, 2024, or migrated to the Central Portal by their support).
- Using GitHub Actions for continuous integration.

Most of the steps here are still applicable if you’re using a different setup,
but there may be some differences you need to account for.
An [important limitation](https://kotlinlang.org/docs/multiplatform-publish-lib.html#host-requirements) is that Apple targets must be built on a machine with macOS.

## Sample library

Throughout this guide, we’ll use the [fibonacci](https://github.com/kotlin-hands-on/fibonacci) library as an example.
You can refer to the code of that repository to see how the publishing setup works.
If you'd like to reuse the code, you **must replace all example values** with those specific to your project.

## Prepare accounts and credentials

To get started with publishing to Maven Central, sign in (or create a new account) on the [Maven Central](https://central.sonatype.com/) portal.

### Choose and verify a namespace

You’ll need to have a verified namespace that would help to uniquely identify artifacts of your libraries.

Maven artifacts are identified by their coordinates, for example, `com.example:library:1.0.0`.
These coordinates are made up of three parts, separated by colons:

* the `groupId` – `com.example`
* the `artifactId` – `library`
* the `version` – `1.0.0`

Your registered namespace allows you to set the format for your `groupId` on Maven Central:
for example, if you register the `com.example` namespace, you can publish artifacts with the `groupId` set to `com.example`,
`com.example.libraryname`, `com.example.module.feature` and so on.

Once you are signed on Maven Central, navigate to the [Namespaces](https://central.sonatype.com/publishing/namespaces) page.
Then, click the **Add Namespace** button and register your namespace:

<tabs>

<tab id="github" title="With a GitHub repository">

Using your GitHub account to create a namespace is a good option if you don’t own a domain name:

1. Enter `io.github.<your username>` as your namespace, for example, `io.github.kotlinhandson` and click **Submit**.
2. Copy the **Verification Key** displayed under the newly created namespace.
3. On GitHub, log in with the username that you used and create a new public repository with the verification key as the repository’s name.
   For example, `http://github.com/kotlin-hands-on/ex4mpl3c0d`.
4. Navigate back to Maven Central and click the **Verify Namespace** button. When verification succeeds, you can delete the repository you’ve created.

</tab>

<tab id="domain" title="With a domain name">

To use a domain name that you own as your namespace:

1. Enter your domain as the namespace using a reverse-DNS form: If your domain is `example.com`, enter `com.example`.
2. Copy the **Verification Key** displayed.
3. Create a new TXT DNS-record with the verification key as its contents.
   See [Maven Central’s FAQ](https://central.sonatype.org/faq/how-to-set-txt-record/) for more information on how to do this with various domain registrars.
4. Navigate back to Maven Central and click the **Verify Namespace** button. When verification succeeds, you can delete the TXT record you’ve created.

</tab>

</tabs>

#### Generate a key pair

Before you publish something to Maven Central,
you need to sign your artifacts with a [PGP signature](https://central.sonatype.org/publish/requirements/gpg/),
which allows users to validate the origin of artifacts.

To get started with signing, you’ll need to generate a key pair:

* The *private key* is used to sign your artifacts and should never be shared with others.
* The *public key* can be used by others to validate the signature of the artifacts and should be published.

The `gpg` tool that can manage signatures for you is available on the [GnuPG website](https://gnupg.org/download/index.html).
You can also install it using package managers such as [Homebrew](https://brew.sh/):

```bash 
brew install gpg
```

1. Start generating a key pair with the following command and fill in the required details when prompted:

    ```bash
    gpg --full-generate-key
    ```

2. Choose the recommended defaults for the type of key to be created.
    You can leave the selections empty and press Enter to accept the default values.

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

3. When prompted to how long the key should be valid, you can choose the default option of no expiration date.
    If you choose to create a key that automatically expires after a set amount of time,
    you’ll need to [extend its validity](https://central.sonatype.org/publish/requirements/gpg/#dealing-with-expired-keys) when it expires.

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

4. Enter your name, email, and a comment to attach the key to an identity (you can leave the comment empty).

    ```text
    GnuPG needs to construct a user ID to identify your key.

    Real name: Jane Doe
    Email address: janedoe@example.com
    Comment:
    You selected this USER-ID:
        "Jane Doe <janedoe@example.com>"
    ```

5. Enter a passphrase to encrypt the key, which you have to repeat. Keep this passphrase stored securely and privately.
   You’ll be using it later to access the private key to sign the artifacts.

Take a look at the key you’ve created with the following command:

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

You’ll need to use the long alphanumerical identifier of your key displayed here in the following steps.

#### Upload the public key

You need to [upload the public key to a keyserver](https://central.sonatype.org/publish/requirements/gpg/#distributing-your-public-key) for it to be accepted by Maven Central. There are multiple available keyservers, we’ll use `keyserver.ubuntu.com` as a default choice.

Run the following command to upload your public key using `gpg`, **substituting your own key ID** in the parameters:

```bash
gpg --keyserver keyserver.ubuntu.com --send-keys F175482952A225BFC4A07A715EE6B5F76620B385CE
```

#### Export your private key

To let your Gradle project access your private key, you’ll need to export it to a file.
You will be prompted to enter the passphrase you’ve used when creating the key.

Use the following command, **passing in your own key ID** as a parameter:

```bash
gpg --armor --export-secret-keys F175482952A225BFC4A07A715EE6B5F76620B385CE > key.gpg
```

This will create a `key.gpg` file which contains your private key.

> Never share your private key file with anyone — you are the only one who is supposed to have access to it,
> since it allows signing files with your credentials.
> 
{style="warning"}

If you check the contents of the .gpg file, you should see text similar to this:

```text
-----BEGIN PGP PRIVATE KEY BLOCK-----
lQdGBGby2X4BEACvFj7cxScsaBpjty40ehgB6xRmt8ayt+zmgB8p+z8njF7m2XiN
...
bpD/h7ZI7FC0Db2uCU4CYdZoQVl0MNNC1Yr56Pa68qucadJhY0sFNiB23KrBUoiO 
-----END PGP PRIVATE KEY BLOCK-----
```

### Configure the project

#### Prepare your library project

If you started developing your library from a template project, this is a good time to change any default names in the project
to match your own library’s name.
This includes the name of your library module, and the name of the root project in your top-level `build.gradle.kts` file.

If you have an Android target in your project, you should follow the [steps to prepare your Android library release](https://developer.android.com/build/publish-library/prep-lib-release).
This, at a minimum, requires you to [specify an appropriate namespace](https://developer.android.com/build/publish-library/prep-lib-release#choose-namespace)
for your library so that a unique `R` class will be generated when their resources are compiled.
Notice that the namespace is different from the Maven namespace created in the [](#choose-and-verify-a-namespace) section above.

```kotlin
// build.gradle.kts

android {
     namespace = "io.github.kotlinhandson.fibonacci"
}
```

#### Set up the publishing plugin

This guide uses [vanniktech/gradle-maven-publish-plugin](https://github.com/vanniktech/gradle-maven-publish-plugin)
to help with publications to Maven Central.
You can read more about the advantages of the plugin [here](https://vanniktech.github.io/gradle-maven-publish-plugin/#advantages-over-maven-publish).
See the [plugin’s documentation](https://vanniktech.github.io/gradle-maven-publish-plugin/central/)
to learn more about its usage and available configuration options.

To add the plugin to your project, add the following line in the plugins block, in your library module’s `build.gradle.kts` file:

```kotlin
// <module directory>/build.gradle.kts

plugins {
    id("com.vanniktech.maven.publish") version "0.30.0" 
}
```

> For the latest available version of the plugin, check its [releases page](https://github.com/vanniktech/gradle-maven-publish-plugin/releases).
> 
{style="note"}

In the same file, add the following configuration. Customize all these values appropriately for your library.

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

> It’s also possible to use Gradle properties to configure this instead.
> 
{style="tip"}

The most important settings here are:

* The `coordinates`, which specify the `groupId`, `artifactId`, and `version` of your library.
* The [license](https://central.sonatype.org/publish/requirements/#license-information) that you’re publishing your library under.
* The [developer information](https://central.sonatype.org/publish/requirements/#developer-information) which lists the authors of the library.
* [SCM (Source Code Management) information](https://central.sonatype.org/publish/requirements/#scm-information),
  which specifies where the sources of your library are available.

### Publish to Maven Central using continuous integration

### Generate the user token

You need a Maven access token for Maven Central to authorize your publishing requests.
Open the [Setup Token-Based Authentication](https://central.sonatype.com/account) page and click the **Generate User Token** button.

The output looks like the example below, containing a username and a password.
If you lose these credentials, you’ll need to generate new ones later — they are not stored by Maven Central.

```xml
<server>
    <id>${server}</id>
    <username>l2nfaPmz</username>
    <password>gh9jT9XfnGtUngWTZwTu/8141keYdmQpipqLPRKeDLTh</password>
</server>
```

### Add secrets to GitHub

To use the keys and credentials required for publication in your GitHub Action workflow while keeping them private,
you need to place those values into secrets.

1. On your GitHub repository **Settings** page, click **Security** | **Secrets and variables** | **Actions**.
2. Click the `New repository secret` button and add the following secrets:

    * `MAVEN_CENTRAL_USERNAME` and `MAVEN_CENTRAL_PASSWORD` are the values generated as the User Token by the Central Portal website in the [Generate User Token](#generate-the-user-token) section.
    * `SIGNING_KEY_ID` is **the last 8 characters** of your signing key’s identifier, for example, `20B385CE` for `F175482952A225BFC4A07A715EE6B5F76620B385CE`
    * `SIGNING_PASSWORD` is the passphrase you’ve provided when generating your GPG key.
    * `GPG_KEY_CONTENTS` should contain the entire contents of your `key.gpg` file, which you’ve created earlier
        in the [Export your private key](#export-your-private-key) section.

![](../images/publish/github_secrets.png)

You will use the names for these secrets in the CI configuration on the next step.

#### Add a GitHub Actions workflow to your project

You can set up continuous integration which builds and publishes your library automatically.
We’ll use [GitHub Actions](https://docs.github.com/en/actions) as an example.

To get started, add the following workflow to your repository, in the `.github/workflows/publish.yml` file.

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

After committing and pushing this file, the workflow will run automatically when you create a release (including a pre-release)
in the GitHub repository hosting your project.
It checks out the current version of your code, sets up a JDK, and then runs the `publishToMavenCentral` Gradle task.

When using the `publishToMavenCentral` task, you’ll still need to check and release your deployment manually on the Maven Central website,
as described in the [](#create-a-release-on-github) section.
You may use `publishAndReleaseToMavenCentral` instead for a fully automated release.

You can also configure the workflow to [trigger when a tag is pushed](https://stackoverflow.com/a/61892639) to your repository.

> The script above disables Gradle [configuration cache](https://docs.gradle.org/current/userguide/configuration_cache.html)
> for the publication task by adding `--no-configuration-cache` to the Gradle command,
> as the publication plugin does not support it (see this [open issue](https://github.com/gradle/gradle/issues/22779)).
>
{style="tip"}

This action will need your signing details and your Maven Central credentials, which you created as repository secrets
in the [](#add-secrets-to-github) section.

The configuration of the workflow transfers the secrets into environment variables,
which will make them available to the Gradle build automatically.

#### Create a release on GitHub

With the workflow and secrets set up, you’re now ready to [create a release](https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository#creating-a-release)
that will trigger the publication of your library.

1. Make sure that the version number specified in the `build.gradle.kts` file for your library is the one you would like to publish.
2. Go to your GitHub repository’s main page.
3. In the right sidebar click **Releases**.

4. Click the **Draft a new release** button (or the **Create a new release** button if you haven't created a release for this repository before).

5. Each release creates a new tag. Create a new tag in the tags dropdown, and set the release title (these may be identical).
   You probably want these to be the same as the version number of the library that you specified in the `build.gradle.kts` file.

    ![](../images/publish/create_release_and_tag.png)

6. Double-check the branch you want to target with the release (especially if you want to release from a branch that’s different from your default),
    and add appropriate release notes for your new version.

7. The checkboxes below the description allow you to mark a release as a pre-release (useful for early access – alpha, beta, or RC versions of a library).
   You can also mark the release as the latest one (if you already made a release for this repository before):

8. Click the **Publish release** button to create the new release.

9. Click the **Actions** tab at the top of your GitHub repository's page.
    Here you’ll see that the new release triggered your publishing workflow.
    Click on it to see the outputs of the publication task.

10. After the workflow run completes successfully, navigate to the [Deployments](https://central.sonatype.com/publishing/deployments)
     dashboard on Maven Central.
     You should see a new deployment here. This deployment can be in the *pending* or *validating* states for some time
     while Maven Central performs checks on it.

Once your deployment is in the *validated* state, you should check that it contains all the artifacts you’ve uploaded.
If everything looks correct, click the *Publish* button to release these artifacts.

![](../images/publish/published_on_maven_central.png)

> It will take some time (usually about 15–30 minutes) after the release for the artifacts to be available publicly in the Maven Central repository.
> It may take some more time before they are indexed to be found on [the Maven Central website](https://central.sonatype.com/).

To release the artifacts automatically once the deployment is verified, replace the `publishToMavenCentral` task in your workflow
with `publishAndReleaseToMavenCentral`.

## Next steps
- Share your library with the Kotlin Community in the `#feed` channel in the [Kotlin Slack](https://kotlinlang.slack.com/).
    (To sign up visit https://kotl.in/slack.)
- Add [shield.io badges](https://shields.io/badges/maven-central-version) to your README.
- Create a documentation site for your project using [Writerside](https://www.jetbrains.com/writerside/).
- Share API documentation for your project using [Dokka](https://kotl.in/dokka).
- Add [Renovate](https://docs.renovatebot.com/) to automatically update dependencies.