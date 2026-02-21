---
alwaysApply: false
name: iOS Swift & SwiftUI Rules
dcc_uri: dev/rules/ios-rules
description: >-
  iOS development rules for Swift and SwiftUI (iOS 17+). Extends core and
  frontend-core rules.
version: '1.1'
dcc_definition_type: rule
dcc_tags:
  - dev
  - frontend
  - ios
  - swift
  - swiftui
  - mobile
---
# iOS Swift & SwiftUI Rules (iOS 17+)

These rules apply to all iOS projects and extend the core and frontend-core rules.

---

## Swift Language

- Prefer `let` over `var` everywhere; only use `var` when mutation is genuinely required.
- Prefer value types (`struct`, `enum`) over reference types (`class`); reach for `class` only when you need identity, shared mutable state, or lifecycle managed by a framework.
- Never force-unwrap (`!`) in production code. Use `guard let`, `if let`, or provide a sensible default via `??`. The only acceptable exception is IBOutlets in UIKit legacy code.
- Avoid `try!` and `as!`; handle errors and failed casts explicitly.
- Use `guard` for early exits to reduce nesting; keep the happy path at the lowest indentation level.
- Model absence explicitly with `Optional` — don't use sentinel values (`-1`, `""`, `"N/A"`) to mean "no value".
- Use `async/await` for all new async code. Do not introduce new Combine chains or completion-handler APIs unless integrating with an existing layer that requires it.
- Prefer `throws` over returning `Result<T, E>` for immediate call-sites; use `Result` when you need to store or pass the outcome.
- Use `enum` with associated values to model states that are mutually exclusive (e.g. `loading`, `loaded(Data)`, `failed(Error)`); never use multiple optional properties to represent the same concept.

---

## SwiftUI Views

- Keep `View` bodies focused. If a `body` needs more than ~3–4 composed sub-expressions, extract a subview or a `@ViewBuilder` helper.
- Extract subviews aggressively — SwiftUI diffing is cheap; large monolithic bodies are not.
- Never put business logic, data fetching, or side effects inside `body`. Views are a function of state, nothing more.
- Avoid storing state that belongs to the model layer in `@State`. `@State` is for ephemeral, view-local UI state (focus, animation phase, sheet presentation).
- Prefer `private` on all `@State` and `@StateObject` properties — they are implementation details of the view.
- Do not pass `ObservableObject` instances deep through the view tree as parameters; use the environment (`@EnvironmentObject` / `.environment()`) for widely shared objects.
- Use `.task {}` for async work tied to a view's lifetime instead of `onAppear` + a detached `Task`.
- Be deliberate about animation: apply `.animation(_:value:)` with an explicit value rather than the global `.animation()` modifier, which can produce unexpected side effects.

---

## Architecture & State

- Give every screen or feature a dedicated observable (`@Observable` / `ObservableObject`) that owns its state and intent handling. Views should not decide what happens — they should report events.
- Keep the observable (ViewModel, Store, etc.) free of UIKit/SwiftUI imports where possible. Pure Swift makes it trivially testable.
- Side effects (network, persistence, location, etc.) belong in dedicated services injected into the observable, not called inline.
- Model navigation state explicitly rather than scattering `@State var isShowingX` flags. A single `NavigationPath` or a route enum owned by the observable is easier to reason about and test.
- Prefer dependency injection over singletons. Pass services through initializers or the SwiftUI environment; avoid `MyService.shared` deep in view models.

---

## Concurrency

- Always annotate `@MainActor` on types (ViewModels, ObservableObjects) that own UI-bound state. Do not manually dispatch to `DispatchQueue.main`.
- Isolate actor-crossing calls explicitly; do not assume a function is on the main actor unless annotated.
- Avoid `Task.detached` unless you explicitly need to escape the current actor context; prefer `Task {}` which inherits the current actor.
- Treat `Task` handles as resources: cancel them in `onDisappear` / `deinit` when they represent ongoing work tied to a view's or object's lifecycle.

---

## Code Organisation

- Group file contents consistently: types → lifecycle/init → public interface → private helpers. Prefer `// MARK: -` sections for anything beyond ~50 lines.
- One primary type per file. Extensions on that type can live in the same file if small, or in a separate `TypeName+Purpose.swift` file if substantial.
- Prefer small focused extensions over large feature-dump files. An extension should have a clear, single responsibility.
- Keep `Preview` code at the bottom of the file and wrap it in `#if DEBUG` to keep it out of release builds.
- Name files after the primary type they contain. If you need to search for `ContentViewModel`, you should find it in `ContentViewModel.swift`.
