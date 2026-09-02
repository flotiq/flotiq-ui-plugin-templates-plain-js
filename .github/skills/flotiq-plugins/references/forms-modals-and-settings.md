# Forms, modals, settings, and credentials

## Globals

Flotiq supplies globals as the third registration callback argument (the second is the permission-checked `client`):

```js
registerFn(pluginInfo, (handler, client, globals) => {
  const { openModal, openSchemaModal, toast } = globals;
});
```

Relevant helpers:

* `openModal(config)` / `closeModal(id, result)` — plugin-managed HTML content.
* `openSchemaModal(config)` — a Flotiq schema-driven form.
* `getPluginSettings()` / `setPluginSettings(settings)` — serialized settings.
* `getApiUrl()`, `getSpaceId()`, `getLanguage()`, `navigate(url)`, `getFeatureFlag(name)`.
* `toast` for user feedback — confirm its current API before calling a method not covered in the plugin docs.

## FormApi

Form event payloads expose a limited form API as `data.form`. Don't reach into undocumented internal form state or use an undocumented `data.formApi` field.

* Read: `getValue(name)`, `getValues()`, `getError(name)`, `getErrors()`, `getDirtyFields()`.
* State: `dirty`, `isValid`, `isSubmitting`.
* Write: `setFieldValue(name, value)`, `setValues(values)`, `setFieldTouched(name, touched)`, `setTouched(map)`, `resetForm(values, options)`.
* Actions: `validateForm(cause)`, `submitForm()`, `rerenderForm()`.

Respect the payload's disabled/readonly state, and use `formUniqueKey` when form-specific caching is needed.

## Content Type field definitions

Content Type Definitions store custom field schemas in `contentType.schemaDefinition.allOf[1].properties`, not directly in `contentType.schemaDefinition.properties`. When a plugin inspects fields from a form event, read from `allOf[1]` together with `contentType.metaDefinition.propertiesConfig`.

## Schema modals

`openSchemaModal` takes a modal config whose `form.schema` follows the Content Type Definition shape: `schemaDefinition` (properties, required, `additionalProperties`) and `metaDefinition` (field order, `propertiesConfig` with labels/help text/input types), plus optional `initialData`, `labels`, and form `options`.

The returned promise resolves with submitted values or the configured result. A custom `onSubmit` must follow the current `ContentObjectSubmitFn` tuple contract, and a custom `onValidate` returns field-keyed errors or `null` — confirm the current tuple shape against the InternalTypes docs before implementing custom submission, since generated examples can evolve. Disable repeated submits while work is in progress, retain field-level validation errors, and keep the modal open when a recoverable error needs user correction.

## Plugin settings UI

Prefer `flotiq.plugins.manage::form-schema` for schema-driven settings; use `flotiq.plugins.manage::render` only when the schema form can't express the required UI.

The form-schema event returns `{ schema, options? }`, and `schema` must be a **complete Content Type Definition**, not a minimal JSON Schema — the settings renderer reads `schemaDefinition.allOf[1]` and `metaDefinition.order`, and omitting either can crash the management view (`Cannot read properties of undefined (reading '1')`). Every settings schema needs a stable `id`, `schemaDefinition.type`, an `allOf` tuple with `AbstractContentTypeSchemaDefinition` followed by the plugin's field schema, `required`, `additionalProperties`, `metaDefinition.order`, and a `propertiesConfig` entry for every field named in `order`. Start from a known-good shape and change only the field definitions:

```js
handler.on('flotiq.plugins.manage::form-schema', () => ({
  schema: {
    id: 'mycompany.my-plugin-settings',
    schemaDefinition: {
      type: 'object',
      allOf: [
        { $ref: '#/components/schemas/AbstractContentTypeSchemaDefinition' },
        { type: 'object', properties: { apiKey: { type: 'string' } } },
      ],
      required: ['apiKey'],
      additionalProperties: false,
    },
    metaDefinition: {
      order: ['apiKey'],
      propertiesConfig: {
        apiKey: { label: 'API key', unique: false, helpText: '', inputType: 'text', isPassword: true },
      },
    },
  },
}));
```

Before considering a settings implementation done, verify every `metaDefinition.order` item exists in both `schemaDefinition.allOf[1].properties` and `metaDefinition.propertiesConfig`, then open the plugin management view as a manual smoke test.

A custom management renderer (`flotiq.plugins.manage::render`) receives `updateSettings`, `reload`, `modalInstance`, current plugin data, and content types. React to `flotiq.plugin.settings::changed` when already-rendered UI must refresh after a settings update. Parse serialized settings defensively, provide defaults, and handle missing/malformed data without breaking plugin registration.

## Credential fields

Store every configurable credential — a fallback Flotiq REST key, OpenAI/Google Maps/analytics/import-source keys, anything third-party — in plugin settings. Never in source files, `.env`, manifest data, build scripts/arguments, generated assets, or constants; this is a frontend project that bundles build-time values into downloadable JavaScript.

* Render credentials with password semantics (`isPassword` / the current supported password-input configuration).
* Don't populate a management form with the full existing secret if it would be rendered or returned to the browser unnecessarily.
* Treat an empty, unchanged credential field as "preserve the current stored value," not "erase," unless the UI offers an explicit remove action.
* Never surface credentials in toast messages, validation errors, network error details, or console output.
* Validate that required credentials exist before starting an operation, and explain how to configure them.
* Keep credentials out of DOM attributes, cache keys, URLs, analytics, and telemetry.

## Security boundary

Plugin settings are available to plugin JavaScript running in the browser. The plugin docs define string get/set methods but don't promise server-side secret isolation or encryption from browser users — settings prevent accidental source-control and bundle inclusion, they don't turn a browser credential into a real secret.

* Use least-privilege, revocable, restricted credentials with quotas.
* Apply provider restrictions (allowed origins/APIs) where supported.
* Use a backend proxy when a credential grants broad access, incurs material cost, can't be origin-restricted, or protects sensitive data.
