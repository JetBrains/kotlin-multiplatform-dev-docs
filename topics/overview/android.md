[//]: # (title: Making an Android app multiplatform)

This is a page about where to start when you have an Android app and would like to turn it into a KMP project
to create a unified foundation for apps on other platforms.

We're going to be talking about iOS specifically, but general logic works for any platform you may choose. 

General steps:

1. Identify code that can be shared.
2. Move common code to a new module that will be shared between platforms.
3. Add a target for a new platform with minimal UI that uses common code. Now you have a proof of concept.
   * (Optional) you can recreate a single UI screen in common code using Compose Multiplatform 
     to demonstrate sharing UI as well as backend code.
4. Time to sell that PoC and answer questions:
   * Does the development environment need to change?
   * What do we do when we need native APIs?
   * Can we have platform-specific code for other platforms or specific UI screens implemented natively e.g. with SwiftUI?
   * How do we organize code (structure the project, write tests, store resources)?
   * What does the publishing process look like?
5. When everyone is onboard, you can go one of two ways:
   * Commit to implementing new features in KMP while leaving the production core as it is.
   * Start the migration process:
      1. Identify code that can be shared.
      2. Move it to the shared module.
      3. Call it from other targets as needed.
      4. Test.
      5. (Optional) create libraries


## 