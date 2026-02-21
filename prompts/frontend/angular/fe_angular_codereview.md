---
name: Angular Code Review
dcc_uri: dev/prompts/fe_angular_codereview
description: Review Angular code against project-specific rules
invokable: true
version: '0.1'
dcc_definition_type: prompt
dcc_tags:
  - dev
  - frontend
  - web
  - angular
---

Review the selected Angular code. Apply our core, frontend-core, and Angular-specific rules.

Check specifically for:

**Angular correctness**
- `@Input()`/`@Output()` decorators used instead of `input()`/`output()` signals
- Missing `ChangeDetectionStrategy.OnPush`
- NgModule usage where standalone should be used
- `ngOnChanges` used to react to inputs instead of `effect()` or `computed()`
- Logic or data fetching inside the template or `constructor`

**Reactivity**
- State stored in plain properties instead of `signal()`
- Derived values stored in state instead of `computed()`
- Writing to a signal inside `computed()`
- RxJS used where a `computed()` or `signal()` would be simpler and sufficient

**RxJS (where used)**
- Missing `takeUntilDestroyed()` on subscriptions
- Wrong flattening operator for the use case (e.g. `mergeMap` on a form submit that should be `exhaustMap`)
- Nested `subscribe()` calls

**Performance**
- Function calls in templates that are not memoised via `computed()` or pipe
- Missing `trackBy` / `track` on list renders
- Missing `@defer` on heavy components not needed on initial render

Format: Critical → Important → Minor. Be direct.
