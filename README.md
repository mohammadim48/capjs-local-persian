<div align="center">

# Cap Local Persian Pro

**کپچای فارسی برای وب مدرن؛ با میزبانی محلی و بدون نیاز به CDN در نسخهٔ اصلی.**

[![npm version](https://img.shields.io/npm/v/capjs-local-persian-pro?style=flat-square&color=2563eb)](https://www.npmjs.com/package/capjs-local-persian-pro)
[![License](https://img.shields.io/badge/license-Apache--2.0-16a34a?style=flat-square)](./LICENSE)
[![JavaScript](https://img.shields.io/badge/JavaScript-Web%20Component-f7df1e?style=flat-square)](https://github.com/mohammadim48/capjs-local-persian)

رابط فارسی و RTL · SHA-256 Proof of Work · WebAssembly · Web Workers

[npm](https://www.npmjs.com/package/capjs-local-persian-pro) · [GitHub](https://github.com/mohammadim48/capjs-local-persian) · [گزارش مشکل](https://github.com/mohammadim48/capjs-local-persian/issues)

</div>

---

## معرفی

`capjs-local-persian-pro` یک فورک از ویجت [Cap](https://github.com/tiagozip/cap) است که برای رابط‌های فارسی و استقرار محلی آماده شده است. به‌جای معمای تصویری، مرورگر یک چالش محاسباتی مبتنی بر SHA-256 را حل می‌کند و پاسخ را برای دریافت توکن به سرور می‌فرستد.

نسخهٔ اصلی، فایل‌های JavaScript و WebAssembly موردنیاز حل چالش را از مسیر محلی پکیج بارگذاری می‌کند؛ بنابراین می‌توانید دارایی‌های کلاینت را روی زیرساخت خودتان میزبانی کنید. این ویژگی برای شبکه‌هایی با دسترسی محدود به CDN مفید است.

> این پکیج **کلاینت کپچا** است. برای استفادهٔ واقعی، به API سازگار با Cap و اعتبارسنجی توکن در بک‌اند نیاز دارید. میزبانی محلی دارایی‌ها به معنی اجرای کپچا بدون ارتباط با سرور نیست.

## امکانات

- **رابط فارسی و راست‌به‌چپ:** متن فارسی برای حالت اولیه، بررسی و تأیید، به‌همراه چیدمان RTL در فایل اصلی.
- **دارایی‌های محلی:** فایل‌های WASM همراه پکیج هستند و نسخهٔ اصلی برای بارگذاری آن‌ها به CDN نیاز ندارد.
- **حل چالش در Web Workers:** محاسبات خارج از رشتهٔ اصلی رابط کاربری انجام می‌شود.
- **شتاب‌دهی با WebAssembly:** همراه با حل‌کنندهٔ جایگزین مبتنی بر Web Crypto در فایل اصلی.
- **Web Component:** قابل استفاده با تگ `<cap-widget>` در صفحات وب.
- **اجرای برنامه‌ای:** دریافت توکن با `window.Cap`، بدون نمایش ویجت.
- **قابل شخصی‌سازی:** متن‌ها، رنگ‌ها، فونت، اندازه‌ها و تعداد Workerها.
- **رویدادهای کاربردی:** پیشرفت، موفقیت، خطا و بازنشانی؛ همراه با فایل تعریف نوع TypeScript.

## نصب

```bash
npm install capjs-local-persian-pro
```

یا با مدیر بستهٔ دلخواه:

```bash
pnpm add capjs-local-persian-pro
# یا
yarn add capjs-local-persian-pro
```

## شروع سریع

### ۱. فایل‌ها را در پوشهٔ عمومی پروژه قرار دهید

برای پروژه‌هایی که پوشهٔ `public` را در ریشهٔ سایت سرو می‌کنند:

```bash
mkdir -p public/vendor/cap/wasm
cp node_modules/capjs-local-persian-pro/cap.min.js public/vendor/cap/
cp node_modules/capjs-local-persian-pro/wasm/cap_wasm.min.js public/vendor/cap/wasm/
cp node_modules/capjs-local-persian-pro/wasm/cap_wasm_bg.wasm public/vendor/cap/wasm/
```

ساختار نهایی:

```text
public/
└── vendor/
    └── cap/
        ├── cap.min.js
        └── wasm/
            ├── cap_wasm.min.js
            └── cap_wasm_bg.wasm
```

فایل اصلی مسیر WASM را نسبت به URL خودش پیدا می‌کند. با حفظ این ساختار، نیازی به تنظیم دستی مسیر ندارید. اگر برنامه زیر یک مسیر مانند `/app/` سرو می‌شود، URL اسکریپت را متناسب با همان مسیر تغییر دهید.

### ۲. ویجت را به فرم اضافه کنید

```html
<!doctype html>
<html lang="fa" dir="rtl">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>فرم تماس</title>
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

ویجت یک فیلد مخفی با نام پیش‌فرض `cap-token` ایجاد می‌کند و بعد از حل چالش، توکن را داخل آن می‌گذارد. هنگام ارسال فرم، بک‌اند باید این توکن را اعتبارسنجی کند و فقط پس از موفقیت، درخواست را پردازش کند. غیرفعال‌کردن دکمه صرفاً رفتار رابط کاربری است.

> فایل `cap.min.js` از `import.meta.url` استفاده می‌کند؛ آن را به‌صورت ES module بارگذاری کنید. برای نسخهٔ فعلی، روش مستندشده کپی دارایی‌ها و بارگذاری در مرورگر است؛ ورودی پکیج با وجود `type: commonjs` یک فایل دارای نحو ماژول است و نباید اجرای مستقیم آن با `require()` را فرض کرد.

## API موردنیاز سرور

اگر `data-cap-api-endpoint` برابر `/api/cap/` باشد، ویجت درخواست‌های زیر را می‌فرستد:

| مسیر | متد | کاربرد |
| --- | --- | --- |
| `/api/cap/challenge` | `POST` | دریافت چالش و توکن چالش |
| `/api/cap/redeem` | `POST` | ارسال پاسخ‌ها و دریافت توکن نهایی |

پاسخ چالش می‌تواند شامل آرایه‌ای از زوج‌های `[salt, target]` باشد:

```json
{
  "challenge": [["example-salt", "00"]],
  "token": "challenge-token"
}
```

ویجت قالب فشردهٔ `challenge` با فیلدهای `c`، `s` و `d` را هم پشتیبانی می‌کند. این داده‌ها باید توسط سرور سازگار تولید شوند؛ نمونهٔ بالا صرفاً ساختار پاسخ را نشان می‌دهد.

بدنهٔ درخواست redeem:

```json
{
  "token": "challenge-token",
  "solutions": [12345]
}
```

ساختار پاسخ موفق:

```json
{
  "success": true,
  "token": "verification-token",
  "expires": "<ISO-8601 expiration timestamp>"
}
```

`expires` باید یک تاریخ قابل‌تفسیر در JavaScript، در آینده و با فاصلهٔ کمتر از ۲۴ ساعت باشد. ویجت در زمان انقضا بازنشانی می‌شود. برای پیاده‌سازی سرویس و اعتبارسنجی توکن، به [پروژهٔ اصلی Cap](https://github.com/tiagozip/cap) مراجعه کنید. این پکیج سرور یا endpoint اعتبارسنجی درخواست نهایی را ارائه نمی‌کند.

## اجرای برنامه‌ای بدون نمایش ویجت

```html
<script type="module">
  import "/vendor/cap/cap.min.js";

  const cap = new window.Cap({ apiEndpoint: "/api/cap/" });

  document.getElementById("verify-button").addEventListener("click", async () => {
    try {
      const result = await cap.solve();
      if (!result?.success) return;

      // توکن را همراه درخواست به بک‌اند بفرستید تا اعتبارسنجی شود.
      console.log(result.token);
    } catch (error) {
      console.error("بررسی کپچا ناموفق بود:", error);
    }
  });
</script>
```

در این مثال باید دکمه‌ای با شناسهٔ `verify-button` در صفحه وجود داشته باشد. `window.Cap` یک ویجت مخفی ایجاد می‌کند و به همان API و دارایی‌های WASM نیاز دارد.

## تنظیمات ویجت

| ویژگی | کاربرد | پیش‌فرض |
| --- | --- | --- |
| `data-cap-api-endpoint` | آدرس پایهٔ API کپچا | الزامی، مگر با `CAP_CUSTOM_FETCH` |
| `data-cap-hidden-field-name` | نام فیلد مخفی توکن | `cap-token` |
| `data-cap-worker-count` | تعداد Workerهای حل چالش | `navigator.hardwareConcurrency` یا `8` |
| `data-cap-i18n-initial-state` | متن اولیه | `من ربات نیستم!` |
| `data-cap-i18n-verifying-label` | متن هنگام بررسی | `در حال بررسی...` |
| `data-cap-i18n-solved-label` | متن موفقیت | `تایید شد!` |
| `data-cap-i18n-error-label` | متن خطا | `Error. Try again.` |
| `data-cap-i18n-wasm-disabled` | متن راهنمای حل‌کنندهٔ جایگزین | پیام انگلیسی فعال‌سازی WASM |

برای متن‌های دسترس‌پذیری نیز می‌توانید ویژگی‌های زیر را تنظیم کنید:

```html
<cap-widget
  data-cap-api-endpoint="/api/cap/"
  data-cap-i18n-verify-aria-label="برای بررسی کلیک کنید"
  data-cap-i18n-verifying-aria-label="در حال بررسی؛ لطفاً منتظر بمانید"
  data-cap-i18n-verified-aria-label="بررسی موفق بود؛ می‌توانید ادامه دهید"
  data-cap-i18n-error-aria-label="خطایی رخ داد؛ دوباره تلاش کنید"
></cap-widget>
```

تعداد Worker را یک عدد صحیح مثبت و حداکثر برابر `Math.min(navigator.hardwareConcurrency || 8, 16)` انتخاب کنید. مقدار نامعتبر باعث استفاده از مقدار پیش‌فرض می‌شود.

## متدها و رویدادها

| API روی ویجت | کاربرد |
| --- | --- |
| `await widget.solve()` | حل چالش؛ در موفقیت `{ success: true, token }` برمی‌گرداند |
| `widget.reset()` | پاک‌کردن توکن و بازگرداندن حالت اولیه |
| `widget.setWorkersCount(2)` | تنظیم تعداد Workerها |
| `widget.token` / `widget.tokenValue` | توکن فعلی یا `null` |

اگر حل چالش از قبل در حال اجرا باشد، فراخوانی مجدد `solve()` نتیجه‌ای برنمی‌گرداند. خطای حل چالش Promise را reject می‌کند؛ برای اجرای برنامه‌ای از `try/catch` استفاده کنید.

| رویداد | `event.detail` |
| --- | --- |
| `progress` | `{ progress: number }`، درصد پیشرفت |
| `solve` | `{ token: string }` |
| `error` | `{ isCap: true, message: string }` |
| `reset` | `{}` |

```js
widget.addEventListener("progress", ({ detail }) => {
  console.log(`پیشرفت: ${detail.progress}%`);
});

widget.addEventListener("solve", ({ detail }) => {
  console.log("توکن آماده است:", detail.token);
});

widget.addEventListener("error", ({ detail }) => {
  console.error(detail.message);
});
```

در callback رویداد `solve`، توکن را از `event.detail.token` بخوانید؛ این رویداد پیش از به‌روزرسانی پراپرتی `widget.token` ارسال می‌شود. کلاس `window.Cap` نیز `solve()`، `reset()`، `addEventListener()`، `token` و دسترسی به ویجت از طریق `cap.widget` را ارائه می‌کند.

## شخصی‌سازی ظاهر

متغیرهای CSS از بیرون Shadow DOM قابل تنظیم هستند:

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

فونت مثال باید جداگانه در پروژه بارگذاری شود. بخش‌های `checkbox`، `label` و `attribution` با `::part()` قابل هدف‌گیری هستند. متغیرهای دیگری مانند `--cap-checkbox-size`، `--cap-checkbox-border` و `--cap-spinner-thickness` نیز در دسترس‌اند.

## مسیر سفارشی WASM و Fetch

تنظیمات سراسری را **پیش از اجرای فایل ویجت** تعریف کنید:

```html
<script type="module">
  window.CAP_CUSTOM_WASM_URL = new URL(
    "/assets/cap/wasm/cap_wasm.min.js",
    window.location.origin
  ).href;

  await import("/vendor/cap/cap.min.js");
</script>
```

`CAP_CUSTOM_WASM_URL` باید به فایل **JavaScript راه‌انداز WASM** اشاره کند، نه فایل `.wasm`. فایل `cap_wasm_bg.wasm` را کنار آن نگه دارید. استفاده از URL مطلق، مسیر واردکردن ماژول در Worker را روشن می‌کند.

برای تغییر رفتار درخواست‌های API:

```js
window.CAP_CUSTOM_FETCH = (url, options) =>
  fetch(url, { ...options, credentials: "include" });

await import("/vendor/cap/cap.min.js");
```

این hook درخواست‌های challenge و redeem را تغییر می‌دهد؛ بارگذاری داخلی WASM از آن استفاده نمی‌کند. برای style داخلی ویجت نیز `window.CAP_CSS_NONCE` قابل تنظیم است.

## فایل‌های پکیج

| فایل | کاربرد |
| --- | --- |
| `cap.min.js` | نسخهٔ اصلی فارسی و RTL با مسیرهای نسبی WASM؛ بارگذاری به‌صورت module |
| `wasm/cap_wasm.min.js` | ماژول راه‌انداز حل‌کنندهٔ WASM |
| `wasm/cap_wasm_bg.wasm` | باینری WebAssembly |
| `cap.d.ts` | تعریف نوع‌های TypeScript |
| `cap-floating.min.js` | افزونهٔ نمایش شناور ویجت |
| `cap.compat.min.js` | نسخهٔ جداگانه با رفتار متفاوت؛ در حالت پیش‌فرض به CDN ارجاع می‌دهد |
| `src/` | فایل‌های منبع؛ رفتار آن‌ها در همهٔ جزئیات با فایل اصلی یکسان نیست |

برای استقرار محلی مستندشده، از `cap.min.js` و پوشهٔ `wasm/` استفاده کنید. فایل `cap.compat.min.js` را جایگزین هم‌ارز نسخهٔ اصلی در نظر نگیرید.

## رفع مشکلات رایج

| مشکل | بررسی پیشنهادی |
| --- | --- |
| خطای `import.meta` | فایل اصلی را با `type="module"` یا `import()` در مرورگر بارگذاری کنید |
| خطای 404 برای WASM | ساختار پوشه‌ها و URL عمومی دو فایل WASM را بررسی کنید |
| ردشدن ماژول به علت MIME type | فایل JS را با MIME مناسب JavaScript و فایل WASM را با `application/wasm` سرو کنید |
| پیام `Missing API endpoint` | آدرس API را در ویژگی ویجت یا `apiEndpoint` سازنده تنظیم کنید |
| پیام `Invalid solution` | سازگاری سرور، پاسخ challenge و نتیجهٔ redeem را بررسی کنید |
| پیام `Invalid expiration time` | زمان سرور و مقدار آیندهٔ `expires` در پاسخ redeem را بررسی کنید |
| حل چالش کند است | Network و Console را برای خطای بارگذاری WASM و فعال‌شدن fallback بررسی کنید |
| Worker با CSP مسدود می‌شود | سیاست `worker-src` باید ساخت Worker از `blob:` را مجاز کند؛ بارگذاری JS، WASM، درخواست‌های API و style داخلی را هم با سیاست پروژه هماهنگ کنید |

از HTTPS یا localhost استفاده کنید؛ حل‌کنندهٔ جایگزین به Web Crypto نیاز دارد. فایل‌ها را از وب‌سرور سرو کنید، نه با بازکردن مستقیم `file://`. این ویجت برای مرورگر است و باید در پروژه‌های دارای SSR، در سمت کلاینت بارگذاری شود.

## مشارکت و پشتیبانی

برای گزارش خطا یا پیشنهاد قابلیت، یک [Issue](https://github.com/mohammadim48/capjs-local-persian/issues) باز کنید. نسخهٔ پکیج، مرورگر، مراحل بازتولید و پیام خطا را بنویسید؛ توکن‌ها و اطلاعات حساس را از گزارش حذف کنید.

## مجوز و قدردانی

این پروژه با مجوز [Apache-2.0](./LICENSE) منتشر شده است.

بر پایهٔ [Cap](https://github.com/tiagozip/cap)، ساختهٔ **Tiago**؛ نگهداری این فورک توسط [Mohammadim48](https://github.com/mohammadim48). حقوق و مجوز اثر اصلی مطابق فایل LICENSE حفظ شده است.
