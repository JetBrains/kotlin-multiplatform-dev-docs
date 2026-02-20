[//]: # (title: Configure GitHub Actions for continuous integration of a Kotlin Multiplatform application)

<web-summary>Learn how to set up continuous integration (CI/CD) for Kotlin Multiplatform (KMP) apps with GitHub Actions.
Use CI to run shared tests and build artifacts for iOS, Android, and desktop.</web-summary>

This guide shows an example of continuous integration for a Kotlin Multiplatform application set up with GitHub Actions.
You'll set up a workflow that runs shared tests and builds artifacts for Android, iOS, and desktop on every push or pull request to the `main` branch.

This guide is based on the [Jetcaster KMP sample](https://github.com/kotlin-hands-on/jetcaster-kmp-migration/).
You can see the [action and workflow configuration in the repository](https://github.com/kotlin-hands-on/jetcaster-kmp-migration/tree/main/.github)
or follow the steps below for a step-by-step description.

This guide suggests setting up CI in two parts:

* [Reusable composite GitHub Action that sets up Java and Gradle](#create-a-composite-action-for-gradle-setup)
* [Main GitHub Actions workflow](#define-the-build-workflow) that runs tests and starts platform-specific builds
  on every push or pull request to the `main` branch.

## Create a composite action for Gradle setup

Create a [composite action](https://docs.github.com/en/actions/tutorials/create-actions/create-a-composite-action) to synchronize Java and Gradle configuration across jobs.
You will reuse this action in workflow jobs to make sure that the same configuration is used for all builds.

In this example, the action installs Java 17 and configures the default Gradle version.
To set up the action, create the file `.github/actions/gradle-setup/action.yml`:

```yaml
name: gradle-setup
description: Setup Java and Gradle
runs:
  using: "composite"
  steps:
    - name: Setup Java
      uses: actions/setup-java@v4.0.0
      with:
        java-version: "17"
        distribution: "temurin"
    - name: Setup Gradle
      uses: gradle/actions/setup-gradle@v5.0.0
```

## Define the build workflow

Define when the workflow runs and configure Gradle options:

* The workflow should run on every push or pull request to the `main` branch.
* Gradle options should disable the Gradle daemon and enable parallel execution with caching.

Create the file `.github/workflows/build.yml` with the base configuration:

```yaml
name: Build

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  workflow_dispatch:

env:
  GRADLE_OPTS: "-Dorg.gradle.jvmargs=-Xmx4096M -Dorg.gradle.daemon=false -Dorg.gradle.parallel=true -Dorg.gradle.caching=true"
```

With this configuration, you can also trigger the workflow manually using [workflow_dispatch](https://docs.github.com/en/actions/how-tos/manage-workflow-runs/manually-run-a-workflow).

Now you can add jobs to run tests and build application artifacts.

### Run shared tests

This job runs tests using the `jvmTest` Gradle task to validate changes before building the app for all platforms:

1. Check out the repository to run tests for.
2. Use the composite `gradle-setup` action that you prepared earlier to set up Java and Gradle.
3. Run the tests with a `./gradlew` command.
4. Upload test reports as an artifact to the `**/build/reports/tests/` directory.

To set up this job, add the following to the `.github/workflows/build.yml` file:

```yaml
jobs:
  test:
    name: Run tests
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Gradle setup
        uses: ./.github/actions/gradle-setup

      - name: Run unit tests
        run: ./gradlew jvmTest

      - name: Upload test reports
        uses: actions/upload-artifact@v4
        with:
          name: test-reports
          path: "**/build/reports/tests/"
```

Once tests are run, the workflow should build the application artifacts.

### Build the Android debug package

This job builds the Android debug APK using the `:mobile:assembleDebug` Gradle task:

1. Check out the repository to build the package from.
2. Use the composite `gradle-setup` action that you prepared earlier to set up Java and Gradle.
3. Build the APK with a `./gradlew` command.
4. Upload the built package from the `mobile/build/outputs/apk/debug/` directory.

Add the following to the `.github/workflows/build.yml` file, continuing the `jobs` section:

```yaml
jobs:
  # ...
  build-android:
    name: Build Android
    runs-on: ubuntu-latest
    needs: test
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Gradle setup
        uses: ./.github/actions/gradle-setup

      - name: Build Android debug APK
        run: ./gradlew :mobile:assembleDebug

      - name: Upload Android debug APK
        uses: actions/upload-artifact@v4
        with:
          name: android-apk
          path: mobile/build/outputs/apk/debug/*.apk
```

### Build the iOS simulator application

This job targets the iOS Simulator to avoid having to properly sign the app.
The application is built using `xcodebuild`:

1. Check out the repository to build the application from.
2. Use the composite `gradle-setup` action that you prepared earlier to set up Java and Gradle.
3. Build the iOS application using `xcodebuild` — the example shows the options used for the Jetcaster KMP sample.
4. Upload the folder with the built application (everything inside `build/Build/Products/Debug-iphonesimulator/*`) as an artifact.

The iOS application is built on a macOS runner (`macos-latest`), which includes `xcodebuild`. 

Continue in `.github/workflows/build.yml`:

```yaml
jobs:
  #...
  build-ios:
    name: Build iOS simulator app
    runs-on: macos-latest
    needs: test
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Gradle setup
        uses: ./.github/actions/gradle-setup

      - name: Build iOS simulator app
        run: |
          xcodebuild build \
          -project JetcasterMigration/JetcasterMigration.xcodeproj \
          -configuration Debug \
          -scheme JetcasterMigration \
          -sdk iphonesimulator \
          -derivedDataPath ./build \
          -verbose

      - name: Upload app folder
        uses: actions/upload-artifact@v4
        with:
          name: iphonesimulator-app
          path: build/Build/Products/Debug-iphonesimulator/*
```

## Push and test your CI

The CI workflow is going to be triggered for the first time when you push the workflow configuration to the `main` branch 
or create a pull request with these configuration files.

You can check the workflow results in the **Actions** tab of your repository to verify that everything works correctly.

Remember that you can also trigger the workflow manually: select the workflow in the list of actions on the left
and click **Run workflow**.

## What’s next

For complete CI configurations, see the [Jetcaster sample](https://github.com/kotlin-hands-on/jetcaster-kmp-migration/tree/main/.github),
which also includes jobs to build desktop JVM applications for macOS, Windows, and Linux.

For guidance on publishing applications to app stores with GitHub Actions, see [Marco Gomiero’s series of posts on the subject](https://www.marcogomiero.com/posts/2024/kmp-ci-ios/).