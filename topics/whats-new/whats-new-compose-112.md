[//]: # (title: What's new in Compose Multiplatform %org.jetbrains.compose-eap%)

Here are the highlights for this EAP release:

 * [Automatic font fallback for web](#automatic-font-fallback)
 * 
 * 

You can find the full list of changes for this release on [GitHub](https://github.com/JetBrains/compose-multiplatform/releases/tag/v1.12.0-beta01).
For details about specific component versions, refer to the [Dependencies](#dependencies) section.

## Breaking changes and deprecations

## Across platforms

## iOS

## Web

### Automatic font fallback
<primary-label ref="Experimental"/>

Previously, characters not covered by the application's loaded fonts were displayed as replacement glyphs (□, known as "tofu").

Compose Multiplatform for Web now automatically downloads the required Noto font subsets 
on demand when unresolved characters are encountered during rendering. 
After the fonts are downloaded, Compose recomposes the affected text. 
Note that tofu may briefly appear until the required font is fetched.

## Desktop

## Gradle

## Dependencies