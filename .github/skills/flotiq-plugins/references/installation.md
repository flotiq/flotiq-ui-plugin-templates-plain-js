# Installation and publishing

## Temporary installation (development)

1. Serve the built plugin JavaScript over HTTPS (a template's local dev server, or any HTTPS-capable static host).
2. Visit that URL directly first and confirm the response is JavaScript, not an HTML error page — accept any local development certificate the browser asks for.
3. In the authenticated Flotiq browser console, run:

   ```js
   FlotiqPlugins.loadPlugin('your-company.your-plugin', 'https://your-dev-url/index.js');
   ```

Temporary loading lasts until refresh/logout. Loading the same plugin ID again replaces the previous registration — useful for fast iteration, since there's no separate "unload" step.

## Permanent installation

For organization-wide installation, host both the built JavaScript and `plugin-manifest.json` at browser-accessible HTTPS URLs, set the manifest's `url` to the hosted JavaScript, and add the hosted manifest URL in Flotiq's plugin management. Localhost is unsuitable for other organization users.

Before publishing:

* Use a unique, stable ID, accurate metadata, a semantic version, the production JavaScript URL, and least-privilege permissions.
* Verify the production host's MIME type, HTTPS certificate, CORS policy, availability, and cache/version behavior — a stale cached bundle after a version bump is a common failure mode.
* Never publish credentials in either built artifact.
