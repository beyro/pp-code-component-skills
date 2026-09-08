# PCF Property Types (`of-type`)

Reference for the `of-type` attribute on a `<property>` element in `ControlManifest.Input.xml`. Use this when interviewing the user about bound/input/output properties to confirm the exact manifest type string.

Official reference: https://learn.microsoft.com/en-us/power-apps/developer/component-framework/manifest-schema-reference/property#using-of-type

## Supported `of-type` values

| Value | Description | Available for |
|---|---|---|
| `SingleLine.Text` | Plain single-line text. | Model-driven and canvas apps |
| `SingleLine.TextArea` | Multi-line text, up to 4000 characters. | Model-driven and canvas apps |
| `SingleLine.Email` | Email-formatted string. | Model-driven and canvas apps |
| `SingleLine.Phone` | Phone-formatted string. | Model-driven and canvas apps |
| `SingleLine.URL` | Hyperlink text (auto-prepends `https://`; HTTP/HTTPS/FTP/FTPS/OneNote/TEL only). | Model-driven and canvas apps |
| `SingleLine.Ticker` | Ticker-formatted string. | Model-driven and canvas apps |
| `Multiple` | Multi-line text, up to 1,048,576 characters. | Model-driven and canvas apps |
| `Whole.None` | Plain integer. | Model-driven and canvas apps |
| `Decimal` | Up to 10 decimal points of precision. | Model-driven and canvas apps |
| `FP` | Floating point, up to 5 decimal points of precision. | Model-driven and canvas apps |
| `Currency` | Monetary value. | Model-driven and canvas apps |
| `DateAndTime.DateOnly` | Date, no time component. | Model-driven and canvas apps |
| `DateAndTime.DateAndTime` | Date and time. | Model-driven and canvas apps |
| `TwoOptions` | Boolean-like choice (0/1) with custom labels (e.g. Yes/No, On/Off). | Model-driven and canvas apps |
| `OptionSet` | Single-select choice column. | Model-driven and canvas apps |
| `MultiSelectOptionSet` | Multi-select choice column. | Model-driven and canvas apps |
| `Lookup.Simple` | Single reference to one specific table. All custom lookups use this type. If the manifest also declares a `data-set`, wrap `Lookup.Simple` properties in the `data-set` element too. | Model-driven apps only |
| `Enum` | Enumerated data type. | Model-driven and canvas apps |
| `Object` | Object data type. **Output properties only.** | Model-driven and canvas apps |

## Not supported (do not offer these)

`Lookup.Customer`, `Lookup.Owner`, `Lookup.PartyList`, `Lookup.Regarding`, `Status`, `Status Reason`, `Whole.Duration`, `Whole.Language`, `Whole.TimeZone`, and File columns are not currently supported as PCF property types.

## Usage attribute

Every property also needs a `usage`, one of: `bound` (host-editable, tied to a column), `input` (read-only config value), `output` (value the control returns via `notifyOutputChanged`/`getOutputs`). `Object`-typed properties may only be `output`.

## `of-type-group`

Instead of a single `of-type`, a property can reference a `type-group` (defined elsewhere in the manifest) via `of-type-group`, letting one property accept several compatible types (e.g. any numeric type). Model-driven apps only.
