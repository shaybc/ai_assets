---
alwaysApply: false
name: Angular Base Rules
dcc_uri: dev/rules/angular_base_rules
description: Angular-specific development rules (v19+). Extends core and frontend-core rules.
version: '1.0.0'
dcc_definition_type: rule
dcc_tags:
  - dev
  - frontend
  - angular
---

# Angular Rules (v19+)
These rules apply to all Angular projects and extend the core and frontend-core rules.

## 1) Components
- MUST use `input()`, `output()`, and `model()` signals instead of `@Input()` / `@Output()` decorators.
- MUST default to `ChangeDetectionStrategy.OnPush` on every component.
- MUST prefer standalone components; avoid NgModules unless integrating legacy code.
- MUST not use `ngOnChanges` to react to input changes — use `effect()` or `computed()` on signal inputs instead.
- MUST keep templates logic-free: move conditions and transformations into the component class or a `computed()`.

## 2) Signals & Reactivity
- MUST use signals (`signal()`, `computed()`, `effect()`) as the default reactivity primitive.
- MUST not write to a signal inside `computed()` — computed values are derived, not side effects.
- MUST scope `effect()` to a reactive context (component/directive); avoid creating effects in services unless clearly justified.
- MUST clean up effects and subscriptions that are not automatically managed (e.g. manual `effect()` refs).
- SHOULD prefer `computed()` over manual `effect()` when the goal is deriving a value.

## 3) RxJS
- MUST use RxJS only for inherently async/event-driven flows (HTTP, WebSockets, complex event streams); prefer signals for local/UI state.
- MUST use `takeUntilDestroyed()` (from `@angular/core/rxjs-interop`) for subscription cleanup; avoid manual `Subject`/`takeUntil` teardown patterns.
- MUST avoid `subscribe()` inside `subscribe()` — use `switchMap` / `mergeMap` / `concatMap` as appropriate.
- MUST choose the correct flattening operator: `switchMap` for cancellable (e.g. search), `concatMap` for ordered, `mergeMap` for parallel, `exhaustMap` for non-interruptible (e.g. form submit).
- MUST not use `async` pipe and manual `subscribe()` for the same stream in the same component.

## 4) Services & Dependency Injection
- MUST use `providedIn: 'root'` for singleton services; use component/route-level providers only when explicit scoping is needed.
- MUST use the `inject()` function instead of constructor injection in new code.
- MUST not perform side effects (HTTP calls, subscriptions) in a service constructor — use explicit init methods or lazy patterns.

## 5) Routing
- MUST use lazy-loaded routes (`loadComponent` / `loadChildren`) for all feature areas.
- MUST use route-level resolvers or `ResolveFn` to load data before navigation; avoid loading data inside `ngOnInit` triggered by route activation.
- MUST use typed router params (`withComponentInputBinding()`) instead of manually injecting `ActivatedRoute` where possible.

## 6) Forms
- MUST use Reactive Forms for any form with validation, multi-field interaction, or dynamic fields.
- MUST use Template-driven forms only for simple, single-field, low-stakes inputs.
- MUST type `FormControl<T>` explicitly; avoid untyped form APIs (`UntypedFormControl` etc.) in new code.
- MUST not put form logic (validation, submission) in the template — keep it in the component class.

## 7) Performance
- MUST use `@defer` blocks for components that are not needed on initial render.
- MUST use `trackBy` (or the `track` expression in `@for`) on all list renders.
- MUST not call functions in templates that are not pure/memoized — use `computed()` or pipes instead.
- MUST use `NgOptimizedImage` for all `<img>` elements where applicable.
