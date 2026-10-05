<div align="center">

# Cap Local Persian

**A Persian CAPTCHA widget with RTL support and locally hosted solver assets.**

[![npm version](https://img.shields.io/npm/v/capjs-local-persian?style=flat-square&color=2563eb)](https://www.npmjs.com/package/capjs-local-persian)
[![License](https://img.shields.io/badge/license-Apache--2.0-16a34a?style=flat-square)](./LICENSE)

Persian UI · RTL layout · SHA-256 Proof of Work · WebAssembly · Web Workers

[npm](https://www.npmjs.com/package/capjs-local-persian) · [GitHub](https://github.com/mohammadim48/capjs-local-persian) · [Report an issue](https://github.com/mohammadim48/capjs-local-persian/issues)

</div>

---

## Overview

`capjs-local-persian` is a fork of the [Cap](https://github.com/tiagozip/cap) client widget, adapted for Persian interfaces and local asset hosting. Instead of an image puzzle, the browser solves a SHA-256 proof-of-work challenge and submits the solution to your server to obtain a verification token.

The main build loads the bundled WebAssembly solver relative to its own URL. Host these assets on your infrastructure without a runtime CDN dependency—useful for networks with limited access to external asset providers.

> This package is the **CAPTCHA client**. Production use requires a Cap-compatible server API and backend token validation. Local asset hosting does not mean verification works without a server connection.

## Features

- **Persian and RTL:** Persian initial, verification, and success labels, with right-to-left layout in the main build.
- **Local assets:** WASM files ship with the package.
- **Web Workers:** Computation runs outside the main UI thread.
- **WebAssembly:** Accelerated solving, with a Web Crypto fallback in the main build.
- **Web Component:** Integrate with the `<cap-widget>` element.
- **Programmatic verification:** Solve challenges through `window.Cap` without displaying the widget.
- **Customization:** Configure text, colors, fonts, dimensions, and worker count.
- **Events and types:** Progress, solve, error, and reset events, plus TypeScript declarations.

## Installation

```bash
npm install capjs-local-persian
```

```bash
pnpm add capjs-local-persian
# or
yarn add capjs-local-persian
```

## Quick start

### 1. Copy the public assets

For applications that serve their `public` directory at the site root:

```bash
mkdir -p public/vendor/cap/wasm
cp node_modules/capjs-local-persian/cap.min.js public/vendor/cap/
cp node_modules/capjs-local-persian/wasm/cap_wasm.min.js public/vendor/cap/wasm/
cp node_modules/capjs-local-persian/wasm/cap_wasm_bg.wasm public/vendor/cap/wasm/
```

Keep this structure so the main build can resolve its solver assets:

```text
public/vendor/cap/
├── cap.min.js
└── wasm/
    ├── cap_wasm.min.js
    └── cap_wasm_bg.wasm
```

Adjust the public URLs if your application uses a base path such as `/app/`.

### 2. Add the widget

```html
<form action="/contact" method="post" dir="rtl">
  <label>
    پیام شما
    <textarea name="message" required></textarea>
  </label>

  <cap-widget
    id="captcha"
    data-cap-api-endpoint="/api/cap/"
    data-cap-i18n-error-label="خطا؛ دوباره تلاش کنید"
  ></cap-widget>

  <button id="submit-button" type="submit" disabled>ارسال پیام</button>
</form>

<script type="module">
  import "/vendor/cap/cap.min.js";

  const widget = document.getElementById("captcha");
  const button = document.getElementById("submit-button");

  widget.addEventListener("solve", () => {
    button.disabled = false;
  });
  widget.addEventListener("reset", () => {
    button.disabled = true;
  });
  widget.addEventListener("error", () => {
    button.disabled = true;
  });
</script>
```

The widget creates a hidden input named `cap-token` and populates it after solving. Your backend must validate this token before processing the form. Disabling the button is only a UI convenience.

> **Load as a module:** `cap.min.js` uses `import.meta.url`. Use `type="module"` or browser `import()`. The package currently declares `type: commonjs`, while its main file contains module syntax; do not assume direct `require()` usage works. Copying and serving browser assets is the documented integration for this release.

## Server API

For `data-cap-api-endpoint="/api/cap/"`:

| Endpoint | Method | Purpose |
| --- | --- | --- |
| `/api/cap/challenge` | `POST` | Obtain a challenge and challenge token |
| `/api/cap/redeem` | `POST` | Submit solutions and receive a verification token |

Example challenge response structure:

```json
{
  "challenge": [["example-salt", "00"]],
  "token": "challenge-token"
}
```

The widget also supports the compact challenge format with `c`, `s`, and `d` fields. A compatible server must generate the challenge; the values above are illustrative.

Redeem request body:

```json
{
  "token": "challenge-token",
  "solutions": [12345]
}
```

Successful response structure:

```json
{
  "success": true,
  "token": "verification-token",
  "expires": "<ISO-8601 expiration timestamp>"
}
```

`expires` must be a parseable future date less than 24 hours away. The widget resets when it expires. See [upstream Cap](https://github.com/tiagozip/cap) for server implementation and token validation. This package does not provide a backend or your application's final token-validation endpoint.

## Programmatic verification

```html
<button id="verify-button" type="button">Verify</button>

<script type="module">
  import "/vendor/cap/cap.min.js";

  const cap = new window.Cap({ apiEndpoint: "/api/cap/" });

  document.getElementById("verify-button").addEventListener("click", async () => {
    try {
      const result = await cap.solve();
      if (!result?.success) return;
      // Send the token with your request for server-side validation.
      console.log(result.token);
    } catch (error) {
      console.error("Verification failed:", error);
    }
  });
</script>
```

The constructor creates a hidden widget and requires the same API and solver assets.

## Configuration

| Attribute | Purpose | Default |
| --- | --- | --- |
| `data-cap-api-endpoint` | API base URL | Required unless using `CAP_CUSTOM_FETCH` |
| `data-cap-hidden-field-name` | Hidden input name | `cap-token` |
| `data-cap-worker-count` | Solver worker count | `navigator.hardwareConcurrency` or `8` |
| `data-cap-i18n-initial-state` | Initial text | `من ربات نیستم!` |
| `data-cap-i18n-verifying-label` | Verification text | `در حال بررسی...` |
| `data-cap-i18n-solved-label` | Success text | `تایید شد!` |
| `data-cap-i18n-error-label` | Error text | `Error. Try again.` |
| `data-cap-i18n-wasm-disabled` | Fallback hint | English WASM activation hint |

Accessibility text can be configured with `data-cap-i18n-verify-aria-label`, `data-cap-i18n-verifying-aria-label`, `data-cap-i18n-verified-aria-label`, and `data-cap-i18n-error-aria-label`.

Use a positive integer worker count no greater than `Math.min(navigator.hardwareConcurrency || 8, 16)`. Invalid values use the default count.

## Methods and events

| Widget API | Description |
| --- | --- |
| `await widget.solve()` | Returns `{ success: true, token }` on success |
| `widget.reset()` | Clear the token and restore the initial state |
| `widget.setWorkersCount(2)` | Configure worker count |
| `widget.token` / `widget.tokenValue` | Current token or `null` |

Errors reject the solve Promise. Calling `solve()` while a solve is running returns no result.

| Event | `event.detail` |
| --- | --- |
| `progress` | `{ progress: number }`, as a percentage |
| `solve` | `{ token: string }` |
| `error` | `{ isCap: true, message: string }` |
| `reset` | `{}` |

```js
widget.addEventListener("progress", ({ detail }) => {
  console.log(`Progress: ${detail.progress}%`);
});
widget.addEventListener("solve", ({ detail }) => {
  console.log("Token:", detail.token);
});
widget.addEventListener("error", ({ detail }) => {
  console.error(detail.message);
});
```

Inside the `solve` listener, read `event.detail.token`: the event fires before `widget.token` is updated. `window.Cap` exposes `solve()`, `reset()`, `addEventListener()`, `token`, and the underlying widget as `cap.widget`.

## Styling

```css
cap-widget {
  --cap-font: "Vazirmatn", sans-serif;
  --cap-background: #ffffff;
  --cap-color: #0f172a;
  --cap-border-color: #cbd5e1;
  --cap-border-radius: 16px;
  --cap-widget-width: 280px;
  --cap-widget-height: 64px;
  --cap-widget-padding: 16px;
  --cap-gap: 14px;
  --cap-spinner-color: #2563eb;
  --cap-spinner-background-color: #dbeafe;
}

cap-widget::part(label) {
  font-size: 14px;
}
```

Load the example font separately. Available parts include `checkbox`, `label`, and `attribution`. Additional CSS variables include `--cap-checkbox-size`, `--cap-checkbox-border`, and `--cap-spinner-thickness`.

## Custom asset URLs and fetch

Set globals before executing the widget script:

```html
<script type="module">
  window.CAP_CUSTOM_WASM_URL = new URL(
    "/assets/cap/wasm/cap_wasm.min.js",
    window.location.origin
  ).href;

  await import("/vendor/cap/cap.min.js");
</script>
```

Point to the JavaScript WASM loader, not the binary, and keep `cap_wasm_bg.wasm` beside it. Prefer an absolute URL for worker module imports.

```js
window.CAP_CUSTOM_FETCH = (url, options) =>
  fetch(url, { ...options, credentials: "include" });

await import("/vendor/cap/cap.min.js");
```

The fetch hook handles API requests, not internal WASM loading. Set `window.CAP_CSS_NONCE` before loading to add a nonce to the internal style element.

## Package files

| File | Purpose |
| --- | --- |
| `cap.min.js` | Main Persian/RTL browser build with relative WASM URLs |
| `wasm/cap_wasm.min.js` | WASM loader module |
| `wasm/cap_wasm_bg.wasm` | WebAssembly binary |
| `cap.d.ts` | TypeScript declarations |
| `cap-floating.min.js` | Floating widget extension |
| `cap.compat.min.js` | Separate build; references a CDN by default |
| `src/` | Source files; behavior differs from the main build in some details |

Use the main build and `wasm/` directory for local deployment. The compatibility build is not an equivalent replacement.

## Troubleshooting

| Problem | Check |
| --- | --- |
| `import.meta` syntax error | Load the main script as a browser module |
| WASM 404 | Confirm the public directory structure and asset URLs |
| MIME type error | Serve JS as JavaScript and WASM as `application/wasm` |
| `Missing API endpoint` | Set the widget endpoint or constructor's `apiEndpoint` |
| `Invalid solution` | Check server compatibility and API responses |
| `Invalid expiration time` | Check the server clock and future `expires` value |
| Slow solving | Check Console and Network for WASM failures and fallback activation |
| CSP blocks workers | Allow `blob:` in `worker-src` and configure JS, WASM, API, and style policies for your deployment |

Use HTTPS or localhost; the fallback requires Web Crypto. Serve files through a web server rather than `file://`. In SSR applications, load the widget on the client.

## Support and contributing

[Open an issue](https://github.com/mohammadim48/capjs-local-persian/issues) with the package version, browser, reproduction steps, and error message. Remove tokens and sensitive data from reports.

## License and credits

Licensed under [Apache-2.0](./LICENSE).

Based on [Cap](https://github.com/tiagozip/cap), created by **Tiago**. This fork is maintained by [Mohammadim48](https://github.com/mohammadim48). The original copyright and license are retained.
