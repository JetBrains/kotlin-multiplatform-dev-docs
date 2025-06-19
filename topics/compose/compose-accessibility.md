[//]: # (title: Accessibility)

Compose Multiplatform provides features essential for meeting accessibility standards, such as semantic properties, 
accessibility APIs, and support for assistive technologies, including screen readers and keyboard navigation.

The framework enables the design of applications that comply with the requirements of the 
[European Accessibility Act](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX%3A32019L0882) (EAA)
and the [Web Content Accessibility Guidelines](https://www.w3.org/TR/WCAG21/) (WCAG).

## Semantic properties

Semantic properties define the meaning and role of a component, adding context for services such as accessibility, 
autofill, and testing.

Semantic properties are the building blocks of the semantic tree. When you define semantic properties in composables, 
they are automatically added to the semantic tree. When assistive technologies interact with the app, they traverse 
the semantic tree, not the entire UI tree.

Key semantic properties for accessibility:

* `contentDescription` provides a description for non-textual or ambiguous UI elements like
  [`IconButton`](https://kotlinlang.org/api/compose-multiplatform/material3/androidx.compose.material3/-icon-button.html),  
  and [`FloatingActionButton`](https://kotlinlang.org/api/compose-multiplatform/material3/androidx.compose.material3/-floating-action-button.html).
  It is the primary accessibility API and serves for providing a textual label that screen readers announce.

  ```kotlin
  Modifier.semantics { contentDescription = "Description of the image" }
  ```

* `role` informs accessibility services about the UI component's functional type, such as button, 
  checkbox, or image. This helps screen readers interpret interaction models and announce the element properly.

  ```kotlin
  Modifier.semantics { role = Role.Button }
  ```

* `stateDescription` describes the current state of an interactive UI element.

  ```kotlin
  Modifier.semantics { stateDescription = if (isChecked) "Checked" else "Unchecked" }
  ```

* `testTag` is used primarily in UI testing via accessibility identifiers, like in the Espresso 
  framework on Android or XCUITest on iOS. In addition, `testTag` can be useful for debugging or in specific 
  automation scenarios where a component identifier is required.

  ```kotlin
  Modifier.testTag("my_unique_element_id")
  ```

For a full list of semantics properties, see the 
[SemanticsProperties API](https://developer.android.com/reference/kotlin/androidx/compose/ui/semantics/SemanticsProperties) 
reference.

## Traversal order

By default, screen readers navigate through UI elements in a fixed order, following their layout from left 
to right and top to bottom. However, for complex layouts, screen readers may not automatically determine the correct 
reading order. This is crucial for layouts with container views, 
such as tables and nested views, that support the scrolling and zooming of contained views.

To ensure the correct reading order when scrolling and swiping through complex views, define traversal semantic properties.
This also ensures correct navigation between different traversal groups with the swipe-up 
or swipe-down accessibility gestures.

The default value of the traversal index is `0f`.
The lower the traversal index value of a component, the earlier its content description will be read relative
to other components.

For example, if you want the screen reader to prioritize a floating action button, 
you can set its traversal index to `-1f`:

```kotlin
@Composable
fun FloatingBox() {
    Box(
        modifier =
        Modifier.semantics {
            isTraversalGroup = true
            // Sets a negative index to prioritize over elements with the default index
            traversalIndex = -1f
        }
    ) {
        FloatingActionButton(onClick = {}) {
            Icon(
                imageVector = Icons.Default.Add,
                contentDescription = "Icon of floating action button"
            )
        }
    }
}
```

## What’s next

Learn more about accessibility features for iOS:

* [High-contrast theme](compose-ios-accessibility.md#high-contrast-theme)
* [Test accessibility with XCTest framework](compose-ios-accessibility.md#test-accessibility-with-xctest-framework)
* [Control via trackpad and keyboard](compose-ios-accessibility.md#control-via-trackpad-and-keyboard)
* [Synchronize the semantic tree with the iOS accessibility tree](compose-ios-accessibility.md#choose-the-tree-synchronization-option) 
  (for Compose Multiplatform 1.7.3 and earlier)

Learn more about accessibility features for desktop:

* [Enable accessibility on Windows](compose-desktop-accessibility.md#enabling-accessibility-on-windows)
* [Test your app with macOS and Windows tools](compose-desktop-accessibility.md#example-custom-button-with-semantic-rules)

For implementation details, see the [Jetpack Compose documentation](https://developer.android.com/develop/ui/compose/accessibility).