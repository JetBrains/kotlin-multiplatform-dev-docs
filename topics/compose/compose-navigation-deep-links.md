[//]: # (title: Deep links)

Deep linking is a navigation mechanism that allows the operating system to handle custom links
by taking the user to a specific destination in the corresponding app.

To implement a deep link in Compose Multiplatform:

1. Register your deep link schema in the app configuration.
2. Assign specific deep links to composables in the navigation graph.
3. Handle deep links received by the app by converting them into navigation commands.

## Registering deep links schemas on various systems

