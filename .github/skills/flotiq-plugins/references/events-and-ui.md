# Events and UI placement

Use exact documented event names — never invent a plausible-looking one. Full payload field lists live in the [Events API docs](https://flotiq.com/docs/panel/PluginsDevelopment/PluginDocs/5_Events/); this page covers which event to reach for and what trips people up, not the field list.

## Grid events

| Event | Use it for |
|---|---|
| `flotiq.grid::add` | Additive controls above a CTD grid (e.g. an import button). Prefer this over replacing the grid. |
| `flotiq.grid::render` | Replacing the entire default grid. Only when additive UI can't do the job. |
| `flotiq.grid.cell::render` | Replacing one field's cell rendering. |
| `flotiq.grid.filter::render` | Replacing a column's filter control. |

Call the payload's `reload()` after any action that changes grid data — nothing else refreshes it.

## Form events

| Event | Use it for |
|---|---|
| `flotiq.form::add` | An element at the start of a content-object form. |
| `flotiq.form.sidebar-panel::add` | An object-level command in the form sidebar. |
| `flotiq.form.secondary-column::add` | A side-by-side work area; returning content hides the default left nav and right sidebar. |
| `flotiq.form.field::render` | Replacing a visible field control. |
| `flotiq.form.field::config` | Mutating field config in place — unlike the `::render` events, no return value is expected. |
| `flotiq.form.field.listeners::add` | Adding an `onChange`/`onBlur`/`onMount`/`onSubmit` listener; return `[eventName, listener]`. |
| `flotiq.form::after-submit` | Observing the API-submission result; skipped on client-side validation failure. |
| `flotiq.form.relation::after-submit` | Observing edits to a related object made through the main form. |

Form events expose their `FormApi` instance as `data.form`, not `data.formApi`. For example, a `flotiq.form.sidebar-panel::add` handler should read `data.form.getValues()` and call `data.form.setFieldValue(name, value)`.

Use the payload's `formUniqueKey` in any cache key — multiple forms (e.g. version comparison) can be on screen at once. Respect `disabled`, `readonly`, `create`, and `duplicate` rather than offering an action that can't complete.

## Plugin and settings events

| Event | Use it for |
|---|---|
| `flotiq.plugins.manage::form-schema` | A schema-driven settings form (see `forms-modals-and-settings.md`). |
| `flotiq.plugins.manage::render` | A fully custom settings UI, only when the schema form can't express it. |
| `flotiq.plugin.settings::changed` | Refreshing already-rendered UI after a settings change. |
| `flotiq.plugin::removed` | Cleanup when the current plugin is removed. |
| `flotiq.language::changed` | Updating localized plugin UI (`pl`/`en`). |

## Render and lifecycle rules

* `::render`/`::add` handlers must not be `async`. Return the root element (or `null`) immediately; do async work after and update that same element when it resolves.
* Return `null` when the current CTD, accessor, field, mode, or state is outside the feature's scope, so Flotiq and other plugins keep rendering normally.
* Reuse DOM elements across repeated renders where possible instead of replacing them — it avoids UI churn. Flotiq dispatches `flotiq.attached`/`flotiq.detached` DOM events only on the root element a plugin returns, and the same root can attach/detach repeatedly (e.g. brief detach/reattach cycles) — start DOM-dependent work on attach, clean up observers/timers/subscriptions on detach, and don't treat a detach as necessarily final.
* Inside an interactive grid cell, stop propagation and prevent default navigation on a button that shouldn't open the row.
