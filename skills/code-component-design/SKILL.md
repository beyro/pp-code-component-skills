---
name: code-component-design
description: Interactive discovery skill for Power Apps Component Framework (PCF) code components (also called PCF Controls). Use when the user wants to plan, scope, or design a new Field Control (single column) or Dataset Control (views/subgrids) before any code exists. Interviews the user about data types, bound vs. input/output properties, UI behavior, Fluent UI v9 usage, and events, then writes a design-spec.md that the code-component-init and code-component-build skills consume. Trigger on requests like "design a PCF control", "plan a code component", "help me spec out a dataset grid control", or when code-component-build finds no design-spec.md.
metadata:
  suite: pp-code-component-skills
  produces: design-spec.md
---

# Code Component Design

Gather requirements for a PCF code component through a structured interview, then produce `design-spec.md` at the workspace root. "PCF Control" and "Code Component" are interchangeable.

## Workflow

1. **Check for existing artifacts.** If `design-spec.md` already exists, ask the user whether to refine it or start over. If a `ControlManifest.Input.xml` already exists, the project is already initialized — tell the user to use `code-component-build` instead, and offer to update the spec only.

2. **Interview the user.** Ask questions in small batches (never one giant list). Cover, in order:
   - **Purpose**: What user problem does the control solve? Where will it be used (model-driven form, canvas app, view/subgrid)?
   - **Component type**: Field Control (binds to one column) or Dataset Control (binds to a view/subgrid). If unclear, ask how many records/columns it must display — one value means Field, a list of records means Dataset.
   - **Bound data**: For Field Controls, the column data type (e.g. `SingleLine.Text`, `Multiple`, `TwoOptions`, `Whole.None`, `Decimal`, `FP`, `Currency`, `DateAndTime.DateOnly`, `OptionSet`, `MultiSelectOptionSet`, `Lookup.Simple`). For Dataset Controls, which columns the grid needs and their types.
   - **Additional properties**: Any static input properties (labels, max length, config JSON) and output properties beyond the bound value.
   - **UI spec**: Layout, interaction states (hover/focus/disabled/error), theming expectations. Default to Fluent UI v9 (`@fluentui/react-components`) and React Virtual Controls (`ComponentFramework.ReactControl`) unless the user objects.
   - **Events & outputs**: What user actions must notify the host (value change, record selection, page change)?
   - **Test expectations**: Key behaviors that must be unit-tested.

3. **Resolve ambiguities explicitly.** If multiple interpretations exist, present them and let the user choose. Do not silently pick. Note every assumption in the spec under an Assumptions heading.

4. **Write `design-spec.md`** to the workspace root using the template below, then summarize the spec back to the user and ask for confirmation before finishing.

## design-spec.md template

```markdown
# Design Spec: <Control Name>

## Overview
<One-paragraph purpose statement.>

## Component Type
<Field Control | Dataset Control> — React Virtual Control (`ComponentFramework.ReactControl`), Fluent UI v9.

## Properties
| Name | Type (manifest `of-type`) | Usage (bound/input/output) | Required | Description |
|------|---------------------------|----------------------------|----------|-------------|
<one row per property; Field Controls must have exactly one bound property; Dataset Controls list the data-set plus any bound/input properties>

## Dataset Requirements (Dataset Controls only)
- Columns needed: <list>
- Pagination: <required? page size?>
- Selection/linking behavior: <record navigation, openRecord, etc.>

## UI Specification
- Layout and visual structure
- Interaction states: default / hover / focus / disabled / error
- Fluent UI v9 components to use
- Theming: use host-provided `context.fluentDesignLanguage` when available, else `webLightTheme`

## Events & Outputs
<What triggers notifyOutputChanged, and which output values are returned from getOutputs.>

## Test Plan
<Bulleted list of unit test cases: rendering, value changes, disabled state, edge cases.>

## Assumptions
<Every assumption made during the interview.>
```

## Constraints & best practices

- Field Controls bind to exactly one column; if the user needs multiple values, recommend multiple bound properties is NOT allowed for the primary value — use one bound property plus input properties, or reconsider as a Dataset Control.
- Prefer React Virtual Controls over standard (DOM-injected) controls: the framework owns the React root, eliminating a whole class of memory leaks.
- Do not design around direct DOM manipulation, jQuery, or global state.
- Keep the property list minimal — every manifest property is public API surface that is hard to change later.
- Do not run `pac pcf init` or write any code in this skill; that is `code-component-init`'s job.

## Example user prompts

- "I want to build a PCF control — help me think through the design first."
- "Spec out a dataset control that shows opportunities in a card grid."
- "Design a slider field control for a currency column."
- "We need a code component for the account view; what do you need to know?"
