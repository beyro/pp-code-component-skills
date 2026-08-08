# Design Spec: PriorityBadge

## Overview
A field control that displays a record's priority as a colored badge and lets the user change it via a dropdown.

## Component Type
Field Control — React Virtual Control (`ComponentFramework.ReactControl`), Fluent UI v9.

## Properties
| Name | Type (manifest `of-type`) | Usage (bound/input/output) | Required | Description |
|------|---------------------------|-----------------------------|----------|-------------|
| value | OptionSet | bound | true | The priority option set value (Low/Medium/High) |

## UI Specification
- Renders a Fluent UI `Badge` colored by priority (Low = informative, Medium = warning, High = danger)
- Clicking the badge opens a Fluent UI `Dropdown` to change the value
- Disabled state: badge only, no dropdown
- Theming: use `context.fluentDesignLanguage` when available, else `webLightTheme`

## Events & Outputs
- Selecting a new option in the dropdown calls `notifyOutputChanged`; `getOutputs` returns `{ value: <new option value> }`

## Test Plan
- Renders the badge with the color matching `context.parameters.value.raw`
- Selecting a dropdown option calls `notifyOutputChanged` and `getOutputs` returns the new value
- Disabled state renders the badge without an interactive dropdown
- `destroy` cleans up any listeners

## Assumptions
- Namespace: `Contoso`
- Control name: `PriorityBadge`
- Option set values: 1=Low, 2=Medium, 3=High (labels only; no live choice metadata needed for scaffolding)
