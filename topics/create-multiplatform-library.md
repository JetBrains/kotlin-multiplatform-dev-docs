[//]: # (title: Create your Kotlin Multiplatform library – tutorial)

<primary-label ref="intellij-and-android-studio"/>

In this tutorial, you'll learn how to create a Multiplatform library in IntelliJ IDEA, publish the library to a local Maven repository,
and add it as a dependency in another project.

The tutorial is based on our [Multiplatform library template](https://github.com/Kotlin/multiplatform-library-template),
which is a simple library containing a function to generate the Fibonacci 
sequence.

## Set up the environment 

[Install all the necessary tools and update them to the latest versions](multiplatform-setup.md).

## Create a project

1. In IntelliJ IDEA, select **File** | **New** | **Project from Version Control**.
2. Type the URL of the [Multiplatform library template project](https://github.com/Kotlin/multiplatform-library-template):
    ```text
    https://github.com/Kotlin/multiplatform-library-template
    ```
3. Click **Clone**.