# pp-code-component-skills

A set of four Agent Skills, following the Agent Skills specification, that cover the full lifecycle of building a **Power Apps Component Framework (PCF)** code component — also known as a PCF Control — from a blank slate to a pre-ship audit.

Each skill hands off to the next automatically. Ask for any one of them, or just ask for the end result ("build me a PCF control that shows X"), and your agent will invoke design → init → build → review in order, gated on each step's output.

## The skills

| Skill | What it does | Reads | Writes |
|---|---|---|---|
| [`code-component-design`](skills/code-component-design/SKILL.md) | Interviews you about the control's purpose, data types, properties, UI, and events, then writes a spec. | — | `design-spec.md` |
| [`code-component-init`](skills/code-component-init/SKILL.md) | Scaffolds the project (`pac pcf init`), installs Fluent UI v9 / React / Jest, configures the manifest from the spec. | `design-spec.md` | `ControlManifest.Input.xml`, project scaffold |
| [`code-component-build`](skills/code-component-build/SKILL.md) | TDD-implements the control: tests first (Jest + React Testing Library), then the `ComponentFramework.ReactControl` lifecycle. | `design-spec.md`, scaffold | `index.ts`, `.tsx` components, tests |
| [`code-component-review`](skills/code-component-review/SKILL.md) | Read-only pre-ship audit: memory leaks, manifest accuracy, Fluent UI theming, accessibility, dataset pagination. Reports a ship / fix-then-ship verdict. | finished project | audit report only |

Each skill's trigger conditions and full workflow are documented in its own `SKILL.md`.

### Design principles

- **React Virtual Controls only** (`ComponentFramework.ReactControl`), not standard DOM-injected controls — the framework owns the React root, which eliminates a whole class of mount/unmount memory leaks.
- **Fluent UI v9** (`@fluentui/react-components`), themed via `context.fluentDesignLanguage?.tokenTheme` with a `webLightTheme` fallback — not hardcoded, so controls match the host's light/dark/custom theme.
- **Minimal manifest surface** — every property is public API that's hard to change later, so the design skill pushes back on unnecessary ones.
- **Tests before code.** The build skill writes failing tests against the spec's Test Plan first, confirms they fail for the right reason, then implements.
- **Scope discipline between skills.** Init scaffolds but never implements; build implements but never scaffolds; review reports but never rewrites (unless you explicitly ask).

## Requirements

- [Power Platform CLI](https://aka.ms/PowerPlatformCLI) (`pac`) — install via `dotnet tool install --global Microsoft.PowerApps.CLI.Tool`, or the Power Platform Tools VS Code extension.
- Node.js LTS and npm.
- Network access for `pac pcf init` and `npm install`.

## Installation

This repo is packaged two ways: as a Claude Code plugin (with its own marketplace) and as a [pi](https://pi.dev) skills package published to npm. Both read the same `skills/` directory — nothing is duplicated.

### Claude Code

```
/plugin marketplace add byronmatus/pp-code-component-skills
/plugin install pcf-controls@pp-code-component-skills
```

Or, without the plugin system, just drop `skills/` into your project-level `.claude/skills/` or global `~/.claude/skills/` — Claude picks up each `SKILL.md` automatically based on its `description` frontmatter.

### pi

Published to npm with the `pi-package` keyword, so it's listed on [pi.dev/packages](https://pi.dev/packages):

```
pi install npm:pp-code-component-skills
```

### Other agents

The skill format is just a folder with a `SKILL.md` (YAML frontmatter + Markdown body) per the Agent Skills specification, so any agent that reads that format can use `skills/` directly — no adapter needed.

## Example prompts

- "Design a PCF control that shows a satisfaction score as a slider."
- "Scaffold the PCF control from my design spec."
- "Implement the control from design-spec.md — tests first."
- "Review my PCF control before I ship it."
- Or just: "Build me a PCF field control for a priority badge with a dropdown to change it." — Claude will chain all four skills as needed.

## Evals

Each skill has an eval suite under `skills/<name>/evals/evals.json` (plus `evals/full-pipeline/` for an end-to-end integration eval chaining all four). These were built and run with the [skill-creator](https://github.com/anthropics/claude-skills) workflow: a with-skill run is compared against an unguided baseline agent doing the same task, graded against a fixed set of assertions.

Across all four skills, the recurring pattern was that an unguided baseline tends to either **over-scope** (write more tests/code than the spec asked for) or **under-commit** (defer a design decision behind a list of questions instead of resolving it) — exactly the discipline these skills are meant to enforce.
