---
title: "Standard QR Flow - WebButton"
---

# Standard QR Flow - Web Button

An embeddable, self-contained JavaScript payment button that lets any merchant website accept Fex wallet payments with a few lines of code.

---

## How It Works

```text
Merchant site          Fex Pay Widget              Fex Backend            Customer's Fex App
──────────────         ─────────────────           ────────────           ──────────────────
FexPay.pay()    ──▶   POST /payment-intents  ──▶  Create intent
                       Show QR modal         ◀──   { qr_code_url, id }
                       Poll /payment-intents
                                                                   ◀──  Customer scans QR
                                                   status: completed ──▶
                       onSuccess callback    ◀──   Intent completed
FexPay.confirm()──▶   Show success screen
```

1. Merchant calls `FexPay.pay()` with amount, currency, and optional description.
2. The widget creates a **payment intent** on the Fex backend and renders a modal with the QR code.
3. The customer opens the **Fex mobile app**, scans the QR, and approves the payment.
4. The widget polls the backend every 5 seconds (up to 5 minutes) and automatically detects when the payment completes.
5. On completion, the `onSuccess` callback fires and the success screen is shown.

Alternatively, if the merchant receives a **webhook** from the backend, they can call `FexPay.confirm()` to resolve the payment immediately without waiting for the next poll cycle.

---

## Installation

```html
<script src="https://fex.app/widget/v1/fex-pay.js"></script>
```

The script is a self-contained IIFE with **no external dependencies**. It exposes a single global: `window.FexPay`.

---

## Quick Start

```html
<button id="checkout-btn">Pay with Fex</button>

<script src="/js/fex-pay.js"></script>
<script>
  // 1. Initialize once with your Merchant ID
  FexPay.init({
    merchantId: 'YOUR_MERCHANT_UUID',
  });

  // 2. Trigger a payment when the customer clicks Pay
  document.getElementById('checkout-btn').addEventListener('click', function () {
    FexPay.pay({
      amount: 49.99,
      currency: 'USD',
      description: 'Order #1042',
      onSuccess: function (intent) {
        console.log('Paid!', intent.id);
        window.location.href = '/order/success';
      },
      onError: function (err) {
        console.error('Payment failed:', err.message);
      },
      onCancel: function () {
        console.log('Customer closed the modal');
      },
    });
  });
</script>
```

---

## API Reference

### `FexPay.init(config)`

Must be called **once** before any `pay()` call. Typically called on page load.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `config.merchantId` | `string` | Yes | Your Fex merchant UUID |
| `config.baseUrl` | `string` | No | Override the API base URL (default: production endpoint) |

```js
FexPay.init({
  merchantId: '74815c83-e47b-4dc5-aefe-abe4484dd280',
  // baseUrl: 'https://staging-api.fex.app/merchant/api/v1', // optional
});
```

Throws a synchronous `Error` if `merchantId` is missing.

---

### `FexPay.pay(params)`

Opens the payment modal, creates a payment intent, and starts polling for completion. Returns a `Promise` that resolves with the created `PaymentIntent` object once the modal appears (not when payment completes — use `onSuccess` for that).

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `params.amount` | `number` | Yes | Amount to charge. Must be a positive number |
| `params.currency` | `string` | Yes | ISO 4217 currency code, e.g. `"USD"`, `"EUR"`, `"GBP"` |
| `params.description` | `string` | No | Payment description shown to the customer in the modal |
| `params.callbackUrl` | `string` | No | Webhook URL the backend will POST to when payment status changes |
| `params.onSuccess` | `function` | No | Called when payment completes successfully |
| `params.onError` | `function` | No | Called when payment fails or times out |
| `params.onCancel` | `function` | No | Called when the customer closes the modal |

#### `onSuccess(intent)`

Called with the completed `PaymentIntent` object:

```js
onSuccess: function (intent) {
  // intent.id         — payment intent UUID
  // intent.amount     — number
  // intent.currency   — string
  // intent.status     — "completed" | "paid"
  // intent.merchant_id
  // ...
}
```

#### `onError(error)`

Called with an error object:

```js
onError: function (err) {
  // err.code     — one of: "VALIDATION" | "API_ERROR" | "TIMEOUT" | "PAYMENT_FAILED"
  // err.message  — human-readable description
  // err.status   — HTTP status code (API_ERROR only)
}
```

| Error code | Cause |
| --- | --- |
| `VALIDATION` | Invalid `params` passed to `pay()` (e.g. missing amount) |
| `API_ERROR` | Network or backend error when creating/fetching the payment intent |
| `TIMEOUT` | Customer did not complete payment within 5 minutes |
| `PAYMENT_FAILED` | Intent status became `expired`, `cancelled`, or `failed` |

#### `onCancel()`

Called when the customer clicks the × button or the backdrop. No arguments.

#### Example with all params

