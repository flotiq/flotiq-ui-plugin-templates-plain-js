---
name: flotiq-plugins
description: Use when building or extending a Flotiq UI plugin — choosing panel events, adding forms/settings, configuring manifest permissions and API access, or installing/publishing the plugin. Not for consuming Flotiq content in a frontend app (see flotiq-sdk) or rendering Flotiq page sections (see flotiq-ui-sections).
---

# Flotiq Plugins

Build UI plugins that extend the Flotiq panel: grid/form additions, custom renderers, settings screens, and API-backed actions. Use only documented event names and payload contracts, request least-privilege permissions, and keep credentials out of the bundle. Not for building an app that consumes Flotiq content (`flotiq-sdk`) or rendering CMS page sections in a frontend (`flotiq-ui-sections`).

## When to use

* Adding a button, renderer, or panel to a CTD grid or content-object form.
* Reacting to a Flotiq panel event (grid, form, settings, lifecycle).
* Adding a plugin settings screen, including credentials for third-party APIs.
* Calling the Flotiq API or a third-party API from a plugin.
* Installing a plugin locally for development, or preparing one for publishing.

## Workflow

1. Identify the target CTD, UI placement, inputs, and any destructive behavior before writing code; ask when placement or destructive behavior is ambiguous.
2. Pick the exact documented event for that placement (`references/events-and-ui.md`). Never invent a plausible-looking event name.
3. If the feature involves a form, modal, settings screen, or credentials, read `references/forms-modals-and-settings.md` first — several of its shapes (settings schema, credential handling) fail in non-obvious ways if skipped.
4. Plan data access: request only the permissions the implementation actually needs (`references/manifest.md`), then use the client's permission-checked methods (`references/api-access.md`).
5. Implement the handler: return synchronously from `::render`/`::add`, return `null` outside the feature's scope, and follow the DOM attach/detach lifecycle for cleanup.
6. Validate locally, then follow `references/installation.md` for temporary install and, if applicable, publishing.

## Best practices

* Use only documented event names and payload fields; check `events-and-ui.md` rather than guessing from a similar-looking event.
* Prefer the client's permission-checked methods over direct Flotiq REST calls; request least-privilege permissions per operation.
* Read credentials (including any fallback Flotiq key) from plugin settings at runtime — never from source, `.env`, the manifest, or build-time constants that end up in the browser bundle.
* Return the root element synchronously from `::render`/`::add`; do async work after, and update the already-returned element when it resolves.
* Return `null` when a handler doesn't apply to the current content type, field, or view, so other plugins and the default UI can still render.

## Common mistakes

* Inventing an event name or payload field that "sounds right" instead of checking the reference.
* Making `::render`/`::add` handlers `async`, which breaks Flotiq's expectation of a synchronous return.
* Storing a credential in source, `.env`, or the manifest instead of plugin settings — this is a frontend bundle; anything there ships to the browser.
* Building a settings schema as a minimal JSON Schema instead of a full Content Type Definition shape (a missing `allOf`/`metaDefinition.order` crashes the settings UI).
* Requesting wildcard (`'*'`) or broader permissions than the implemented behavior needs.
* Reaching for direct Flotiq REST calls before confirming the operation is genuinely missing from the client.

## Completion checklist

* [ ] Every event used is a documented one, with a synchronous return from `::render`/`::add`.
* [ ] Handlers return `null` for out-of-scope content types/fields/views.
* [ ] `plugin-manifest.json` describes the plugin, and permissions match exactly what the implementation calls.
* [ ] No credential appears in source, `.env`, the manifest, or a bundled constant.
* [ ] A settings schema, if any, has `allOf`, `metaDefinition.order`, and a `propertiesConfig` entry for every field in `order`.
* [ ] The plugin was smoke-tested locally per `references/installation.md`.

## References

* [`references/events-and-ui.md`](references/events-and-ui.md) — grid/form/settings events, render and lifecycle rules.
* [`references/forms-modals-and-settings.md`](references/forms-modals-and-settings.md) — FormApi, schema modals, plugin settings, credential handling.
* [`references/manifest.md`](references/manifest.md) — `plugin-manifest.json` fields and least-privilege permissions.
* [`references/api-access.md`](references/api-access.md) — the plugin API client, direct REST access, external APIs.
* [`references/installation.md`](references/installation.md) — temporary install and publishing a plugin.

## Official documentation

* [Plugin development and installation](https://flotiq.com/docs/panel/PluginsDevelopment/plugins/)
* [Plugin examples](https://flotiq.com/docs/panel/PluginsDevelopment/plugin-examples/)
* [Events API](https://flotiq.com/docs/panel/PluginsDevelopment/PluginDocs/5_Events/)
* [Flotiq globals](https://flotiq.com/docs/panel/PluginsDevelopment/PluginDocs/2_FlotiqGlobals/)
* [FormApi](https://flotiq.com/docs/panel/PluginsDevelopment/PluginDocs/6_FormApi/)
* [PluginInfo](https://flotiq.com/docs/panel/PluginsDevelopment/PluginDocs/4_FlotiqPluginInfo/)
* [Plugin API client](https://flotiq.com/docs/panel/PluginsDevelopment/PluginDocs/3_FlotiqPluginApiClient/)
