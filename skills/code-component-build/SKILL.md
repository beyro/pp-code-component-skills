---
name: code-component-build
description: TDD implementation skill for Power Apps Component Framework (PCF) code components (also called PCF Controls). Use when the user wants to implement, code, or add features to a Field Control or Dataset Control in an existing (or to-be-scaffolded) project. Reads design-spec.md (invoking code-component-design if missing, code-component-init if the project is not scaffolded), writes Jest + React Testing Library unit tests first, then implements the ComponentFramework.ReactControl lifecycle (init, updateView, getOutputs, destroy) with Fluent UI v9. Trigger on requests like "implement the PCF control", "build the code component from the spec", "add pagination to my dataset control", or "write tests and code for this PCF project".
compatibility: Requires Power Platform CLI (pac), Node.js LTS, and npm.
metadata:
  suite: pp-code-component-skills
  consumes: design-spec.md
---

# Code Component Build

Implement a PCF code component test-first from `design-spec.md`. "PCF Control" and "Code Component" are interchangeable.

## Workflow

### 1. Gate on prerequisites

- If `design-spec.md` is missing from the workspace root → invoke the `code-component-design` skill.
- If no `ControlManifest.Input.xml` exists → invoke the `code-component-init` skill.
- Read `design-spec.md` fully before writing any code. If the spec contradicts the existing manifest, stop and ask the user which is authoritative.

### 2. Write failing tests first (Red)

For each behavior in the spec's Test Plan, create a test in `<ControlName>/__tests__/`:

- Use the `contextMock.ts` factory from init to build `ComponentFramework.Context<IInputs>` mocks.
- For React Virtual Controls, test the component returned by `updateView` directly: render the returned React element with `@testing-library/react`'s `render()` — do not try to render `updateView`'s internals through the framework.
- Minimum coverage for every control:
  - Renders the bound value from `context.parameters.<prop>.raw`.
  - User interaction calls the captured `notifyOutputChanged` and `getOutputs()` returns the new value.
  - Disabled state (`context.mode.isControlDisabled === true`) renders a disabled/read-only control.
  - `destroy` releases anything the control allocated (listeners, timers, subscriptions).
- Dataset Controls additionally:
  - Renders rows from `context.parameters.<dataset>.records` (each record's `getValue(<column>)`).
  - Pagination: when `paging.hasNextPage` is true, the next-page action calls `paging.loadNextPage()`; same for `loadPrevPage()` / `loadExactPage(n)` if in spec.
  - Record selection calls `navigation.openForm`/`openRecord` only if the spec requires it.

Run `npm test` and confirm the new tests FAIL for the right reason (not import/config errors).

### 3. Implement (Green)

Implement the generated class in `<ControlName>/index.ts` and any extracted `.tsx` components:

- **`init`**: Store `context`, `notifyOutputChanged`, and `state` references. Do NOT render here — virtual controls render in `updateView`. Do NOT touch a container div.
- **`updateView`**: Return the root React element. Wrap it in Fluent UI v9's `FluentProvider`, using the host theme when defined, falling back to `webLightTheme`. `context.fluentDesignLanguage` is a `FluentDesignState` wrapper, not a `Theme` itself — the actual theme object is at its `.tokenTheme` property:
  ```tsx
  <FluentProvider theme={context.fluentDesignLanguage?.tokenTheme ?? webLightTheme}>
    <YourComponent ... />
  </FluentProvider>
  ```
  Read all values fresh from `context.parameters` on every call — `updateView` fires whenever the host updates data.
- **`getOutputs`**: Return an object keyed by output/bound property names, e.g. `{ value: this.currentValue }`. Only called by the framework after `notifyOutputChanged`.
- **`destroy`**: Remove event listeners, clear timers, unsubscribe observables. For virtual controls the framework unmounts the React root — never call `root.unmount()` yourself (there is no root you own), but do clean up anything you allocated.

Keep the component a controlled React component: local state mirrors the bound value, user edits update local state AND stash the pending output value, then call `notifyOutputChanged`.

### 4. Verify the loop

```pwsh
npm test
npm run build
```

Both must pass. If tests fail, fix the implementation (not the test) unless the test contradicts the spec — then flag it to the user.

Don't trust the process exit code alone for `npm run build`: `pcf-scripts build` can print `[build] Failed:` for a failed sub-step (e.g. an ESLint error) while the wrapping npm process still exits 0. Read the build output for a `[build] Failed` line, not just `$LASTEXITCODE`.

### 5. Report

Summarize: behaviors implemented, test count, and any spec deviations. Suggest `msbuild /t:build /restore` or `pac pcf push` only if the user asks about deployment — deploying is out of scope.

## Constraints & PCF best practices

- Never mutate `context.parameters` values directly; flow changes through `notifyOutputChanged` + `getOutputs`.
- Never use `ReactDOM.createRoot` or `ReactDOM.render` in a virtual control.
- Never reach outside the control: no `document.querySelector` on host DOM, no global CSS, no Xrm.Page, no `window` event listeners without a matching removal in `destroy`.
- Dataset Controls: always check `paging.hasNextPage`/`hasPrevPage` before calling load methods, and use `dataset.linking`/`getSelectedRecords` only when the spec requires selection.
- Keep third-party dependencies to those already installed by init; ask before adding more.
- Match the code style of the generated template (naming, exports, IInputs/IOutputs types).

## Example user prompts

- "Implement the control from design-spec.md."
- "Build the dataset card grid — tests first."
- "Add the disabled-state behavior to my PCF control."
- "Wire up pagination for my dataset code component."
