[//]: # (title: Common ViewModel)

The Android [ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel)
approach to building UI can be implemented in common code using Compose Multiplatform.

To use the multiplatform `ViewModel` implementation, add the following dependency to your `commonMain` source set:

```kotlin
kotlin {
    // ...
    sourceSets {
        // ...
        commonMain.dependencies {
            // ...
            implementation("org.jetbrains.androidx.lifecycle:lifecycle-viewmodel-compose:%composeViewmodelVersion%")
        }
        // ...
    }
}
```

To create a ViewModel, you need to pass it an object that implements the `ViewModelStoreOwner` interface.
Compose Multiplatform offers a common `ViewModelStoreOwner` which you can implement in your common code to use ViewModels
to build your UI. Common `ViewModelStoreOwner` doesn't work for desktop

Keep in mind the limitations of the library:

* The  `ViewModel` implementation is currently considered [Experimental](supported-platforms.md#core-kotlin-multiplatform-technology-stability-levels).
* The `ViewModel` class works out of the box only for Android and desktop, where objects of the needed class can be created
  through class reflection. For iOS and web targets, you have to implement factories that explicitly create
  new `ViewModel` instances.
* 
* **You can't call viewModel() without args on non-JVM targets**