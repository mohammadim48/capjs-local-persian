<div align="center">

# Cap Local Persian Pro

**A Persian CAPTCHA widget for the modern web, with locally hosted assets.**

[![npm version](https://img.shields.io/npm/v/capjs-local-persian-pro?style=flat-square&color=2563eb)](https://www.npmjs.com/package/capjs-local-persian-pro)
[![License](https://img.shields.io/badge/license-Apache--2.0-16a34a?style=flat-square)](./LICENSE)
[![JavaScript](https://img.shields.io/badge/JavaScript-Web%20Component-f7df1e?style=flat-square)](https://github.com/mohammadim48/capjs-local-persian)

Persian UI & RTL · SHA-256 Proof of Work · WebAssembly · Web Workers

[npm](https://www.npmjs.com/package/capjs-local-persian-pro) · [GitHub](https://github.com/mohammadim48/capjs-local-persian) · [Report an issue](https://github.com/mohammadim48/capjs-local-persian/issues)

</div>

---

## Overview

`capjs-local-persian-pro` is a fork of the [Cap](https://github.com/tiagozip/cap) client widget, adapted for Persian interfaces and local asset hosting. Instead of an image puzzle, the browser solves a SHA-256 proof-of-work challenge and submits the solution to your server to obtain a verification token.

The main build loads its JavaScript and WebAssembly solver assets from the package's local directory. You can serve these files from your own infrastructure without a runtime CDN dependency—useful for deployments with restricted access to external asset providers.

> This package provides the **CAPTCHA client**. A Cap-compatible API and server-side token validation are required for production use. Local asset hosting does not mean verification works without a server connection.

## Features

- **Persian interface and RTL layout:** Persian initial, verification, and success labels in the main build.
- **Locally hosted assets:** Bundled WASM files, with relative asset URLs in the main build.
- **Web Workers:** Challenge computation runs outside the main UI thread.
- **WebAssembly acceleration:** The main build includes a Web Crypto fallback solver.
- **Web Component integration:** Add a `<cap-widget>` element to your page.
- **Programmatic verification:** Obtain a token through `window.Cap` without displaying the widget.
- **Customizable appearance:** Configure labels, colors, fonts, dimensions, and worker count.
- **Events and types:** Progress, success, error, and reset events, plus TypeScript declarations.

## Installation

```bash
npm install capjs-local-persian-pro
```

Or use your preferred package manager:

```bash
pnpm add capjs-local-persian-pro
# or
yarn add capjs-local-persian-pro
```

## Quick start

### 1. Copy the assets into your public directory

For projects that serve a `public` directory at the site root:

```bash
mkdir -p public/vendor/cap/wasm
cp node_modules/capjs-local-persian-pro/cap.min.js public/vendor/cap/
cp node_modules/capjs-local-persian-pro/wasm/cap_wasm.min.js public/vendor/cap/wasm/
cp node_modules/capjs-local-persian-pro/wasm/cap_wasm_bg.wasm public/vendor/cap/wasm/
```

Keep this directory structure:

```text
public/
└── vendor/
    └── cap/
        ├── cap.min.js
        └── wasm/
            ├── cap_wasm.min.js
            └── cap_wasm_bg.wasm
```

The main build resolves the WASM assets relative to its own URL. Keeping this structure avoids manual asset configuration. If your application is served under a path such as `/app/`, adjust the script URL accordingly.

### 2. Add the widget to a form

```html
<!doctype html>
<html lang="fa" dir="rtl">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Contact form</title>
  </head>
  <body>
    <form id="contact-form" action="/contact" method="post">
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
  </body>
</html>
```

The widget creates a hidden input named `cap-token` by default and fills it with the verification token after solving the challenge. Your backend must validate this token before processing the submitted form. Disabling the button is a UI convenience; server-side validation enforces verification.

> **Module loading:** `cap.min.js` uses `import.meta.url` and must be loaded as an ES module. For the current release, the documented integration is to copy the assets and load them in the browser. Although the package metadata declares `type: commonjs`, its main file contains module syntax; do not assume it can be loaded directly with `require()`.

## Server API requirements

With `data-cap-api-endpoint="/api/cap/"`, the widget sends these requests:

| Endpoint | Method | Purpose |
| --- | --- | --- |
| `/api/cap/challenge` | `POST` | Obtain a challenge and challenge token |
| `/api/cap/redeem` | `POST` | Submit solutions and obtain a verification token |

The challenge response can contain an array of `[salt, target]` pairs:

```json
{
  "challenge": [["example-salt", "00"]],
  "token": "challenge-token"
}
```

The widget also supports the compact `challenge` format with `c`, `s`, and `d` fields. These values must be generated by a compatible server; the example above illustrates the response structure only.

Redeem request body:

```json
{
  "token": "challenge-token",
  "solutions": [12345]
}
```

Successful redeem response structure:

```json
{
  "success": true,
  "token": "verification-token",
  "expires": "<ISO-8601 expiration timestamp>"
}
```

`expires` must be a date JavaScript can parse, in the future and less than 24 hours away. The widget resets when the token expires. See the [upstream Cap project](https://github.com/tiagozip/cap) for server implementation and token validation. This package does not provide the server or the endpoint that validates tokens for your protected application requests.

## Programmatic verification

Obtain a token without displaying the widget:

```html
<button id="verify-button" type="button">Verify</button>

<script type="module">
  import "/vendor/cap/cap.min.js";

  const cap = new window.Cap({ apiEndpoint: "/api/cap/" });

  document.getElementById("verify-button").addEventListener("click", async () => {
    try {
      const result = await cap.solve();
      if (!result?.success) return;

      // Send this token with your application request for backend validation.
      console.log(result.token);
    } catch (error) {
      console.error("CAPTCHA verification failed:", error);
    }
  });
</script>
```

`window.Cap` creates a hidden widget. It requires the same server API and WASM assets as the visible widget.

## Widget configuration

| Attribute | Purpose | Default |
| --- | --- | --- |
| `data-cap-api-endpoint` | Base URL of the CAPTCHA API | Required unless using `CAP_CUSTOM_FETCH` |
| `data-cap-hidden-field-name` | Hidden token input name | `cap-token` |
| `data-cap-worker-count` | Number of solver workers | `navigator.hardwareConcurrency` or `8` |
| `data-cap-i18n-initial-state` | Initial label | `من ربات نیستم!` |
| `data-cap-i18n-verifying-label` | Verification label | `در حال بررسی...` |
| `data-cap-i18n-solved-label` | Success label | `تایید شد!` |
| `data-cap-i18n-error-label` | Error label | `Error. Try again.` |
| `data-cap-i18n-wasm-disabled` | Fallback solver hint | English message suggesting WASM activation |

Accessibility labels can also be customized:

```html
<cap-widget
  data-cap-api-endpoint="/api/cap/"
  data-cap-i18n-verify-aria-label="برای بررسی کلیک کنید"
  data-cap-i18n-verifying-aria-label="در حال بررسی؛ لطفاً منتظر بمانید"
  data-cap-i18n-verified-aria-label="بررسی موفق بود؛ می‌توانید ادامه دهید"
  data-cap-i18n-error-aria-label="خطایی رخ داد؛ دوباره تلاش کنید"
></cap-widget>
```

Choose a positive integer worker count no greater than `Math.min(navigator.hardwareConcurrency || 8, 16)`. Invalid values use the default worker count.

## Methods and events

| Widget API | Description |
| --- | --- |
| `await widget.solve()` | Solve a challenge; returns `{ success: true, token }` on success |
| `widget.reset()` | Clear the token and restore the initial state |
| `widget.setWorkersCount(2)` | Configure the worker count |
| `widget.token` / `widget.tokenValue` | Current token or `null` |

Calling `solve()` while a solve is already running returns no result. Solver errors reject the Promise; use `try/catch` for programmatic verification.

| Event | `event.detail` |
| --- | --- |
| `progress` | `{ progress: number }`, expressed as a percentage |
| `solve` | `{ token: string }` |
| `error` | `{ isCap: true, message: string }` |
| `reset` | `{}` |

```js
widget.addEventListener("progress", ({ detail }) => {
  console.log(`Progress: ${detail.progress}%`);
});

widget.addEventListener("solve", ({ detail }) => {
  console.log("Verification token:", detail.token);
});

widget.addEventListener("error", ({ detail }) => {
  console.error(detail.message);
});
```

Read the token from `event.detail.token` inside a `solve` callback: the event fires before `widget.token` is updated. The `window.Cap` class also exposes `solve()`, `reset()`, `addEventListener()`, `token`, and the underlying widget through `cap.widget`.

## Styling

Customize the widget through CSS variables inherited by its Shadow DOM:

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

Load the example font separately in your application. The `checkbox`, `label`, and `attribution` parts can be targeted with `::part()`. Additional variables include `--cap-checkbox-size`, `--cap-checkbox-border`, and `--cap-spinner-thickness`.

## Custom WASM location and fetch behavior

Set global configuration **before executing the widget script**:

```html
<script type="module">
  window.CAP_CUSTOM_WASM_URL = new URL(
    "/assets/cap/wasm/cap_wasm.min.js",
    window.location.origin
  ).href;

  await import("/vendor/cap/cap.min.js");
</script>
```

`CAP_CUSTOM_WASM_URL` must point to the **JavaScript WASM loader**, not the `.wasm` binary. Keep `cap_wasm_bg.wasm` beside that loader. An absolute URL makes the worker's module import location explicit.

To customize API requests:

```js
window.CAP_CUSTOM_FETCH = (url, options) =>
  fetch(url, { ...options, credentials: "include" });

await import("/vendor/cap/cap.min.js");
```

This hook handles challenge and redeem requests; internal WASM loading does not use it. You can also set `window.CAP_CSS_NONCE` to apply a nonce to the widget's internal style element.

## Package files

| File | Purpose |
| --- | --- |
| `cap.min.js` | Main Persian/RTL build with relative WASM URLs; load as a module |
| `wasm/cap_wasm.min.js` | WASM solver loader module |
| `wasm/cap_wasm_bg.wasm` | WebAssembly binary |
| `cap.d.ts` | TypeScript declarations |
| `cap-floating.min.js` | Floating widget extension |
| `cap.compat.min.js` | Separate build with different behavior; references a CDN by default |
| `src/` | Source files; their behavior differs from the main build in some details |

Use `cap.min.js` and the `wasm/` directory for the documented local deployment. Treat `cap.compat.min.js` as a separate build rather than an equivalent replacement for the main build.

## Troubleshooting

| Problem | What to check |
| --- | --- |
| `import.meta` syntax error | Load the main file with `type="module"` or browser `import()` |
| WASM assets return 404 | Check the directory structure and public URLs of both WASM assets |
| Module rejected due to MIME type | Serve JS with an appropriate JavaScript MIME type and WASM with `application/wasm` |
| `Missing API endpoint` | Configure the widget attribute or constructor's `apiEndpoint` |
| `Invalid solution` | Check server compatibility, the challenge response, and the redeem result |
| `Invalid expiration time` | Check the server clock and the future `expires` value in the redeem response |
| Slow challenge solving | Inspect Network and Console for WASM loading failures and fallback activation |
| CSP blocks workers | Allow `blob:` workers in `worker-src`; also configure the application's policy for JS, WASM, API requests, and internal styles |

Use HTTPS or localhost; the fallback solver requires Web Crypto. Serve the assets through a web server rather than opening them with `file://`. The widget runs in the browser and must be loaded on the client in applications with server-side rendering.

## Contributing and support

Found a bug or have a feature suggestion? [Open an issue](https://github.com/mohammadim48/capjs-local-persian/issues) with the package version, browser, reproduction steps, and error message. Remove tokens and sensitive information from your report.

## License and credits

Released under the [Apache-2.0 license](./LICENSE).

Based on [Cap](https://github.com/tiagozip/cap), created by **Tiago**. This fork is maintained by [Mohammadim48](https://github.com/mohammadim48). The original copyright and license are retained in the LICENSE file.
