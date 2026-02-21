---
name: Android Code Review
dcc_uri: dev/prompts/fe_android_codereview
description: Review Android Kotlin/Compose code against project-specific rules
invokable: true
version: '0.1'
dcc_definition_type: prompt
dcc_tags:
  - dev
  - frontend
  - mobile
  - android
---

Review the selected Android code. Apply our core, frontend-core, and Android-specific rules.

Check specifically for:

**Kotlin correctness**
- `!!` (non-null assertion) in production code
- `var` used where `val` is correct
- Sentinel values used instead of `null` or a sealed type
- Mutable state exposed publicly from a ViewModel (`MutableStateFlow`, `MutableLiveData`)

**Coroutines & Flow**
- Coroutines launched outside `viewModelScope` / `lifecycleScope`
- `lifecycleScope.launch { flow.collect {} }` without `repeatOnLifecycle` — lifecycle-unsafe collection
- `StateFlow` not using `WhileSubscribed(5_000)` for screen-level state
- `GlobalScope` usage
- Hardcoded `Dispatchers.IO` / `Dispatchers.Default` in business logic (should be injected)

**Compose**
- Business logic or repository calls inside a composable body
- `remember` used where `rememberSaveable` is needed (survives process death) or vice versa
- State over-hoisted into ViewModel when it's purely ephemeral UI state
- ViewModel state collected deep inside a child composable instead of at screen level
- `LaunchedEffect` without a meaningful key
- Missing `Loading`, `Error`, or `Empty` state handling

**Architecture**
- ViewModel holding a `Context`, `Activity`, or `View` reference
- Multiple separate `StateFlow` properties for the same screen instead of a single state class
- Business logic inline in the ViewModel instead of delegated to a use-case or repository

Format: Critical → Important → Minor. Be direct.
