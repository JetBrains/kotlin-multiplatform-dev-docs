[//]: # (title: Navigation and routing)

Navigation is a key part of modern UI applications that allows users to navigate between different application screens.
Unfortunately, the [Navigation component](https://developer.android.com/guide/navigation) from Jetpack Compose's suite
of libraries is currently unavailable in Compose Multiplatform. However, there are other third-party alternatives that
you can choose from:

| Name                                                | Description                                                                                              |
|-----------------------------------------------------|-------------------------------------------------------------------------------------------------------------------|
| [Voyager](https://voyager.adriel.cafe)              | A pragmatic approach to navigation                                                                                |
| [Decompose](https://arkivanov.github.io/Decompose/) | An advanced approach to navigation that covers the full lifecycle and any potential dependency injection          |
| [Appyx](https://bumble-tech.github.io/appyx/)       | Model-driven navigation with gesture control                                                                      |
| [PreCompose](https://tlaster.github.io/PreCompose/) | A navigation and view model inspired by Jetpack Lifecycle, ViewModel, LiveData, and Navigation                    |
| [Circuit](https://slackhq.github.io/circuit/)       | A Compose-driven architecture for Kotlin applications with navigation and advanced state management.  |

In future, Compose Multiplatform intends to provide an out-of-the-box navigation library.
