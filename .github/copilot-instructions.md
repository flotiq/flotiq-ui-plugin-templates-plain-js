# Flotiq Plain JavaScript Plugin Guidelines

## Repository Architecture

- Keep `plugins/index.js` as the bundle entry point and register the plugin through `registerFn` from `common/plugin-element-cache.js`.
- Put feature handlers in focused modules under `plugins/`; keep registration, global CSS injection, and handler wiring in `plugins/index.js`.
- Import plugin metadata from `plugin-manifest.json`. Keep its identity, version, URL, and permissions synchronized with implemented behavior.
- Preserve the existing plain JavaScript, ES modules, ESLint, Prettier, and esbuild conventions. Do not introduce React or TypeScript unless requested. If React requested, suggest the `flotiq/flotiq-ui-plugin-templates-react` plugin template.
- Load the `flotiq-plugins` skill for plugin events, UI placement, forms, settings, permissions, API access, or local development. This file only covers what's specific to the plain-JS boilerplate.

## Removing Template Demo Behavior

Before implementing a real feature, choose one path:

- **Production plugin**: remove the template demo before adding the real feature.
- **Sample extension**: keep only the demo behavior that directly illustrates the request.

Cleanup checklist:

1. Remove the `handleGridPlugin` import and the `flotiq.grid.cell::render` registration from `plugins/index.js`.
2. Delete `plugins/grid-renderers/index.js` after confirming no remaining module imports it. This removes random text colors, bold number rendering, and relation-title rendering from grid cells.
3. Remove the demo `.plugin-name-cell-renderer` rule, `@font-face`, and `plugins/styles/RobotoMono-Medium.ttf` when the new plugin does not use them. Keep `style.css` and its one-time injection when the new feature needs plugin CSS; otherwise remove the unused CSS import/injection too.
4. Remove `common/api-helpers.js` only when `getRelationData` has no remaining callers. Preserve `common/plugin-element-cache.js` when the new feature uses registration or element caching.
5. Replace template metadata in `plugin-manifest.json` and `package.json`: plugin ID/name, description, version, repository, production URL as applicable, and permissions. Remove the wildcard sample read permission unless the real feature needs it.
6. Search for stale demo identifiers and descriptions such as `grid-renderers`, `handleGridPlugin`, `plugin-name-cell-renderer`, `Plain JS Plugin Template`, and "colorful text."
7. Run `yarn build` after cleanup so deleted imports/assets and stale references fail immediately.

If the user asks for another reusable example rather than a production plugin, keep only the sample behavior that directly demonstrates the requested concept and update its metadata accordingly.

## API Access And Credentials

- Prefer the permission-checked `apiClient[ctdName]` methods for every Flotiq API operation and request only the required `CO` or `CTD` permissions in `plugin-manifest.json`.
- Direct Flotiq REST access is a project-specific exception because official UI-plugin guidance disallows own API keys and direct Flotiq API calls. Use it only after confirming the required operation is absent from `apiClient`, documenting the gap, and obtaining explicit project-owner acceptance.
- Read any fallback Flotiq REST key and all third-party credentials, such as OpenAI or Google Maps keys, from plugin settings at runtime. Never place credentials in source code, `.env`, the manifest, build arguments, or constants that become part of the browser bundle.

## Build And Validation

- Install dependencies with `yarn install`.
- Run `yarn build` after changes. It bundles `plugins/index.js` to `dist/index.js` and copies the manifest to `dist/plugin-manifest.json`.
- `yarn build` runs ESLint as an esbuild plugin, including its configuration (`.eslintrc.cjs`), and there is no separate `yarn lint` script — a lint failure only surfaces after the full build runs. Run `yarn format` scoped to just the files you edited (e.g. `npx prettier --write plugins/my-file.js`) before `yarn build` to catch style errors without a wasted full build cycle.
- Run `yarn start` for watch mode and the local HTTPS endpoint at `https://localhost:3053`.
- Run `yarn format` only when formatting is needed; avoid unrelated formatting churn — prefer scoping it to changed files over the whole repo.

## Release Hygiene

- This project keeps a `CHANGELOG.md` (Keep a Changelog format, Semantic Versioning).
- After making a change, bump `version` in both `package.json` and `plugin-manifest.json`, and add a corresponding entry to `CHANGELOG.md`. CI checks both files independently against `main`.
