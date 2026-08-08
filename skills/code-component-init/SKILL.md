---
name: code-component-init
description: Project scaffolding skill for Power Apps Component Framework (PCF) code components (also called PCF Controls). Use when the user wants to create, initialize, or set up a new code component project. Runs pac pcf init with the React framework template (field or dataset), installs Fluent UI v9 / React / Jest / React Testing Library packages, configures ControlManifest.Input.xml properties and resources from design-spec.md, and creates test scaffolding. Trigger on requests like "scaffold a PCF control", "set up a new code component project", "run pac pcf init for me", or when code-component-build finds no initialized project.
compatibility: Requires Power Platform CLI (pac), Node.js LTS, and npm. Network access needed for npm installs.
metadata:
  suite: pp-code-component-skills
  consumes: design-spec.md
---

# Code Component Init

Scaffold a PCF code component project from `design-spec.md`. "PCF Control" and "Code Component" are interchangeable.

## Prerequisites check

1. Verify the tooling exists before doing anything else:
   ```pwsh
   pac --version
   node --version
   npm --version
   ```
   If `pac` is missing, try to install it. If you are not able to install it, tell the user to install the Power Platform CLI (https://aka.ms/PowerPlatformCLI) and stop.

2. Read `design-spec.md` from the workspace root. If it does not exist, stop and invoke the `code-component-design` skill first — do not guess the manifest properties.

## Workflow

### 1. Initialize the project

Create a new empty directory for the project and run:

```pwsh
mkdir <ControlName>; cd <ControlName>
pac pcf init --namespace <Namespace> --name <ControlName> --template <field|dataset> --framework react --run-npm-install
```

- `--template field` for Field Controls, `dataset` for Dataset Controls (from the spec).
- `--framework react` is required: it generates a React Virtual Control (`ComponentFramework.ReactControl`).
- Derive `<Namespace>` and `<ControlName>` from the spec; confirm with the user if absent. Names must be alphanumeric, no spaces or hyphens.

### 2. Install dependencies

```pwsh
npm install @fluentui/react-components
npm install --save-dev jest jest-environment-jsdom ts-jest @types/jest @testing-library/jest-dom @testing-library/user-event identity-obj-proxy
```

Note: `react` and `react-dom` ship with the pac template for virtual controls; only add them explicitly if `package.json` lacks them (check first, do not blindly install).

Check the React major version the template pinned before adding React Testing Library:

```pwsh
npm ls react --depth=0
```

RTL's current major requires React 18/19; the pac template still pins React 16. Installing `@testing-library/react` unpinned against React 16 hits an npm `ERESOLVE` conflict — and forcing it through with `--legacy-peer-deps` can silently drop unrelated peer deps (e.g. `eslint`, which `pcf-scripts` needs at build time) rather than actually resolving anything. Pin the major to match instead of overriding the resolver:

```pwsh
npm install --save-dev @testing-library/react@^12   # React 16/17 templates
# npm install --save-dev @testing-library/react     # React 18+ templates, unpinned
```

### 3. Configure ControlManifest.Input.xml

Edit the generated `ControlManifest.Input.xml` to match the spec:

- `<control>`: confirm `namespace`, `constructor`, `control-type="virtual"`, and bump `version` only on subsequent releases, never on first scaffold.
- `<property>` elements: one per spec property, with correct `name`, `display-name-key`, `of-type`, `usage` (`bound` | `input` | `output`), and `required`.
- Dataset Controls: configure the `<data-set>` element (`name`, `display-name-key`) and keep `cds-data-set-options` as generated.
- `<resources>`: ensure `<code path="index.ts" order="1"/>`; add `<css>` only if the spec requires custom styles (prefer Fluent UI `makeStyles` and skip CSS files).
- `<feature-usage>`: add `uses-feature` entries (e.g. `Utility`, `WebAPI`, `Device`) ONLY if the spec actually calls those APIs. Never enable features speculatively.

### 4. Set up test scaffolding

Create `jest.config.js` at the project root:

```js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'jest-environment-jsdom',
  roots: ['<rootDir>/<ControlName>'],
  testMatch: ['**/*.test.ts?(x)'],
  setupFilesAfterEnv: ['<rootDir>/jest.setup.ts'],
  moduleNameMapper: { '\\.(css|less|scss)$': 'identity-obj-proxy' },
};
```

The explicit `testMatch` matters: Jest's default pattern treats *every* file under any `__tests__/` folder as a test suite, not just files named `*.test.ts`. Without overriding it, the `contextMock.ts` factory below gets swept up and run as an empty test suite once real specs exist, failing with "must contain at least one test." Scoping `testMatch` to `*.test.ts?(x)` lets `contextMock.ts` sit alongside real tests without being treated as one.

Create `jest.setup.ts`:

```ts
import '@testing-library/jest-dom';
```

Add to `package.json` scripts: `"test": "jest"`.

Create `<ControlName>/__tests__/contextMock.ts` exporting a factory that builds a minimal `ComponentFramework.Context<IInputs>` mock (parameters bag, `mode.isControlDisabled`, `mode.allocateRowHeight` for dataset, `userSettings`, and a `jest.fn()` for `notifyOutputChanged`). Keep it minimal — extend only when tests need more.

### 5. Verify

```pwsh
npm run build
npx jest --showConfig
```

The build must succeed. No test files exist yet — that's expected, since writing them is `code-component-build`'s job (TDD red comes from that skill, not from init). Use `npx jest --showConfig` instead of `npm test` to confirm the Jest config itself resolves cleanly (preset, transform, moduleNameMapper) without tripping Jest's "no tests found" exit code. Fix any scaffolding or config errors before finishing.

## Constraints & best practices

- Never run `pac pcf init` in a non-empty directory; it will fail or clobber files.
- Do not modify generated files beyond what the spec requires — no gratuitous reformatting of `pcfconfig.json`, `tsconfig.json`, or the generated class.
- Do not implement component logic here; that is `code-component-build`'s job.
- Pin nothing and add no packages beyond the lists above unless the spec demands it.

## Example user prompts

- "Scaffold the PCF control from my design spec."
- "Set up a new dataset code component project with React and Fluent."
- "Run pac pcf init and get Jest working for this control."
- "Create the project for the slider field control we designed."
