[//]: # (title: Migrating a Kotlin Multiplatform project to support AGP 9.0)

Kotlin Multiplatform projects with Android targets which were created using the Android Gradle plugin version earlier than 9.0
need to be restructured to upgrade to AGP 9.0.
Several APIs needed for KMP configuration are hidden in AGP 9.0 and eventually are going to be removed.
To solve this in the long term, we recommend updating your project structure to isolate AGP usage to an Android module.

On top of AGP compatibility, there is an issue of migrating to the new [Android Gradle Library plugin](https://developer.android.com/kotlin/multiplatform/plugin).
In the following guide, we highlight changes related to this as well.

> To make your project work with AGP 9.0 in the short term, you can manually enable the hidden APIs.
> TODO: explain how to do that.
> 
{style="note"}

## Migration guide

In this guide, we show how to restructure a combined multiplatform module into discrete modules clearly delineating
shared logic, shared UI, and individual entry points.
The example project is the default template project produced by a previous version of the KMP IDE plugin,
with the unified `composeApp` module that contains all of the shared logic and shared UI code.

Shared UI code and shared logic code are separated to demonstrate the most flexible and scalable project structure
that helps isolate UI implementation from the rest of shared code.
<!-- TODO I'm still not sure when this is beneficial, need details here. --> 
If your project is simple enough, it might suffice to combine all shared code in a single module.

<!-- TODO: will people have access to the older template somewhere? Can we save it in a branch on GitHub or something so that
people could compare their own project and follow the instructions? -->

### Create a shared logic module

You will isolate code implementing shared business logic in a `sharedLogic` module:

1. Create the `sharedLogic` directory at the root of the project.

### Create a shared UI module

You will isolate code implementing shared business logic in a `sharedUi` module:

1. Create the `sharedLogic` directory at the root of the project.

### Create modules for each app entry point

As stated in the beginning of this page, the only module that needs isolating is the Android app entry point.
But if you have other targets enabled, to make the structure transparent it's more straightforward to keep all of them
on the same level of the project hierarchy.

#### androidApp

#### desktopApp

#### webApp

### Update the iOS integration

## Anything else?