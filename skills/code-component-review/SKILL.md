---
name: code-component-review
description: Code audit skill for Power Apps Component Framework (PCF) code components (also called PCF Controls). Use when the user wants a review, audit, or pre-submission check of a Field Control or Dataset Control — before deployment, after a build, or when debugging lifecycle bugs. Audits for React memory leaks (unmounted roots, orphaned listeners/timers/subscriptions), ControlManifest.Input.xml accuracy vs. the code, Fluent UI v9 provider wrapping and theming, disabled/read-only and accessibility handling, and dataset pagination correctness. Reports findings by severity; does not rewrite code unless asked. Trigger on requests like "review my PCF control", "audit this code component", "why does my control leak memory / not update", or "check my control before I ship it".
metadata:
  suite: pp-code-component-skills
---

# Code Component Review

Audit a PCF code component and report findings by severity. "PCF Control" and "Code Component" are interchangeable. Report only — do not modify code unless the user explicitly asks for fixes.

## Workflow

1. **Locate the project.** Find `ControlManifest.Input.xml`, `index.ts` (or the entry point it references), `.tsx` components, and `__tests__/`. Read them all before judging anything.

2. **Run the five audit passes below.** For each finding, record: severity (Critical / Warning / Info), file:line, what is wrong, and the concrete fix.

3. **Verify with commands** where applicable:
   ```pwsh
   npm test
   npm run build
   ```
   A failing build or test suite is automatically a Critical finding. Check the build output text for a `[build] Failed` line, not just the process exit code — `pcf-scripts build` can print that for a failed sub-step (e.g. an ESLint error) while the wrapping npm process still exits 0.

4. **Report** in this order: Critical → Warning → Info → Passed checks. End with a one-line verdict: ship / fix-then-ship.

## Audit pass 1 — Memory leaks

- **Standard (DOM) controls**: if the code calls `ReactDOM.createRoot`, `destroy()` MUST call `root.unmount()`. Missing unmount = Critical.
- **Virtual controls** (`ComponentFramework.ReactControl`): the framework owns unmounting. Flag any `ReactDOM.createRoot`/`ReactDOM.render` usage as Critical — it means the control is mis-architected.
- Every `addEventListener` (DOM or `window`), `setInterval`/`setTimeout`, `ResizeObserver`/`MutationObserver`, and API subscription must have a matching cleanup in `destroy()` or a React `useEffect` cleanup. Orphans = Critical.
- Async callbacks (fetch, WebAPI calls) must not set state after destroy; look for a disposed/cancelled flag.

## Audit pass 2 — Manifest accuracy

- Every `<property>` in `ControlManifest.Input.xml` must be read or written in code, and every `context.parameters.X` referenced in code must exist in the manifest. Mismatches = Critical (runtime binding failures).
- `of-type` must match how the value is consumed (e.g. don't declare `Whole.None` then treat the raw value as a string).
- Outputs returned from `getOutputs()` must be declared as `bound` or `output` properties.
- `<feature-usage>` entries must correspond to APIs actually called (e.g. `Utility` for `context.utils`, `WebAPI` for `context.webAPI`). Unused features = Warning (unnecessary permission prompts); used-but-undeclared = Critical.
- `control-type` must be `virtual` for React Virtual Controls; `version` should not be `0.0.x` for a control about to ship = Info.

## Audit pass 3 — Fluent UI v9 provider & theming

- The React tree returned by `updateView` must be wrapped in `FluentProvider`. Missing provider = Critical (unstyled, broken components).
- The theme should prefer `context.fluentDesignLanguage?.tokenTheme` (host-provided theme — note `.tokenTheme`, not the `FluentDesignState` wrapper itself) with a `webLightTheme` fallback. Hardcoded `webLightTheme` only = Warning (control won't match dark/custom host themes). Passing `context.fluentDesignLanguage` directly to `theme` without `.tokenTheme` = Critical (wrong shape, breaks styling).
- Components must come from `@fluentui/react-components` (v9), not `@fluentui/react` (v8). v8 imports = Warning.
- Styling via `makeStyles`/tokens preferred; flag global CSS selectors or `!important` hacks = Warning.

## Audit pass 4 — Access control & accessibility

- The control must honor `context.mode.isControlDisabled` (render disabled/read-only). Ignoring it = Critical — the host will show the control as editable on read-only forms.
- Bound values must be treated as untrusted: no `dangerouslySetInnerHTML` on parameter values = Critical.
- Interactive elements need keyboard support and ARIA labeling (Fluent UI v9 components provide most of this; custom elements must add it). Missing = Warning.
- Value flow must go through `notifyOutputChanged` + `getOutputs`; direct mutation of `context.parameters` = Critical.

## Audit pass 5 — Dataset pagination (Dataset Controls only)

- Next/prev page actions must guard on `paging.hasNextPage` / `paging.hasPrevPage` before calling `loadNextPage()` / `loadPrevPage()` / `loadExactPage(n)`. Unguarded calls = Warning.
- Row rendering must iterate `dataset.records` and use `record.getValue(<column>)` / `getFormattedValue(<column>)`; hardcoded column logic not matching the manifest's `<data-set>` = Critical.
- `dataset.refresh()` after actions that mutate data; `totalResultCount` used for page indicators where the spec requires it.
- Selection must use `dataset.getSelectedRecords()` / `setSelectedRecordIds()`, not parallel local state that can drift = Warning.

## Constraints

- Be surgical: audit, don't refactor. Mention unrelated dead code in the report; never delete it.
- Every Critical finding must cite file:line and a concrete fix — no vague "consider improving" items.
- If tests are missing for a Critical-risk area (destroy cleanup, disabled state, pagination), note it as a Warning and suggest `code-component-build` for the TDD fix.

## Example user prompts

- "Review my PCF control before I deploy it."
- "Audit this dataset code component for leaks."
- "My control doesn't reflect form read-only state — check it."
- "Give this code component a pre-submission check."
