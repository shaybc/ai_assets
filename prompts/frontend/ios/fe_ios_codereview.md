---
name: iOS Code Review
dcc_uri: dev/prompts/fe_ios_codereview
description: Review Swift/SwiftUI code against project-specific rules
invokable: true
version: '0.1'
dcc_definition_type: prompt
dcc_tags:
  - dev
  - frontend
  - mobile
  - ios
---

Review the selected Swift/SwiftUI code. Apply our core, frontend-core, and iOS-specific rules.

Check specifically for:

**Swift correctness**
- Force-unwraps (`!`) or `try!` / `as!` in production code
- `var` used where `let` would be correct
- Sentinel values (`-1`, `""`) used instead of `Optional`
- `Optional` used as a field type, collection element, or parameter (should be return-type only)
- `class` used where a `struct` is the right choice

**SwiftUI**
- Business logic, data fetching, or `Task` creation directly inside `body`
- `@State` holding model-layer data instead of ephemeral UI state
- `onAppear` + detached `Task` instead of `.task {}`
- Missing loading, error, or empty state handling
- Monolithic `body` that should be extracted into subviews

**Concurrency**
- Missing `@MainActor` on types that own UI-bound state
- Manual `DispatchQueue.main.async` instead of `@MainActor`
- `Task.detached` used without clear justification
- `Task` handles not cancelled on view disappear or `deinit`

**Architecture**
- ViewModel holding `UIViewController`, `UIView`, or SwiftUI view references
- Singleton `MyService.shared` accessed directly inside a ViewModel
- Navigation state scattered as multiple `@State var isShowingX` booleans

Format: Critical → Important → Minor. Be direct.
