# Installation and publishing

## Temporary installation (development)

1. Serve the built plugin JavaScript over HTTPS (a template's local dev server, or any HTTPS-capable static host).
2. Visit that URL directly first and confirm the response is JavaScript, not an HTML error page — accept any local development certificate the browser asks for. Skipping this is the most common cause of `loadPlugin` failing with "Error occurred while connecting to the server, please try again later." — Flotiq's fetch to the local dev URL is silently blocked by the browser's untrusted self-signed certificate, and Flotiq reports that as this generic connection error. Fix: open the plugin URL (and the manifest URL, if used) directly in a browser tab and accept the certificate warning, then retry.
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
