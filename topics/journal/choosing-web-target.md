# Choose the right web target for a Kotlin Multiplatform project

Kotlin Multiplatform (KMP) offers two approaches for web development:

- JavaScript-based (using the Kotlin/JS compiler)

- WebAssembly-based (using the Kotlin/Wasm compiler)

Both options let you use shared code in your web application. However, they differ in important ways, including performance, interoperability, application size, and target browser support. This guide explains when to use each target and how to satisfy your requirements with the appropriate choice.

### Quick guide

The table below summarizes the recommended target based on your use case.



| Use case                              | Recommended target | Reasoning                                                                                                                                                                                                                     |
| ------------------------------------- | ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Sharing business logic, but native UI | JS                 | Offers straightforward interop with JavaScript and minimal overhead                                                                                                                                                          |
| Sharing both UI and business logic    | Wasm               | Provides better performance for rendering with [Compose Multiplatform](https://www.jetbrains.com/compose-multiplatform/)                                                                                                      |
| Non-shareable UI                      | JS                 | Allows building UIs with HTML-based frameworks like [Kobweb](https://kobweb.varabyte.com/), [Kilua](https://kilua.dev/), or [React](https://kotlinlang.org/docs/js-react.html), leveraging existing JS ecosystems and tooling |

## When to choose Kotlin/JS

Kotlin/JS provides a great solution if your goal is to:

- [Share business logic with a JavaScript/TypeScript codebase](#share-business-logic-with-a-javascript-typescript-codebase)

- [Build non-shareable web apps with Kotlin](#build-non-shareable-web-apps-with-kotlin)

### Share business logic with a JavaScript/TypeScript codebase

If you want to share a chunk of Kotlin code (like domain or data logic) with a native JavaScript/TypeScript-based application, JS target offers:

- Straightforward interop with JavaScript/TypeScript.

- Minimal overhead in interoperability (for example, no unnecessary data copying). This helps your code integrate seamlessly into your JS-based workflow.

  
### Build non-shareable web apps with Kotlin

For teams looking to build entire web apps using Kotlin, but without the intention to share it to other platforms (iOS, Android, or Desktop), an HTML-based solution could be a better choice. It improves SEO and accessibility, and provides seamless browser integration by default (such as “find on page” functionality or translating a page). In this case, the Kotlin/JS provides several options. You can:

- Utilize Compose HTML-based frameworks, such as [Kobweb](https://kobweb.varabyte.com/) or [Kilua](https://kilua.dev/), to build UIs using a familiar Compose Multiplatform architecture.

- Take advantage of React-based solutions with Kotlin wrappers to build [React components in Kotlin](https://kotlinlang.org/docs/js-react.html).

## When to choose Kotlin/Wasm

### Build cross-platform apps with Compose Multiplatform

If you want to share both logic and UI across multiple platforms — including web — Kotlin/Wasm combined with [Compose Multiplatform](https://www.jetbrains.com/compose-multiplatform/) is the way to go:

- The UI experience is more consistent across platforms.

- You can take advantage of Wasm for improved rendering and smooth, responsive animations.

- Browser support for [WasmGC](https://developer.chrome.com/blog/wasmgc) has matured, allowing Kotlin/Wasm to run on all major modern browsers with near-native performance.

For projects with requirements for older browser version support, you can use a compatibility mode for Compose Multiplatform: build your UI in Wasm for modern browsers, but gracefully fall back to JS on older browsers. You can also share the common logic between Wasm and JS targets in your project.

——————————————————————————————————————————

Still not sure which route to take? Join our [Slack community](https://slack-chats.kotlinlang.org) and ask about key differences, performance considerations, and best practices for choosing the right target.
