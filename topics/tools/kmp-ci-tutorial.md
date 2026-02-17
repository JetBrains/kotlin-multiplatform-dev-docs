[//]: # (title: Configure continuous integration for a Kotlin Multiplatform application)

<web-summary>Learn how to set up continuous integration (CI/CD) for Kotlin Multiplatform (KMP) apps with GitHub Actions.
Run shared tests, build Android and iOS artifacts, and streamline CI for cross-platform projects.</web-summary>

This guide shows how to configure continuous integration for a Kotlin Multiplatform application by using GitHub Actions. You set up a workflow that runs shared tests once and builds artifacts for Android and iOS. The workflow runs on every push or pull request to the main branch.

This guide is based on the [Jetcaster KMP sample](https://github.com/kotlin-hands-on/jetcaster-kmp-migration/).
You can see the [action and workflow configuration in the repository](https://github.com/kotlin-hands-on/jetcaster-kmp-migration/tree/main/.github)
or follow the steps below for a detailed description.

## Overview of the CI structure

The CI setup contains two main parts:

* A reusable composite action that sets up Java and Gradle.
* A GitHub Actions workflow that runs tests and platform-specific builds on every push or pull request.

## Create a composite action for Gradle setup

Create a [composite action](https://docs.github.com/en/actions/tutorials/create-actions/create-a-composite-action) to synchronize Java and Gradle configuration across jobs.
This action installs Java 17 and configures Gradle so that it can be reused for multiple workflows.

Create the file `.github/actions/gradle-setup/action.yml`:

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
Define when the workflow runs and configure Gradle options. The workflow runs on every push or pull request to the main branch. You can also trigger it manually in GitHub Actions using [workflow_dispatch](https://docs.github.com/en/actions/how-tos/manage-workflow-runs/manually-run-a-workflow). Set Gradle options to disable the Gradle daemon and enable parallel execution with caching.


Create the file `.github/workflows/build.yml`:

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

## Run shared tests
Run tests to validate changes before building the app for all platforms. Use the composite action to set up Java and Gradle. Test reports are saved in the `build/reports/tests/` folder. In the Jetcaster sample, tests run with the `jvmTest` Gradle task.

Continue in `.github/workflows/build.yml`:

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

## Build the Android application
After tests complete, build the applications. For Jetcaster, build the debug APK and place it in the `mobile/build/outputs/apk/debug/` folder.

Continue in `.github/workflows/build.yml`:

```yaml
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

## Build the iOS application
Build the iOS application on a macOS runner by using `macos-latest`. The runner includes xcodebuild. To build the app on CI without configuring signing, target the iOS simulator.

Continue in `.github/workflows/build.yml`:

```yaml
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
Push your workflow changes to the main branch. This triggers the CI workflow, which runs tests and builds the applications. Check the workflow results in GitHub Actions to verify that everything works correctly. Alternatively, trigger the workflow manually from the Actions tab in GitHub.

## What’s next
For more CI configurations, see the [Jetcaster sample](https://github.com/kotlin-hands-on/jetcaster-kmp-migration/pull/1/changes/fef86abd0bd461ab8dfd5ce9826918398c24a541#r2758188651), which shows how to build applications for macOS, Windows, and Linux.

For guidance on [publishing applications to app stores with GitHub Actions](https://www.marcogomiero.com/posts/2024/kmp-ci-ios/), see Marco Gomiero’s series.