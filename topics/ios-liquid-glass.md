[//]: # (title: Native iOS Navigation with Liquid Glass in a Compose Multiplatform App)

This tutorial walks through migrating an iOS app from full Compose-driven navigation to native SwiftUI navigation with
iOS 26 liquid glass styling, while keeping Compose rendering each screen's content.

![Shared UI](ios-kotlinconf-no-liquid-glass.png){ width="250" style="inline"}
![Native iOS UI with liquid glass](ios-kotlinconf-liquid-glass.png){ width="250" style="inline"}

Sample project: [KotlinConfApp](https://github.com/JetBrains/kotlinconf-app)

* [main branch](https://github.com/JetBrains/kotlinconf-app/tree/main) — starting state with Compose-driven navigation
* [lg-nav branch](https://github.com/JetBrains/kotlinconf-app/tree/lg-nav) — final state after this migration
  Clone the repo and check out either branch to follow along, or compare them side by side: [main...lg-nav](https://github.com/JetBrains/kotlinconf-app/compare/main...lg-nav).

Prerequisites: Xcode 26+ with the iOS 26 SDK.
Tooling versions (Kotlin, Compose Multiplatform, Gradle)
clone the repo first and verify the app builds before starting the migration.

The example uses a simplified two-tab app (**Schedule** and **Info**), but the same pattern scales to any number of tabs.

## The plan

In a standard Compose Multiplatform setup, a single `ComposeUIViewController` owns the entire iOS app:
tabs, navigation stack, back gestures, and screen content. 

This works, but it means:

* The tab bar and navigation transitions look like Compose, not native iOS
* You can't use iOS 26's liquid glass tab bar or system `NavigationStack` behavior

The solution is to hand navigation *ownership* back to SwiftUI while keeping Compose for screen content.
Kotlin's `NavHost` gets *escape hatch callbacks* (`onNavigate`, `onActivate`).
Instead of navigating internally, it calls these and Swift handles the stack.

**Before:**

```
Swift: ContentView
  └── ComposeUIViewController  (owns everything: tabs, navigation, screens)
```

**After:**

```
Swift: ContentView
  └── TabView  (liquid glass, iOS 26)
        ├── Tab: Schedule
        │     └── NavigationStack
        │           ├── NativeNavComposeView  ← Kotlin tab root
        │           └── DetailComposeView     ← Kotlin detail screen (per destination)
        └── Tab: Info
              └── NavigationStack
                    ├── NativeNavComposeView  ← Kotlin tab root
                    └── DetailComposeView     ← Kotlin detail screen
```

To complete the migration, you will:

1. [Add `title` and `subtitle` properties to route objects](#add-title-metadata-to-routes) so Swift can display
   navigation bar titles without calling back into Kotlin.
2. [Add `topLevelRoute`, `onNavigate`, and `onActivate` parameters to `App()`](#add-navigation-callbacks-to-app) so the
   iOS layer can control which tab opens and receive push events.
3. [Intercept navigation in `NavHost`](#intercept-navigation-in-navhost) so any route added to Compose's back stack is
   immediately forwarded to Swift and removed from Compose.
4. [Add `LocalUseNativeNavigation`](#add-localusenativenavigation) and use it to hide Compose's own back buttons, title
   bars, and bottom tab bar when Swift owns navigation.
5. [Extract `ScreenContent`](#extract-screencontent) — a flat `when` expression that renders any detail route as a
   standalone composable, used by Swift for each `NavigationStack` destination.
6. [Add `SingleScreenApp`](#add-singlescreenapp-ios-source-set) to the iOS source set, wrapping `ScreenContent` with the
   full DI and theme setup.
7. [Expose two new Kotlin entry points](#expose-ios-entry-points-in-main-ios-kt) — `MainViewController` (with callbacks)
   and `ScreenViewController` — that Swift calls to create each `UIViewController`.
8. [Build the Swift navigation layer](#build-the-swift-navigation-layer): `TabView`, per-tab `NavigationStack`, and the
   `UIViewControllerRepresentable` bridges.

## Add title metadata to routes

Swift's `NavigationStack` shows a navigation bar title for each pushed destination. Carry the title directly on the
route object so Swift doesn't need to call back into Kotlin to fetch it.

1. In the `navigation/Routes.kt` file, add `title` and `subtitle` properties to `AppRoute`:

    ```kotlin
    @Serializable
    sealed interface AppRoute {
        val title: String? get() = null
        val subtitle: String? get() = null
    }
    ```

2. Add `title` to routes that appear as detail screens. For routes that already carry data, add it as an optional
   parameter:

    ```kotlin
    @Serializable
    data class SessionScreen(
        val sessionId: SessionId,
        override val title: String? = null,
    ) : AppRoute
    
    @Serializable
    data class SpeakerDetailScreen(
        val speakerId: SpeakerId,
        override val title: String = "",
        override val subtitle: String = "",  // speaker role / company
    ) : AppRoute
    ```

3. For routes that were `data object` and need a title, convert them to `data class`:

    ```kotlin
    // Before:
    data object SettingsScreen : AppRoute
    
    // After:
    data class SettingsScreen(override val title: String = "") : AppRoute
    ```

4. Pass the localized title at the call site. Because `stringResource` is a `@Composable` function, hoist it to the
   enclosing `entry` scope — not inside a click callback:

    ```kotlin
    entry<InfoScreen> {
        val settingsTitle = stringResource(Res.string.info_link_settings)
        InfoScreen(
            onSettings = { navigator.add(SettingsScreen(settingsTitle)) },
            // ...
        )
    }
    ```

## Add navigation callbacks to `App()`

`App()` is the Kotlin entry point for iOS calls. 
In the `App.kt` file, add a `topLevelRoute` parameter so iOS can specify which tab to open, 
and two optional callbacks for navigation events:

```kotlin
@Composable
fun App(
    appGraph: AppGraph,
    topLevelRoute: TopLevelRoute = ScheduleScreen,
    onThemeChange: ((isDarkTheme: Boolean) -> Unit)? = null,
    onNavigate: ((AppRoute) -> Unit)? = null,
    onActivate: ((TopLevelRoute) -> Unit)? = null,
) {
    // ...
    val startRoute: AppRoute = remember {
        if (isOnboardingComplete) topLevelRoute else StartPrivacyNoticeScreen
    }
    NavHost(startRoute, isDarkTheme, onThemeChange, onNavigate, onActivate)
}
```

Both callback parameters default to `null`, so Android, Desktop, and Web remain unchanged.

## Intercept navigation in `NavHost`

Update `NavHost` to accept the new parameters, then intercept any detail routes before Compose can manage them.
In `navigation/NavHost.kt`:

```kotlin
import androidx.compose.runtime.snapshotFlow

@Composable
internal fun NavHost(
    startRoute: AppRoute,
    isDarkTheme: Boolean,
    onThemeChange: ((Boolean) -> Unit)?,
    onNavigate: ((AppRoute) -> Unit)? = null,
    onActivate: ((TopLevelRoute) -> Unit)? = null,
) {
    // ...

    // Intercepts detail routes: forward to Swift and remove from Compose's stack
    if (onNavigate != null) {
        LaunchedEffect(navState) {
            snapshotFlow { navState.currentBackstack.toList() }.collect { backstack ->
                val detailRoutes = backstack.drop(1)
                if (detailRoutes.isNotEmpty()) {
                    detailRoutes.forEach { onNavigate(it) }
                    navState.currentBackstack.removeRange(1, navState.currentBackstack.size)
                }
            }
        }
    }

    // Notifies Swift when the user switches tabs from within Compose
    if (onActivate != null) {
        LaunchedEffect(navState) {
            snapshotFlow { navState.topLevelRoute }.collect { route ->
                if (route != null) onActivate(route)
            }
        }
    }

    // ...
}
```

## Add `LocalUseNativeNavigation`

Compose screens render their own back buttons, headers, and bottom tab bar. 
When iOS owns navigation, hide those to avoid duplication.

1. Declare the composition local in `NavHost.kt`:

    ```kotlin
    val LocalUseNativeNavigation = staticCompositionLocalOf { false }
    ```

2. Set it based on whether native navigation callbacks are active, and skip `NavScaffold` (the Compose bottom bar) when
   they are:

    ```kotlin
    val useNativeNavigation = onNavigate != null

    CompositionLocalProvider(LocalUseNativeNavigation provides useNativeNavigation) {
        if (useNativeNavigation) {
            NavDisplay(entries = ..., onBack = ...)
        } else {
            NavScaffold(...) {
                NavDisplay(entries = ..., onBack = ...)
            }
        }
    }
    ```

3. Read the flag in `BaseScreens.kt` to conditionally hide the Compose back button and title bar:

    ```kotlin
    // BaseScreens.kt — ScreenWithTitle

    val useNativeNavigation = LocalUseNativeNavigation.current

    if (!useNativeNavigation) {
        MainHeaderTitleBar(...)
        HorizontalDivider(...)
    }
    ```

## Extract `ScreenContent`

`ScreenContent` is a flat `when` expression that renders a single detail `AppRoute` as a standalone composable. Swift
calls `ScreenViewController` for each destination it pushes, which internally calls `ScreenContent`.

> `ScreenContent` only handles detail routes — screens pushed onto a `NavigationStack`. Top-level tab
> roots (`ScheduleScreen`, `InfoScreen`) are never passed here; they are rendered by the full `App()/NavHost` inside
`NativeNavComposeView`.
> 
> {style="note"}

In `navigation/NavHost.kt`:

```kotlin
@Composable
fun ScreenContent(
    route: AppRoute,
    onNavigate: (AppRoute) -> Unit,
    onBack: () -> Unit,
    onSet: (AppRoute) -> Unit,
    onActivate: (TopLevelRoute) -> Unit,
) {
    val uriHandler = LocalUriHandler.current
    when (route) {
        is SessionScreen -> SessionScreen(
            sessionId = route.sessionId,
            onBack = onBack,
            onSpeaker = { speakerId -> onNavigate(SpeakerDetailScreen(speakerId)) },
            // ...
        )
        is SpeakerDetailScreen -> SpeakerDetailScreen(
            speakerId = route.speakerId,
            onBack = onBack,
            onSession = { sessionId -> onNavigate(SessionScreen(sessionId)) },
        )
        is SettingsScreen -> SettingsScreen(onBack = onBack)
        is AboutAppScreen -> AboutAppScreen(
            onBack = onBack,
            onLicenses = { onNavigate(LicensesScreen) },
            // ...
        )
        // ... all other detail routes
        else -> {}
    }
}
```

The `title` for each pushed screen (for example, a `SettingsScreen`'s title) is embedded in the route object when the
route is created — Swift reads `wrapper.route.title` directly for the navigation bar.

## Add `SingleScreenApp` (iOS source set)

`SingleScreenApp` wraps `ScreenContent` with the same DI, theme, and window setup as `App()`, but provides
`LocalUseNativeNavigation = true` so screens automatically hide their Compose chrome.

In `iosMain/SingleScreenApp.kt`: 
```kotlin
@Composable
internal fun SingleScreenApp(
    appGraph: AppGraph,
    route: AppRoute,
    onNavigate: (AppRoute) -> Unit,
    onGoBack: () -> Unit,
    onSet: (AppRoute) -> Unit,
    onActivate: (TopLevelRoute) -> Unit,
) {
    // ... theme and flags setup

    CompositionLocalProvider(
        LocalUseNativeNavigation provides true,
        LocalFlags provides flags,
        LocalAppGraph provides appGraph,
        // ... other providers
    ) {
        KotlinConfTheme(colors = colors) {
            Box(Modifier.fillMaxSize().background(KotlinConfTheme.colors.mainBackground)) {
                ScreenContent(route, onNavigate, onGoBack, onSet, onActivate)
            }
        }
    }
}
```

## Expose iOS entry points in `main.ios.kt`

In `iosMain/main.ios.kt`, add three Kotlin functions that Swift can call:

```kotlin
// Pre-iOS 26 fallback: full Compose navigation, no native callbacks
@Suppress("unused")
fun MainViewController(
    topLevelRoute: TopLevelRoute,
): UIViewController = ComposeUIViewController(
    configure = { onFocusBehavior = OnFocusBehavior.DoNothing }
) {
    App(appGraph, topLevelRoute)
}

// Tab root: Compose runs NavHost but hands navigation events to Swift
@Suppress("unused")
fun MainViewController(
    topLevelRoute: TopLevelRoute,
    onNavigate: (AppRoute) -> Unit,
    onGoBack: () -> Unit,
    onSet: (AppRoute) -> Unit,
    onActivate: (TopLevelRoute) -> Unit,
): UIViewController = ComposeUIViewController(
    configure = { onFocusBehavior = OnFocusBehavior.DoNothing }
) {
    App(appGraph, topLevelRoute, onNavigate = onNavigate, onActivate = onActivate)
}

// Detail screen: renders a single screen with LocalUseNativeNavigation = true
@Suppress("unused")
fun ScreenViewController(
    route: AppRoute,
    onNavigate: (AppRoute) -> Unit,
    onGoBack: () -> Unit,
    onSet: (AppRoute) -> Unit,
    onActivate: (TopLevelRoute) -> Unit,
): UIViewController = ComposeUIViewController(
    configure = { onFocusBehavior = OnFocusBehavior.DoNothing }
) {
    SingleScreenApp(appGraph, route, onNavigate, onGoBack, onSet, onActivate)
}
```

The first `MainViewController` overload (no nav callbacks) is required for the pre-iOS 26 `ComposeView`
fallback in Swift. Without it, the `#available` branch for older iOS versions won't compile.

## Build the Swift navigation layer

### Make `AppRoute` hashable

`NavigationStack` requires its path elements to be `Hashable`. `AppRoute` is a Kotlin sealed interface, 
so wrap it in a Swift struct:

```swift
@available(iOS 26.0, *)
struct RouteWrapper: Hashable, Identifiable {
    let id = UUID()
    let route: AppRoute

    static func ==(lhs: RouteWrapper, rhs: RouteWrapper) -> Bool {
        lhs.id == rhs.id
    }

    func hash(into hasher: inout Hasher) {
        hasher.combine(id)
    }
}
```

Using `UUID` as the identity (not the route's content) means pushing the same route twice creates two distinct stack
entries, which is the correct behavior.

### Create per-tab and app-level coordinators

```swift
@available(iOS 26.0, *)
@Observable
class TabNavigationCoordinator {
    var path: [RouteWrapper] = []

    func push(_ route: AppRoute) {
        path.append(RouteWrapper(route: route))
    }

    func pop() {
        if !path.isEmpty {
            path.removeLast()
        }
    }

    func popToRoot() {
        path.removeAll()
    }
}

@available(iOS 26.0, *)
@Observable
class AppNavigationCoordinator {
    enum AppTab {
        case schedule, info
    }

    var selectedTab: AppTab = .schedule
    let scheduleCoordinator = TabNavigationCoordinator()
    let infoCoordinator = TabNavigationCoordinator()

    func activateTab(for route: TopLevelRoute) {
        if route is ScheduleScreen {
            selectedTab = .schedule
        } else if route is InfoScreen {
            selectedTab = .info
        }
    }
}
```

### Bridge Kotlin views into SwiftUI

`NativeNavComposeView` renders a tab root (Compose NavHost) and forwards navigation events to Swift:

```swift
@available(iOS 26.0, *)
struct NativeNavComposeView: UIViewControllerRepresentable {
    let topLevelRoute: TopLevelRoute
    let coordinator: TabNavigationCoordinator
    let appCoordinator: AppNavigationCoordinator

    func makeUIViewController(context: Context) -> UIViewController {
        return Main_iosKt.MainViewController(
            topLevelRoute: topLevelRoute,
            onNavigate: { route in self.coordinator.push(route) },
            onGoBack: { self.coordinator.pop() },
            onSet: { route in
                self.coordinator.popToRoot()
                if let topLevel = route as? TopLevelRoute {
                    self.appCoordinator.activateTab(for: topLevel)
                } else {
                    self.coordinator.push(route)
                }
            },
            onActivate: { route in self.appCoordinator.activateTab(for: route) }
        )
    }

    func updateUIViewController(_ uiViewController: UIViewController, context: Context) {
    }
}
```

`DetailComposeView` renders a single detail screen for each `NavigationStack` destination:

```swift
@available(iOS 26.0, *)
struct DetailComposeView: UIViewControllerRepresentable {
    let route: AppRoute
    let coordinator: TabNavigationCoordinator
    let appCoordinator: AppNavigationCoordinator

    func makeUIViewController(context: Context) -> UIViewController {
        return Main_iosKt.ScreenViewController(
            route: route,
            onNavigate: { newRoute in self.coordinator.push(newRoute) },
            onGoBack: { self.coordinator.pop() },
            onSet: { route in
                self.coordinator.popToRoot()
                if let topLevel = route as? TopLevelRoute {
                    self.appCoordinator.activateTab(for: topLevel)
                } else {
                    self.coordinator.push(route)
                }
            },
            onActivate: { route in self.appCoordinator.activateTab(for: route) }
        )
    }

    func updateUIViewController(_ uiViewController: UIViewController, context: Context) {
    }
}
```

### Wrap each tab in a `NavigationStack`

```swift
@available(iOS 26.0, *)
struct TabContentView: View {
    let topLevelRoute: TopLevelRoute
    let coordinator: TabNavigationCoordinator
    let appCoordinator: AppNavigationCoordinator
    let title: String

    var body: some View {
        NavigationStack(path: Binding(
            get: { coordinator.path },
            set: { coordinator.path = $0 }
        )) {
            NativeNavComposeView(
                topLevelRoute: topLevelRoute,
                coordinator: coordinator,
                appCoordinator: appCoordinator
            )
                .ignoresSafeArea(.all)
                .navigationTitle(title)    // read by iOS 26 for the tab bar label
                .navigationBarHidden(true) // Compose renders its own tab root header
                .navigationDestination(for: RouteWrapper.self) { wrapper in
                    DetailComposeView(
                        route: wrapper.route,
                        coordinator: coordinator,
                        appCoordinator: appCoordinator
                    )
                        .ignoresSafeArea(.all)
                        .navigationTitle(wrapper.route.title ?? "")
                        .navigationSubtitle(wrapper.route.subtitle ?? "")
                        .toolbarTitleDisplayMode(.inline)
                }
        }
    }
}
```

`.navigationTitle(title)` must be set on the tab root view even when `.navigationBarHidden(true)` is
applied. iOS 26 reads this value to label the tab in the floating tab bar.

### Assemble the `TabView`

```swift
@available(iOS 26.0, *)
struct NativeNavContentView: View {
    @State private var appCoordinator = AppNavigationCoordinator()

    var body: some View {
        TabView(selection: Binding(
            get: { appCoordinator.selectedTab },
            set: { appCoordinator.selectedTab = $0 }
        )) {
            Tab(String(localized: "Schedule"), systemImage: "clock",
                value: AppNavigationCoordinator.AppTab.schedule) {
                TabContentView(topLevelRoute: ScheduleScreen(),
                               coordinator: appCoordinator.scheduleCoordinator,
                               appCoordinator: appCoordinator, title: String(localized: "Schedule"))
            }
            Tab(String(localized: "Info"), systemImage: "info.circle",
                value: AppNavigationCoordinator.AppTab.info) {
                TabContentView(topLevelRoute: InfoScreen(),
                               coordinator: appCoordinator.infoCoordinator,
                               appCoordinator: appCoordinator, title: String(localized: "Info"))
            }
        }
        // Enables the liquid glass floating tab bar
        .tabBarMinimizeBehavior(.automatic) 
        .tint(Color(.accent))
    }
}
```

With two tabs, the app will look as follows:

![Two tabs](ios-kotlinconf-two-tabs.png){ width="250" style="block"}

The same pattern scales to any number of tabs.

### Add the iOS version gate in `ContentView`

```swift
struct ContentView: View {
    var body: some View {
        if #available(iOS 26.0, *) {
            NativeNavContentView()
        } else {
            ComposeView(topLevelRoute: ScheduleScreen())
                .ignoresSafeArea(.all)
        }
    }
}
```

The complete file is at [`app/iosApp/iosApp/ContentView.swift`](https://github.com/JetBrains/kotlinconf-app/blob/lg-nav/app/iosApp/iosApp/ContentView.swift).

## What's next

* Check out the [Official KotlinConf application](https://github.com/JetBrains/kotlinconf-app/tree/lg-nav) with the Liquid Glass effect applied.
* See [Adopting Liquid Glass](https://developer.apple.com/documentation/TechnologyOverviews/adopting-liquid-glass), Apple's overview of the new material and adoption checklist.
* Refer to [Integration with the SwiftUI framework](compose-swiftui-integration.md) for the official guidance on using Compose Multiplatform inside 
SwiftUI and embedding SwiftUI inside a Compose Multiplatform app.