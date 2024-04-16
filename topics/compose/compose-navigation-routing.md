[//]: # (title: Navigation and routing)

> The navigation library is currently [Experimental](supported-platforms.md#core-kotlin-multiplatform-technology-stability-levels).
> You're welcome to try it in your Compose Multiplatform projects.
> We would appreciate your feedback on [GitHub](https://github.com/JetBrains/compose-multiplatform/issues).
>
{type="warning"}

Navigation is a key part of UI applications that allows users to navigate between different application screens.
Compose Multiplatform adopts the [Jetpack Compose approach to navigation](https://developer.android.com/guide/navigation/design#frameworks).

Just as with Jetpack Compose, to set up navigation, you should:
1. List routes that should be included in the navigation graph. Each route must be a unique string that defines a path.
2. Create a `NavHostController` instance as a property of your main composable.
3. Add a `NavHost` composable to your app:
   1. Choose the starting destination from the list of routes you defined.
   2. Create a navigation graph, either directly, as part of adding the `NavHost`, or programmatically, using the
     `NavController.createGraph()` function.

Each back stack entry (each navigation route included in the graph) implements the `LifecycleOwner` interface.
A switch between different screens of the app makes it change its state from `RESUMED` to `STARTED` and back.
`RESUMED` is also described as "settled": the act of navigation is considered finished when the new screen is prepared and active.
See the [Lifecycle](compose-lifecycle.md) page for details on the current lifecycle implementation in Compose Multiplatform. 

## Setup

To use the navigation library, add the following dependency to your common source set:

```kotlin
kotlin {
    ...
    sourceSets {
        ...
        commonMain.dependencies {
            ...
            implementation(libs.androidx.navigation.compose)
            implementation("org.jetbrains.androidx.navigation:navigation-compose:2.8.0-alpha01")
        }
        ...
    }
}
```

## Sample project

To see the Compose Multiplatform navigation library in action, check out the [nav_cupcake project](https://github.com/MatkovIvan/nav_cupcake)
which was converted from the [Navigate between screens with Compose](https://developer.android.com/codelabs/basic-android-kotlin-compose-navigation#0)
Android codelab.

Things to note:
* [The ViewModel factory](https://github.com/MatkovIvan/nav_cupcake/blob/1dc15b6ef68f68ba358a32501802142967f6494b/composeApp/src/commonMain/kotlin/com/matkovivan/nav_cupcake/ViewModels.kt#L18)
  allows creating `ViewModel` entities of the correct type ob iOS and web.
* The [ComposeViewModelStoreOwner](https://github.com/MatkovIvan/nav_cupcake/blob/1dc15b6ef68f68ba358a32501802142967f6494b/composeApp/src/commonMain/kotlin/com/matkovivan/nav_cupcake/ViewModels.kt#L27)
  class implements the `ViewModelStoreOwner` interface to provide a fallback for platforms other than Android and desktop.
* Compose Multiplatform string resources used in place of Jetpack Compose resources.

## Limitations

Current limitations of navigation in Compose Multiplatform, compared to Jetpack Compose:
* [Deep links](https://developer.android.com/guide/navigation/design/deep-link) (handling or following them) are not supported.
* The [BackHandler](https://developer.android.com/develop/ui/compose/libraries#handling_the_system_back_button) function
  and [predictive back gestures](https://developer.android.com/guide/navigation/custom-back/predictive-back-gesture)
  are not supported on any platform besides Android.

## Third-party alternatives

If the Compose Multiplatform Navigation components do not solve your problems,
there are third-party alternatives that you can choose from:

| Name                                                | Description                                                                                              |
|-----------------------------------------------------|----------------------------------------------------------------------------------------------------------|
| [Voyager](https://voyager.adriel.cafe)              | A pragmatic approach to navigation                                                                       |
| [Decompose](https://arkivanov.github.io/Decompose/) | An advanced approach to navigation that covers the full lifecycle and any potential dependency injection |
| [Appyx](https://bumble-tech.github.io/appyx/)       | Model-driven navigation with gesture control                                                             |
| [PreCompose](https://tlaster.github.io/PreCompose/) | A navigation and view model inspired by Jetpack Lifecycle, ViewModel, LiveData, and Navigation           |
