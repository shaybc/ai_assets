---
alwaysApply: false
name: Android Kotlin & Compose
dcc_uri: dev/rules/android_kotlin_rules
description: >-
  Android development rules for Kotlin and Jetpack Compose (API 26+, Compose BOM
  2024+). Extends core and frontend-core rules.
version: '1.2'
dcc_definition_type: rule
dcc_tags:
  - dev
  - frontend
  - android
  - kotlin
  - compose
  - mobile
---
# Android Kotlin & Compose Rules

These rules apply to all Android projects and extend the core and frontend-core rules.

---

## Kotlin Language

- Prefer `val` over `var`; only use `var` when mutation is genuinely required.
- Prefer data classes and sealed classes over raw classes for models and state; use `copy()` for mutations instead of modifying fields.
- Never use `!!` (non-null assertion) in production code. Use `?: return`, `?: throw`, `let {}`, or `requireNotNull()` with a meaningful message.
- Use `sealed class` / `sealed interface` to model mutually exclusive states (e.g. `Loading`, `Success(data)`, `Error(cause)`); never use multiple nullable fields to represent the same concept.
- Prefer named arguments when a function has more than two parameters of the same type or when the meaning of a positional argument is not obvious at the call site.
- Use `object` for singletons and stateless utilities; avoid companion object abuse for things that should be injected.
- Avoid platform-specific types in domain/data layer code — keep pure Kotlin types there so logic stays testable without Android instrumentation.

---

## Coroutines & Flow

- Use `suspend` functions for all new async work. Do not introduce new callback-based or RxJava APIs unless integrating with a layer that requires it.
- Never launch coroutines directly in a ViewModel constructor or `init` block without a `viewModelScope`; always tie coroutine lifetime to an appropriate scope.
- Use `StateFlow` for UI state that is always present and needs a current value. Use `SharedFlow` for one-shot events (navigation commands, toasts). Do not expose raw `MutableStateFlow` / `MutableSharedFlow` from a ViewModel.
- Collect flows in the UI layer using `repeatOnLifecycle(Lifecycle.State.STARTED)` or `collectAsStateWithLifecycle()` in Compose. Never use `lifecycleScope.launch` + `collect` directly, as it does not respect the lifecycle.
- Keep cold flows (from repositories, database, etc.) as `Flow`; only convert to `StateFlow` at the ViewModel boundary with `stateIn(viewModelScope, SharingStarted.WhileSubscribed(5_000), initialValue)`.
- Avoid `GlobalScope`; it has no structured lifetime and leaks work.
- Use `Dispatchers.IO` for disk/network, `Dispatchers.Default` for CPU-intensive work. Do not hardcode dispatchers in business logic — inject a `CoroutineDispatcher` so it can be replaced in tests.

---

## Jetpack Compose

- Keep `@Composable` functions focused. If a composable needs more than ~3–4 distinct visual sections, extract sub-composables.
- Never put business logic, side effects, or data fetching directly inside a composable body. Composables are a function of state, nothing more.
- Use `remember` for values that are expensive to create and must survive recomposition. Use `rememberSaveable` when the value must also survive process death (simple types and `Parcelable` only).
- Hoist state to the lowest common ancestor that needs it; do not over-hoist into the ViewModel what is purely ephemeral UI state (focus, expanded state, animation phase).
- Avoid reading `ViewModel` state deep inside a child composable — collect state at the screen level and pass it down as plain parameters. This keeps sub-composables stateless, reusable, and previewable.
- Use `LaunchedEffect(key)` for side effects tied to composition with a stable key. Use `SideEffect` for synchronizing non-Compose state. Do not fire side effects without a key.
- Use `derivedStateOf {}` when a derived value depends on other state objects and you want to limit recomposition to when the derived result actually changes.
- Mark stable/immutable model classes with `@Stable` or `@Immutable` when the Compose compiler cannot infer stability, to avoid unnecessary recompositions.
- Always provide a `@Preview` for every screen and significant component. Previews are first-class documentation.

---

## Architecture & State

- Follow unidirectional data flow: UI emits events/intents → ViewModel processes and updates state → UI renders state.
- Expose a single `uiState: StateFlow<ScreenState>` from each ViewModel rather than multiple individual `StateFlow` properties for the same screen. A single sealed/data class state is easier to reason about and test.
- ViewModels must not hold references to `Context`, `Activity`, `Fragment`, or any `View`/`Composable`. Use `ApplicationContext` via Hilt if context is unavoidable.
- Keep the ViewModel free of Android UI framework imports where possible. Pure Kotlin logic is trivially testable on the JVM without an emulator.
- Side effects (network, database, sensors, etc.) belong in dedicated repository or use-case classes injected into the ViewModel, not called inline.
- Use use-cases (interactors) to encapsulate single business operations when logic would otherwise be duplicated across ViewModels.

---

## Dependency Injection

- Use Hilt as the standard DI framework. Do not manually wire dependencies with companion objects or service locators in new code.
- Inject interfaces, not concrete implementations, into ViewModels and use-cases. This is the single most impactful thing for testability.
- Scope dependencies deliberately: `@Singleton` for truly app-wide shared state, `@ActivityScoped` / `@ViewModelScoped` for narrower lifetimes. Default to the narrowest scope that works.

---

## Code Organisation

- One primary class per file, named identically to the file (`UserProfileViewModel.kt` contains `UserProfileViewModel`).
- Group file contents consistently: class declaration → properties → init/lifecycle → public interface → private helpers.
- Keep `Preview` composables and test fakes in `debug` or `test` source sets, not in `main`.
- Prefer small focused files over large feature-dump files. If a file exceeds ~200–250 lines, consider whether it has more than one responsibility.
- Package by feature, not by layer. `feature/profile/` containing its ViewModel, state, composables, and repo interface is easier to navigate and own than top-level `viewmodels/`, `repositories/` folders.
