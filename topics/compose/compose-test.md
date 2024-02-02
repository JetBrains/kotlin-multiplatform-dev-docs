[//]: # (title: Testing Compose UI)

Changes:
1. "If your project includes a desktop target and you want to run desktop tests, you also need to add implementation(compose.desktop.currentOs)"
   *  declaring desktopTest variable in line with the wizard generation
2. `alias(libs.plugins.jetbrainsCompose) apply false` in the root build.gradle should be commented out while Test API is in beta.
3. The exact version is `id("org.jetbrains.compose") version "1.6.0-dev1397"`
4. Change the `painterResource` call to `Image(painterResource(Res.drawable.compose_multiplatform)`
5. For the Android emulator, imports in the Gradle file:
   ```
   import org.jetbrains.kotlin.gradle.ExperimentalKotlinGradlePluginApi
   import org.jetbrains.kotlin.gradle.plugin.KotlinSourceSetTree
   ```

## Comparison with Jetpack Compose

## Setting up test environment

## Running common tests

### Running test on Android emulator

## Running JUnit tests in desktop-only projects