```js
FexPay.pay({
  amount: 129.99,
  currency: 'USD',
  description: 'Pro subscription — annual plan',
  callbackUrl: 'https://yourshop.com/api/fex/webhook',
  onSuccess: function (intent) {
    // Redirect to a confirmation page
    window.location.href = '/checkout/complete?ref=' + intent.id;
  },
  onError: function (err) {
    if (err.code === 'TIMEOUT') {
      alert('Session expired. Please try again.');
    } else {
      alert('Payment error: ' + err.message);
    }
  },
  onCancel: function () {
    // Nothing — customer decided not to pay
  },
});
```

---

### `FexPay.confirm(paymentIntentId?)`

Tells the widget that a payment has been confirmed — typically called from your own webhook handler after the Fex backend notifies your server. Stops polling immediately, fetches the latest intent data, and shows the success screen.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| `paymentIntentId` | `string` | No | The payment intent UUID to confirm. If omitted, confirms the current active session |

```js
// Called inside your webhook listener on the client side
// (e.g. via WebSocket or Server-Sent Events from your server)
FexPay.confirm('a0eebc99-9c0b-4ef8-bb6d-6bb9bd380a11');
```

If no active session exists, or the ID does not match the active session, the call is silently ignored (a warning is logged to the console).

---

## Webhook Integration

Using `callbackUrl` \+ `FexPay.confirm()` is the most reliable way to detect payment completion — it avoids waiting for the next 5-second poll cycle.

### Recommended flow

```text
1. Customer clicks Pay                     → FexPay.pay({ callbackUrl: '...' })
2. Backend receives payment confirmation   → POST https://yourshop.com/api/fex/webhook
3. Your server validates the webhook
4. Your server pushes the intent ID to the client (WebSocket / SSE)
5. Client calls FexPay.confirm(intentId)   → modal shows success immediately
```

### Server-side webhook handler (Node.js example)

```js
// POST /api/fex/webhook
app.post('/api/fex/webhook', (req, res) => {
  const { payment_intent } = req.body;

  if (payment_intent.status === 'completed') {
    // Update your order in the database
    fulfillOrder(payment_intent.id);

    // Push to the browser via WebSocket / SSE so the client can call FexPay.confirm()
    notifyClient(payment_intent.merchant_id, payment_intent.id);
  }

  res.json({ received: true });
});
```

### Client-side with WebSocket

```js
const ws = new WebSocket('wss://yourshop.com/ws');

ws.addEventListener('message', function (event) {
  const msg = JSON.parse(event.data);
  if (msg.type === 'payment_confirmed') {
    FexPay.confirm(msg.paymentIntentId);
  }
});
```

### Client-side with Server-Sent Events

```js
const sse = new EventSource('/api/events');

sse.addEventListener('payment_confirmed', function (event) {
  const { paymentIntentId } = JSON.parse(event.data);
  FexPay.confirm(paymentIntentId);
});
```

---

## Polling Behaviour

When no webhook is used, the widget polls automatically:

| Setting | Value |
| --- | --- |
| First poll delay | 1 second after QR is shown |
| Poll interval | 5 seconds |
| Maximum polls | 120 (≈ 5 minutes total) |
| Timeout behaviour | Shows error, fires `onError({ code: 'TIMEOUT' })` |
| Transient errors | Logged to console, polling continues |

---

## Modal States

| State | Trigger |
| --- | --- |
| **Loading** | While `POST /payment-intents` is in progress |
| **QR code** | Payment intent created successfully |
| **Success** | Payment completed (`onSuccess` also fires) |
| **Error** | API failure or payment expired/cancelled (`onError` also fires) |

The success screen auto-closes after **3 seconds**.

---

## Security Considerations

- The widget sends your **Merchant ID** as an `X-Merchant-ID` header on every API request. This ID is visible in client-side code — it is a public identifier, not a secret.
- **Never embed API keys or secrets** in the widget configuration. Authentication of payment outcomes should be verified server-side using the webhook.
- The modal is rendered in a **Shadow DOM** (`mode: closed`) so widget styles are fully isolated from your page and vice versa.
- All dynamic text inserted into the modal is **HTML-escaped** to prevent XSS.

---

## Browser Support

| Browser | Support |
| --- | --- |
| Chrome / Edge 80\+ | Full |
| Firefox 75\+ | Full |
| Safari 14\+ | Full |
| iOS Safari 14\+ | Full |
| Android Chrome 80\+ | Full |

Requires: `fetch`, `Promise`, `Shadow DOM`, `Intl.NumberFormat` — all standard in modern browsers. No polyfills needed.

---

## Demo

Open `widget/demo.html` directly in a browser (or serve it locally) to test the widget interactively:

```bash
# Serve locally — any static server works
npx serve widget/
# then open http://localhost:3000/demo.html
```

The demo page lets you configure Merchant ID, amount, currency, and description, shows a live event log, and includes a **Confirm** button to simulate merchant-side webhook confirmation.

---

## File Reference

| File | Description |
| --- | --- |
| `fex-pay.js` | The widget — copy this to your project or CDN |
| `demo.html` | Interactive demo page |