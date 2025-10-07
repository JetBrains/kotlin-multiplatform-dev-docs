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

In this guide, we show how to restructure a combined module (`composeApp`, produced by an older version of KMP IDE plugin)
into discrete modules clearly delineating shared logic, shared UI, and individual entry points.


## Anything else?