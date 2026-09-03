# Manifest and permissions

## Manifest

`plugin-manifest.json` describes the plugin and is typically also bundled/copied by the build. Keep these fields aligned with the implementation:

* `id` — globally unique, stable; prefix with the organization/company name.
* `name` — user-facing name.
* `version` — semantic version used to identify updates.
* `url` — full URL of the built JavaScript file.
* `description`, `repository` — optional metadata that should stay accurate.
* `permissions` — only the API capabilities the current implementation needs.

The manifest has no event list — event handlers are registered in code.

## Permissions

Permission entries use `type: 'CO' | 'CTD'`, a `ctdName` (`'*'` means all), and operation flags. Use the API reference names as canonical: `canRead`, `canCreate`, `canUpdate`, `canDelete`. (One getting-started example also shows `canWrite` — don't infer its behavior; verify current support before using a field absent from the `PluginPermission` API reference.)

Examples of least privilege:

* Create imported `product` objects: `{ "type": "CO", "ctdName": "product", "canCreate": true }`.
* Also read `product` objects before deduplicating: add `canRead: true` to that entry.
* Read the `product` schema: a separate `{ "type": "CTD", "ctdName": "product", "canRead": true }` entry.

Don't request wildcard access merely because it's convenient during development.
