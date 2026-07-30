[//]: # (title: Configure an iOS delivery pipeline for your Kotlin Multiplatform project)

In this tutorial, you'll learn how to set up TeamCity Cloud for a Kotlin Multiplatform project with an iOS target.
You'll create a CI pipeline that builds and tests your iOS app on a hosted macOS agent,
configure Apple signing credentials, upload a build to TestFlight, and automate the process to start a new build
on every push to your GitHub repository.

The [Kotlin Multiplatform IDE plugin](https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform) walks you through
the entire setup directly from your IDE — 
from connecting your project to TeamCity Cloud to publishing your iOS application.

With this workflow, TeamCity Cloud:

* Provides hosted macOS agents, so you don't need to provision a Mac yourself.
* Generates the required pipeline configuration for you.
* Signs your builds and publishes them to [TestFlight](https://developer.apple.com/testflight/).
* Provides a free starting tier with a monthly allowance of build minutes and storage.

## Create the TeamCity pipeline

### Start CI setup from the IDE

1. Commit and push your project changes. If CI has not been configured yet,
   the Kotlin Multiplatform IDE plugin displays a tooltip prompting you to start CI setup.
2. Click **Configure CI**.
   ![The Configure CI tooltip appears after pushing changes](ios-pipeline-Configure-CI.png){width=450 style="block"}
   The plugin generates the files required to build and test your iOS app on a hosted macOS agent.
   These files are added to your local project but are not committed.
3. If the TeamCity plugin is not installed, the IDE prompts you to install it.
   Click **Install TeamCity plugin**, then return to the setup flow.
   ![Install plugin to set up CI/CD](ios-pipeline-install-plugin.png){width=700 style="block"}

<!-- TODO: Add link to marketplace
    > You can also install the [Plugin Name](link) plugin from JetBrains Marketplace directly.
    >
    {style="note"}
-->

4. When the IDE displays **Pipeline is ready**, the initial pipeline configuration is complete. 
   Click **Continue** and, when prompted, allow the IDE to add the generated files to Git. 
   These files remain local until you commit them to your repository.

### Create or connect a TeamCity Cloud workspace

TeamCity requires a Cloud workspace to run builds on hosted macOS agents.

1. Review and accept the TeamCity Cloud Terms of Service.
2. Click **Create free account** to create a TeamCity Cloud account.

    > TeamCity Cloud is hosted by JetBrains and provides a free starting tier.
    > The free tier includes a monthly allowance of build minutes and storage,
    > which is enough to get started and test this pipeline at no cost.
    > You can upgrade your plan at any time as your CI/CD needs grow.
    >
    {style="note"}

3. In the browser, click **Authorize JetBrains** to authorize the integration.

TeamCity creates or connects to the workspace and prepares the build environment.
This usually takes less than 30 seconds.

## Build the iOS app

When the workspace is ready, the IDE automatically opens the **TeamCity** tab and starts your first build.

For this first run, TeamCity uses your local, uncommitted project files. 
It builds the iOS application and runs the configured tests on a hosted macOS agent. 
This lets you verify that the pipeline works as expected before you commit any generated files to your repository.

![iOS build succeeded](ios-pipeline-first-build.png){width=700 style="block"}

When the automated build succeeds, click **Publish to TestFlight** to configure signing and deployment.

## Configure Apple signing and TestFlight

To upload builds to TestFlight, TeamCity needs credentials for App Store Connect and Apple code signing.

### Create an App Store Connect API key

1. Sign in to [App Store Connect](https://appstoreconnect.apple.com/).
2. Go to **Users and Access** and select **Keys**.
3. Create an API key with the permissions required to upload the app.
4. Download the private key (`.p8` file).
5. Note the **Issuer ID** and **Key ID** displayed for the key.

You can download the `.p8` file only once, so store it securely.

### Export an Apple Distribution certificate

1. In Xcode, Go to **Settings** | **Accounts**, or open the [Apple Developer Portal](https://developer.apple.com/account/),
   and then create or locate your Apple Distribution certificate.
2. On a Mac, open **Keychain Access** and find the certificate under **My Certificates**.
3. Right-click the certificate and choose **Export** to save it as a `.p12` file.
4. Set an export password and store it securely; you'll need it later.

> If you do not have access to a Mac, you can 
> [generate](https://www.ssl.com/how-to/manually-generate-a-certificate-signing-request-csr-using-openssl/)
> a Certificate Signing Request (CSR) and 
> [convert](https://www.ssl.com/how-to/create-a-pfx-p12-certificate-file-using-openssl/) the resulting 
> certificate to a `.p12` file using OpenSSL on Windows or Linux.
>
{style="note"}

### Add Apple credentials in the IDE

Return to the IDE and complete the **Add Apple signing credentials** form.
TeamCity stores these values as secure deployment credentials; they are not added to your project source files.

|-------------------|----------------------------------------------------------------------------------------------------------------------------------|
| **Issuer ID**     | Identifies your App Store Connect API integration.                                                                               |
| **Key ID**        | Identifies the API key that you created.                                                                                         |
| **Private Key**   | The `.p8` file downloaded from App Store Connect.                                                                                |
| **Team ID**       | Your Apple Developer team identifier.                                                                                            |
| **Bundle ID**     | Your app's unique identifier (for example, `com.company.app`), as configured in your app target and listed in App Store Connect. |
| **Certificate**   | The `.p12` distribution certificate that includes the private key.                                                               |
| **.p12 password** | The password that you specified when exporting the certificate.                                                                  |
{style="none"}

### Upload the first build to TestFlight

After you add the credentials, the pipeline includes signing and deployment steps.

1. Click **Deploy to TestFlight**.
    TeamCity reruns the pipeline, creates a signed iOS build, and uploads it to App Store Connect.
2. Open App Store Connect or TestFlight and verify that the build appears.

## Automate builds and publishing

### Connect the repository

To automate this process, connect your GitHub repository to TeamCity so that every push triggers a new build.

1. After the first deployment pipeline finishes successfully, click **Connect repository** in the IDE.
   TeamCity opens in the browser and confirms that the repository is authorized.
   It detects your repository and branch automatically and attaches them to the pipeline.
2. Return to the IDE, then commit and push the generated pipeline configuration files.

TeamCity now triggers the pipeline whenever you push changes to the configured branch.

### Verify the pipeline

Your iOS delivery pipeline is now ready! Every push to the configured branch triggers TeamCity to:

1. Start a build on a hosted macOS agent.
2. Build the Kotlin Multiplatform project.
3. Run the configured tests.
4. Sign the iOS application.
5. Upload a new build to TestFlight.

![iOS delivery pipeline is ready](ios-pipeline-success.png){width=700 style="block"}

Click **Done** to close the setup flow.

From now on, push your code and TeamCity handles the rest.

## What's next

* Read more about [TeamCity Cloud pipelines](https://www.jetbrains.com/help/teamcity/cloud/create-and-edit-pipelines.html) 
  to customize your setup further: create more projects, set up build agent requirements, and more.
* Learn how to [publish a multiplatform app](multiplatform-publish-apps.md).
* [Configure GitHub Actions](github-actions-for-kmp.md) for continuous integration of a Kotlin Multiplatform application.