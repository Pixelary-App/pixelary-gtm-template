# Pixelary Conversion Tracking for Google Tag Manager

A Google Tag Manager **web** tag template for [Pixelary](https://pixelary.app). Pixelary sends your conversions to Meta, TikTok and ChatGPT Ads (OpenAI). One setup covers each platform's browser pixel and its server-side Conversions API.

With this template you can:

- load Pixelary on every page without a Custom HTML tag;
- send standard and custom events with value, currency, content IDs and user data from GTM variables;
- identify logged-in customers;
- pass the visitor's Google Consent Mode choices to Pixelary, and keep them in sync when the visitor changes them.

Full documentation: https://pixelary.app/docs/google-tag-manager

## Before you start

1. Create a site in the Pixelary dashboard and connect at least one platform.
2. Add every domain the container runs on to the site. Pixelary ignores events from other domains.
3. Copy your **site key** from the Install page. It looks like `px_8f2kq`.

## Install the template

- **From the Community Template Gallery:** in your container open **Templates** › **Tag Templates** › **Search Gallery**, find "Pixelary Conversion Tracking" and click **Add to workspace**.
- **Manually:** download `template.tpl` from this repository. Then open **Templates** › **Tag Templates** › **New** › **⋮** › **Import** and save.

## Actions

Every Pixelary tag has an **Action** field.

| Action | What it does | Typical trigger |
|---|---|---|
| **Load Pixelary (base tag)** | Loads `https://cdn.pixelary.app/s/<site key>.js`. Pixelary then tracks what you set up in the dashboard: PageView, forms and clicks. | Initialization – All Pages |
| **Send event** | Sends one event, for example Purchase or Lead. | Thank-you page, form success, custom dataLayer event |
| **Identify user** | Attaches customer details (email, phone and more) to the events that follow on the page. | Login or sign-up success, any page where the user is known |
| **Update consent** | Sends consent values from your own variables. Use it only if your banner doesn't set Google Consent Mode. | Your banner's "consent changed" event |

You don't have to fire the base tag before other Pixelary tags. Each tag adds its command to a queue (`window.pixelary.q`), and Pixelary runs the queue in order once it loads. If an event tag has its own site key, it also loads Pixelary. The script loads only once per page, however many tags fire.

## Fields

### Load Pixelary

| Field | Description |
|---|---|
| Site key | Required. `px_` followed by 5 to 12 lowercase letters or digits. A Constant variable works too. |
| Disable automatic events | Stops Pixelary's own PageView, form and click tracking on pages where this tag runs. Use it when every event comes from GTM; otherwise events are counted twice. You then need your own PageView tag (see Example 3). |

### Send event

| Field | Description |
|---|---|
| Site key (optional) | Leave empty if the base tag runs on the page. Enter it if this tag should also load Pixelary. |
| Event name | One of the standard events: AddPaymentInfo, AddToCart, AddToWishlist, CompleteRegistration, Contact, InitiateCheckout, Lead, PageView, Purchase, Schedule, Search, StartTrial, SubmitApplication, Subscribe, ViewContent. Or **Custom event**, or a variable. Pixelary maps standard events to each platform's equivalent. |
| Custom event name | Shown for **Custom event**. Letters, digits, spaces, underscores and hyphens, up to 50 characters. |
| Value | A number such as `49.90`. A comma decimal (`49,90`) is accepted. Anything else is dropped, and the reason is logged in Preview mode. |
| Currency | Three-letter ISO 4217 code such as `USD`. Lowercase is accepted. Meta requires it for Purchase. |
| Content IDs | A comma-separated list (`sku-1, sku-2`), a variable that returns an array of IDs, or an array of GA4 items, from which each `item_id` is used. |
| Content type | Usually `product`, or `product_group` for products with variants. |
| Content name | Product or page name. |
| Number of items | Whole number. |
| Order ID | Your order or transaction ID. |
| Event ID | Optional. If your server also sends this event to Meta, TikTok or OpenAI, use the same ID there so the platform counts it once. Leave empty and Pixelary generates one. |
| Custom parameters | Extra parameters for platforms that accept them. The fields above take priority over a row with the same name. |
| User data | Email, phone, first name, last name, external ID, city, ZIP or postal code, country. These apply to this event only. |

### Identify user

Uses the same **User data** fields. Empty fields are skipped. If every field is empty on a page, the tag sends nothing and still succeeds, so it's safe to fire it everywhere.

**Send raw values; don't hash them.** Pixelary normalizes and hashes (SHA-256) each value separately for each platform, because Meta, TikTok and OpenAI use different rules, especially for phone numbers. User data is sent only when the visitor has granted `ad_user_data`. Values that are already SHA-256 hashes are passed through unchanged, but raw values match better.

### Update consent

Four drop-downs: `ad_storage`, `ad_user_data`, `ad_personalization` and `analytics_storage`. Each takes **Don't change**, **Granted**, **Denied**, or a variable that returns `granted`, `denied`, `true` or `false`. At least one must have a value.

### Consent (all actions except Update consent)

| Field | Description |
|---|---|
| Pass Google Consent Mode state to Pixelary | On by default. See below. |

## Consent handling

### Google Consent Mode (default)

With **Pass Google Consent Mode state to Pixelary** checked, each Pixelary tag:

1. reads `ad_storage`, `ad_user_data`, `ad_personalization` and `analytics_storage` with GTM's consent API;
2. sends them to Pixelary before its own command, so an event is always judged by the current consent;
3. registers a consent listener once per page, and sends the new state when the visitor accepts or declines in your banner.

Pixelary then enforces consent itself:

- Platform pixels load only after `ad_storage` is granted.
- User data and hashed identifiers are sent only after `ad_user_data` is granted.
- Events that happen before the visitor answers are held, then sent if consent is granted and dropped if it's denied.

Consent sent from GTM takes priority over Pixelary's automatic detection of cookie banners.

> **Important:** when a site doesn't use Consent Mode, Google reports every consent type as *granted*. If your banner doesn't set Google Consent Mode, uncheck this box on every Pixelary tag. Then Pixelary detects supported banners itself, or you can send consent with **Update consent** (Example 4).

### Recommended: additional consent checks

For the strictest setup, keep GTM from firing Pixelary tags until consent is granted:

1. Open the tag › **Advanced Settings** › **Consent Settings**.
2. Choose **Require additional consent for tag to fire** and add `ad_storage`.

GTM checks consent at the moment the trigger fires, so a tag that was blocked on page load doesn't fire later by itself. On the **Load Pixelary** tag, add a second trigger on your banner's "consent updated" event (a Custom Event trigger). This is safe because the script loads only once per page.

Without additional checks Pixelary still respects consent as described above. The difference is that events reach Pixelary and wait in the browser until the visitor decides.

## Examples

### 1. Base tag on every page

- **Tag type:** Pixelary Conversion Tracking
- **Action:** Load Pixelary (base tag)
- **Site key:** `px_8f2kq`, or a Constant variable such as `{{Pixelary site key}}`
- **Pass Google Consent Mode state to Pixelary:** checked
- **Trigger:** Initialization – All Pages
- Optional: additional consent checks as described above.

Initialization runs after Consent Initialization, where consent banners set their defaults, and before other tags. That way Pixelary loads early and sees the right consent state.

### 2. Purchase on the thank-you page with dataLayer variables

Your site pushes the order to the dataLayer on the thank-you page:

```html
<script>
  window.dataLayer = window.dataLayer || [];
  dataLayer.push({ ecommerce: null });
  dataLayer.push({
    event: 'purchase',
    ecommerce: {
      transaction_id: 'ORD-1001',
      value: 49.90,
      currency: 'USD',
      items: [{ item_id: 'sku-1', item_name: 'T-shirt', price: 24.95, quantity: 2 }]
    },
    customer: { email: 'jane@example.com', phone: '+1 555 010 0000' }
  });
</script>
```

Create these **Data Layer Variables** (Data Layer Version 2):

| Variable name | Data Layer Variable Name |
|---|---|
| DLV - ecommerce.transaction_id | `ecommerce.transaction_id` |
| DLV - ecommerce.value | `ecommerce.value` |
| DLV - ecommerce.currency | `ecommerce.currency` |
| DLV - ecommerce.items | `ecommerce.items` |
| DLV - customer.email | `customer.email` |
| DLV - customer.phone | `customer.phone` |

Create a **Custom Event** trigger with event name `purchase`. Then create the tag:

- **Action:** Send event
- **Site key (optional):** empty, because the base tag already runs on every page
- **Event name:** Purchase
- **Value:** `{{DLV - ecommerce.value}}`
- **Currency:** `{{DLV - ecommerce.currency}}`
- **Content IDs:** `{{DLV - ecommerce.items}}` (each item's `item_id` is used)
- **Content type:** `product`
- **Order ID:** `{{DLV - ecommerce.transaction_id}}`
- **Event ID:** `{{DLV - ecommerce.transaction_id}}`. Use the same ID if your backend sends the Purchase server-side too.
- **User data › Email:** `{{DLV - customer.email}}`; **Phone:** `{{DLV - customer.phone}}`
- **Trigger:** the `purchase` Custom Event

### 3. Everything from GTM, including PageView (for example a single-page app)

1. On the **Load Pixelary** tag, check **Disable automatic events**.
2. Add a **Send event** tag with **Event name** PageView, fired on **All Pages** and on a **History Change** trigger.
3. Send forms and clicks with your own **Send event** tags.

### 4. Consent from a banner without Google Consent Mode

Suppose your banner pushes `{ event: 'consent_changed', consent: { marketing: true, analytics: false } }`:

1. Create the variables `DLV - consent.marketing` and `DLV - consent.analytics`.
2. Create an **Update consent** tag with:
   - `ad_storage`, `ad_user_data` and `ad_personalization` set to `{{DLV - consent.marketing}}`;
   - `analytics_storage` set to `{{DLV - consent.analytics}}`.
3. Fire it on a Custom Event trigger `consent_changed`. If the banner keeps the choice between pages, also fire it on Initialization – All Pages.
4. Uncheck **Pass Google Consent Mode state to Pixelary** on your other Pixelary tags.

## Troubleshooting

Use GTM **Preview** mode. In Preview, the template writes messages prefixed with `[Pixelary]` to the browser console.

| Symptom | Likely cause and fix |
|---|---|
| Tag status **Failed** with "Site key … is not valid" | The key must be `px_` plus 5 to 12 lowercase letters or digits. Check for a variable that resolves to something else on this page. |
| Tag status **Failed** with "Could not load https://cdn.pixelary.app/s/…" | An ad blocker or a Content Security Policy blocked the script. Allow `script-src https://cdn.pixelary.app` and `connect-src https://api.pixelary.app` in your CSP. |
| Tag status **Failed** with "Event name is empty" | **Custom event** is selected but the name is empty, or the event-name variable is undefined on this page. |
| Tags succeed but nothing reaches Pixelary | No tag loaded the script: there is no Load tag and no site key on the event tags. Run `pixelary.q` in the browser console; if commands are piling up there, add the base tag. Also make sure this domain is added to the site in the dashboard. |
| Events appear only after the banner is accepted | Expected: Pixelary holds events until the visitor grants consent. Check the **Consent** tab in Tag Assistant. |
| Consent never updates | Your banner doesn't call Google Consent Mode `update`. Check the Consent tab in Tag Assistant after you click Accept. Fix the banner integration or use **Update consent**. |
| PageView counted twice | Automatic events are on and you also send PageView from GTM. Check **Disable automatic events** on the Load tag (see Example 3). |
| Purchase counted twice (browser and server) | Use the same **Event ID** in this tag and in your server-side event. |
| Value or currency missing | The value must be a number and the currency a three-letter code. Preview mode logs why a field was dropped. |
| Pixelary is also installed with a `<script>` tag | That's safe: Pixelary runs once per page. Keep only one install method to keep things simple. |

## What the template can access

GTM shows these permissions on the template's **Permissions** tab:

| Permission | Scope | Why |
|---|---|---|
| Injects scripts | `https://cdn.pixelary.app/s/*` only | Loads the Pixelary script for your site key. |
| Accesses global variables | `pixelary` (read, write, execute) and `pixelary.q` (read, write) | Creates the command queue and calls Pixelary. |
| Accesses consent state | Read `ad_storage`, `ad_user_data`, `ad_personalization`, `analytics_storage` | Passes Consent Mode state to Pixelary. |
| Accesses template storage | This template only | Registers consent listeners once per page and avoids duplicate commands. |
| Logs to console | Debug and Preview mode only | Explains failures while you test. |

The template doesn't send network requests or set cookies itself. Everything goes through the Pixelary script, which applies the consent rules above.

## Commands (for developers)

The template calls the same JavaScript API you can use on your site:

```js
pixelary('source', 'gtm', { template_version: '1.0.0' });
pixelary('config', { auto_events: false });
pixelary('consent', { ad_storage: 'granted', ad_user_data: 'granted', ad_personalization: 'denied', analytics_storage: 'granted' }, { source: 'gtm' });
pixelary('track', 'Purchase', { value: 49.9, currency: 'USD', content_ids: ['sku-1'], order_id: 'ORD-1001' }, { event_id: 'ORD-1001', user: { email: 'jane@example.com' } });
pixelary('identify', { email: 'jane@example.com', external_id: 'u_123' });
```

## Support

Open an issue in this repository or see https://pixelary.app/docs/google-tag-manager.

## License

Apache License 2.0. See [LICENSE](LICENSE).
