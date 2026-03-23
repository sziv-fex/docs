# FEX Payment Platform — Merchant & POS Integration Guide

> **Who is this for?** This guide covers everything you need to integrate a point-of-sale terminal or backend system with the FEX Payment Platform — from obtaining credentials through to handling webhooks and managing refunds.

---

## Table of Contents

1. [Overview](#1-overview)
2. [Getting Your Credentials](#2-getting-your-credentials)
3. [Generating an Access Token](#3-generating-an-access-token)
4. [Standard QR Flow](#4-standard-qr-flow)
5. [Reverse QR Flow (Counter Scan)](#5-reverse-qr-flow-counter-scan)
6. [Webhooks](#6-webhooks)
7. [Cancellations & Refunds](#7-cancellations--refunds)
8. [Transaction History](#8-transaction-history)
9. [Counter Management](#9-counter-management)
10. [Error Reference](#10-error-reference)
11. [Quick-Start Checklist](#11-quick-start-checklist)

---

## 1. Overview

The FEX Payment Platform supports two payment flows. Choose the one that fits your checkout experience:

| Flow | Initiated by | How it works |
|------|-------------|--------------|
| **Standard QR** | Merchant / POS terminal | Terminal generates a QR code; customer scans it with their wallet app |
| **Reverse QR** | POS terminal (counter scan) | Customer shows their wallet QR; terminal scans it and sends a push approval to the customer |

<br />

<!-- DIAGRAM: Payment Flow Comparison -->
<p align="center">
<svg width="100%" viewBox="0 0 680 300" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <marker id="arr" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M2 1L8 5L2 9" fill="none" stroke="context-stroke" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
    </marker>
  </defs>

  {/* LEFT: Standard QR */}
  <text x="170" y="30" text-anchor="middle" font-family="system-ui,sans-serif" font-size="13" font-weight="600" fill="#1a1a1a">Standard QR</text>

  {/* POS Terminal box */}
  <rect x="40" y="50" width="120" height="44" rx="8" fill="#EEF2FF" stroke="#6366F1" stroke-width="1"/>
  <text x="100" y="68" text-anchor="middle" font-family="system-ui,sans-serif" font-size="12" font-weight="600" fill="#3730A3">POS Terminal</text>
  <text x="100" y="83" text-anchor="middle" font-family="system-ui,sans-serif" font-size="11" fill="#4338CA">Generates QR</text>

  {/* Arrow down */}
  <line x1="100" y1="94" x2="100" y2="138" stroke="#6366F1" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="110" y="120" font-family="system-ui,sans-serif" font-size="10" fill="#6366F1">displays</text>

  {/* QR Code icon */}
  <rect x="72" y="140" width="56" height="56" rx="6" fill="#F5F5F5" stroke="#CBD5E1" stroke-width="1"/>
  <rect x="80" y="148" width="18" height="18" rx="2" fill="#1a1a1a"/>
  <rect x="104" y="148" width="16" height="8" rx="1" fill="#1a1a1a"/>
  <rect x="104" y="160" width="16" height="6" rx="1" fill="#1a1a1a"/>
  <rect x="80" y="172" width="18" height="12" rx="1" fill="#1a1a1a"/>
  <rect x="104" y="170" width="6" height="18" rx="1" fill="#1a1a1a"/>
  <rect x="114" y="170" width="6" height="18" rx="1" fill="#1a1a1a"/>

  {/* Arrow right */}
  <line x1="130" y1="168" x2="200" y2="168" stroke="#6366F1" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="165" y="162" text-anchor="middle" font-family="system-ui,sans-serif" font-size="10" fill="#6366F1">scans</text>

  {/* Customer box */}
  <rect x="200" y="148" width="110" height="44" rx="8" fill="#F0FDF4" stroke="#22C55E" stroke-width="1"/>
  <text x="255" y="166" text-anchor="middle" font-family="system-ui,sans-serif" font-size="12" font-weight="600" fill="#166534">Customer</text>
  <text x="255" y="181" text-anchor="middle" font-family="system-ui,sans-serif" font-size="11" fill="#15803D">Wallet app</text>

  {/* Arrow up to success */}
  <line x1="255" y1="148" x2="255" y2="104" stroke="#22C55E" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="268" y="130" font-family="system-ui,sans-serif" font-size="10" fill="#22C55E">confirms</text>

  {/* Success badge */}
  <rect x="190" y="58" width="130" height="44" rx="8" fill="#F0FDF4" stroke="#22C55E" stroke-width="1.5"/>
  <text x="255" y="76" text-anchor="middle" font-family="system-ui,sans-serif" font-size="12" font-weight="600" fill="#166534">Payment Complete</text>
  <text x="255" y="91" text-anchor="middle" font-family="system-ui,sans-serif" font-size="11" fill="#15803D">Webhook fired</text>

  {/* divider */}
  <line x1="340" y1="40" x2="340" y2="270" stroke="#E2E8F0" stroke-width="1" stroke-dasharray="4 4"/>

  {/* RIGHT: Reverse QR */}
  <text x="510" y="30" text-anchor="middle" font-family="system-ui,sans-serif" font-size="13" font-weight="600" fill="#1a1a1a">Reverse QR</text>

  {/* Customer shows QR */}
  <rect x="360" y="50" width="120" height="44" rx="8" fill="#F0FDF4" stroke="#22C55E" stroke-width="1"/>
  <text x="420" y="68" text-anchor="middle" font-family="system-ui,sans-serif" font-size="12" font-weight="600" fill="#166534">Customer</text>
  <text x="420" y="83" text-anchor="middle" font-family="system-ui,sans-serif" font-size="11" fill="#15803D">Shows wallet QR</text>

  {/* Arrow right (scan) */}
  <line x1="480" y1="72" x2="538" y2="72" stroke="#6366F1" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="509" y="65" text-anchor="middle" font-family="system-ui,sans-serif" font-size="10" fill="#6366F1">scans</text>

  {/* POS Terminal (right) */}
  <rect x="540" y="50" width="110" height="44" rx="8" fill="#EEF2FF" stroke="#6366F1" stroke-width="1"/>
  <text x="595" y="68" text-anchor="middle" font-family="system-ui,sans-serif" font-size="12" font-weight="600" fill="#3730A3">POS Terminal</text>
  <text x="595" y="83" text-anchor="middle" font-family="system-ui,sans-serif" font-size="11" fill="#4338CA">Sends intent</text>

  {/* Arrow down from POS to API */}
  <line x1="595" y1="94" x2="595" y2="138" stroke="#6366F1" stroke-width="1.5" marker-end="url(#arr)"/>

  {/* FEX API */}
  <rect x="530" y="140" width="130" height="44" rx="8" fill="#FFF7ED" stroke="#F97316" stroke-width="1"/>
  <text x="595" y="158" text-anchor="middle" font-family="system-ui,sans-serif" font-size="12" font-weight="600" fill="#9A3412">FEX API</text>
  <text x="595" y="173" text-anchor="middle" font-family="system-ui,sans-serif" font-size="11" fill="#C2410C">Push notification</text>

  {/* Arrow left back to customer (approval) */}
  <line x1="530" y1="162" x2="488" y2="162" stroke="#F97316" stroke-width="1.5" marker-end="url(#arr)"/>
  <text x="509" y="155" text-anchor="middle" font-family="system-ui,sans-serif" font-size="10" fill="#F97316">approve</text>

  {/* Customer approves */}
  <rect x="360" y="140" width="126" height="44" rx="8" fill="#F0FDF4" stroke="#22C55E" stroke-width="1"/>
  <text x="423" y="158" text-anchor="middle" font-family="system-ui,sans-serif" font-size="12" font-weight="600" fill="#166534">Customer</text>
  <text x="423" y="173" text-anchor="middle" font-family="system-ui,sans-serif" font-size="11" fill="#15803D">Taps approve</text>

  {/* Arrow down to success */}
  <line x1="595" y1="184" x2="595" y2="228" stroke="#22C55E" stroke-width="1.5" marker-end="url(#arr)"/>

  {/* Success badge right */}
  <rect x="486" y="230" width="218" height="44" rx="8" fill="#F0FDF4" stroke="#22C55E" stroke-width="1.5"/>
  <text x="595" y="248" text-anchor="middle" font-family="system-ui,sans-serif" font-size="12" font-weight="600" fill="#166534">Payment Complete</text>
  <text x="595" y="263" text-anchor="middle" font-family="system-ui,sans-serif" font-size="11" fill="#15803D">Webhook fired to POS</text>
</svg>
</p>

---

## 2. Getting Your Credentials

> **This step is completed once per merchant.** A platform administrator registers your merchant account and provides API credentials.

After registration you will receive a JSON payload containing three values. **Store `client_secret` immediately** — it cannot be retrieved again. If lost, you must request a new one from your platform administrator.

```json
{
  "merchant_id":   "a1b2c3d4-...",
  "client_id":     "merchant-a1b2c3d4-...",
  "client_secret": "xxxxxxxxxxxxxxxx",
  "note": "Store client_secret securely — it will not be shown again."
}
```

> **Best practice:** Store `client_secret` in a dedicated secrets manager (e.g. AWS Secrets Manager, HashiCorp Vault, 1Password Secrets Automation) rather than in environment variables or source code.

---

## 3. Generating an Access Token

All API calls require a **Bearer token** obtained via the OAuth 2.0 client credentials flow. Tokens expire after **5 minutes** — implement automatic refresh in your backend.

### Request

```bash
curl -X POST https://auth.fex.example.com/realms/payment-platform/protocol/openid-connect/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials" \
  -d "client_id=merchant-a1b2c3d4-..." \
  -d "client_secret=xxxxxxxxxxxxxxxx"
```

### Response

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIs...",
  "expires_in": 300,
  "token_type": "Bearer"
}
```

### Token Caching (Recommended Pattern)

Request a new token before each API call, or cache and refresh proactively using the pattern below:

```python
if token is None or token_expires_at < now() + 30s:
    token = fetch_new_token(client_id, client_secret)
    token_expires_at = now() + token.expires_in
```

> **Important:** Always refresh at least 30 seconds before expiry to avoid race conditions near the 5-minute boundary.

<br />

<!-- DIAGRAM: Token Flow -->
<p align="center">
<svg width="100%" viewBox="0 0 680 160" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <marker id="arr2" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M2 1L8 5L2 9" fill="none" stroke="context-stroke" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
    </marker>
  </defs>

  {/* POS Backend */}
  <rect x="40" y="58" width="130" height="44" rx="8" fill="#EEF2FF" stroke="#6366F1" stroke-width="1"/>
  <text x="105" y="76" text-anchor="middle" font-family="system-ui,sans-serif" font-size="12" font-weight="600" fill="#3730A3">POS Backend</text>
  <text x="105" y="91" text-anchor="middle" font-family="system-ui,sans-serif" font-size="11" fill="#4338CA">Your server</text>

  {/* Arrow to Auth */}
  <line x1="170" y1="80" x2="232" y2="80" stroke="#6366F1" stroke-width="1.5" marker-end="url(#arr2)"/>
  <text x="201" y="72" text-anchor="middle" font-family="system-ui,sans-serif" font-size="10" fill="#6366F1">credentials</text>

  {/* Auth Server */}
  <rect x="234" y="58" width="130" height="44" rx="8" fill="#FFF7ED" stroke="#F97316" stroke-width="1"/>
  <text x="299" y="76" text-anchor="middle" font-family="system-ui,sans-serif" font-size="12" font-weight="600" fill="#9A3412">Auth Server</text>
  <text x="299" y="91" text-anchor="middle" font-family="system-ui,sans-serif" font-size="11" fill="#C2410C">Keycloak / OAuth2</text>

  {/* Arrow back (token) */}
  <line x1="363" y1="80" x2="425" y2="80" stroke="#22C55E" stroke-width="1.5" marker-end="url(#arr2)"/>
  <text x="394" y="72" text-anchor="middle" font-family="system-ui,sans-serif" font-size="10" fill="#22C55E">access token</text>

  {/* Token cache */}
  <rect x="428" y="58" width="130" height="44" rx="8" fill="#F0FDF4" stroke="#22C55E" stroke-width="1"/>
  <text x="493" y="76" text-anchor="middle" font-family="system-ui,sans-serif" font-size="12" font-weight="600" fill="#166534">POS Backend</text>
  <text x="493" y="91" text-anchor="middle" font-family="system-ui,sans-serif" font-size="11" fill="#15803D">Caches token</text>

  {/* Arrow to FEX API */}
  <line x1="558" y1="80" x2="618" y2="80" stroke="#6366F1" stroke-width="1.5" marker-end="url(#arr2)"/>
  <text x="588" y="72" text-anchor="middle" font-family="system-ui,sans-serif" font-size="10" fill="#6366F1">API calls</text>

  {/* FEX API */}
  <rect x="620" y="58" width="40" height="44" rx="8" fill="#F8FAFC" stroke="#CBD5E1" stroke-width="1"/>
  <text x="640" y="84" text-anchor="middle" font-family="system-ui,sans-serif" font-size="10" font-weight="600" fill="#475569">FEX</text>

  {/* Expiry note */}
  <text x="340" y="130" text-anchor="middle" font-family="system-ui,sans-serif" font-size="11" fill="#94A3B8">Token valid for 5 minutes — refresh at least 30 s before expiry</text>
</svg>
</p>

---

## 4. Standard QR Flow

The POS terminal creates a payment intent, displays the resulting QR code, and waits for the customer to scan and confirm via their wallet app.

<br />

<!-- DIAGRAM: Standard QR Sequence -->
<p align="center">
<svg width="100%" viewBox="0 0 680 380" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <marker id="arr3" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M2 1L8 5L2 9" fill="none" stroke="context-stroke" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
    </marker>
  </defs>

  {/* Swimlane headers */}
  <rect x="40" y="20" width="130" height="36" rx="6" fill="#EEF2FF" stroke="#6366F1" stroke-width="1"/>
  <text x="105" y="42" text-anchor="middle" font-family="system-ui,sans-serif" font-size="12" font-weight="600" fill="#3730A3">POS Terminal</text>

  <rect x="270" y="20" width="130" height="36" rx="6" fill="#FFF7ED" stroke="#F97316" stroke-width="1"/>
  <text x="335" y="42" text-anchor="middle" font-family="system-ui,sans-serif" font-size="12" font-weight="600" fill="#9A3412">FEX API</text>

  <rect x="500" y="20" width="140" height="36" rx="6" fill="#F0FDF4" stroke="#22C55E" stroke-width="1"/>
  <text x="570" y="42" text-anchor="middle" font-family="system-ui,sans-serif" font-size="12" font-weight="600" fill="#166534">Customer Wallet</text>

  {/* Lifelines */}
  <line x1="105" y1="56" x2="105" y2="360" stroke="#CBD5E1" stroke-width="1" stroke-dasharray="4 4"/>
  <line x1="335" y1="56" x2="335" y2="360" stroke="#CBD5E1" stroke-width="1" stroke-dasharray="4 4"/>
  <line x1="570" y1="56" x2="570" y2="360" stroke="#CBD5E1" stroke-width="1" stroke-dasharray="4 4"/>

  {/* Step 1: POS → API */}
  <line x1="105" y1="90" x2="328" y2="90" stroke="#6366F1" stroke-width="1.5" marker-end="url(#arr3)"/>
  <text x="216" y="83" text-anchor="middle" font-family="system-ui,sans-serif" font-size="10" fill="#6366F1">POST /payment-intents</text>

  {/* Step 2: API → POS (response) */}
  <line x1="335" y1="120" x2="112" y2="120" stroke="#F97316" stroke-width="1.5" stroke-dasharray="5 3" marker-end="url(#arr3)"/>
  <text x="224" y="113" text-anchor="middle" font-family="system-ui,sans-serif" font-size="10" fill="#F97316">201 { qr_code_url, qr_data }</text>

  {/* Step 3: POS displays */}
  <rect x="56" y="140" width="98" height="28" rx="6" fill="#EEF2FF" stroke="#6366F1" stroke-width="0.5"/>
  <text x="105" y="158" text-anchor="middle" font-family="system-ui,sans-serif" font-size="10" fill="#3730A3">Display QR code</text>

  {/* Step 4: Customer scans */}
  <line x1="570" y1="188" x2="342" y2="188" stroke="#22C55E" stroke-width="1.5" marker-end="url(#arr3)"/>
  <text x="456" y="181" text-anchor="middle" font-family="system-ui,sans-serif" font-size="10" fill="#22C55E">Scan QR (GET /payment-intents/{id})</text>

  {/* Step 5: Customer confirms */}
  <rect x="520" y="208" width="100" height="28" rx="6" fill="#F0FDF4" stroke="#22C55E" stroke-width="0.5"/>
  <text x="570" y="226" text-anchor="middle" font-family="system-ui,sans-serif" font-size="10" fill="#166534">Tap Confirm</text>

  {/* Step 6: API → POS webhook */}
  <line x1="335" y1="258" x2="112" y2="258" stroke="#F97316" stroke-width="1.5" stroke-dasharray="5 3" marker-end="url(#arr3)"/>
  <text x="224" y="250" text-anchor="middle" font-family="system-ui,sans-serif" font-size="10" fill="#F97316">Webhook: payment.completed</text>

  {/* Step 7: Success */}
  <rect x="56" y="276" width="98" height="28" rx="6" fill="#F0FDF4" stroke="#22C55E" stroke-width="0.5"/>
  <text x="105" y="294" text-anchor="middle" font-family="system-ui,sans-serif" font-size="10" fill="#166534">Show success</text>

  {/* Step labels */}
  <text x="28" y="93" font-family="system-ui,sans-serif" font-size="9" fill="#94A3B8">①</text>
  <text x="28" y="123" font-family="system-ui,sans-serif" font-size="9" fill="#94A3B8">②</text>
  <text x="28" y="157" font-family="system-ui,sans-serif" font-size="9" fill="#94A3B8">③</text>
  <text x="28" y="191" font-family="system-ui,sans-serif" font-size="9" fill="#94A3B8">④</text>
  <text x="28" y="225" font-family="system-ui,sans-serif" font-size="9" fill="#94A3B8">⑤</text>
  <text x="28" y="261" font-family="system-ui,sans-serif" font-size="9" fill="#94A3B8">⑥</text>
  <text x="28" y="292" font-family="system-ui,sans-serif" font-size="9" fill="#94A3B8">⑦</text>

  {/* Poll note */}
  <rect x="56" y="315" width="260" height="30" rx="6" fill="#F8FAFC" stroke="#E2E8F0" stroke-width="1"/>
  <text x="186" y="330" text-anchor="middle" font-family="system-ui,sans-serif" font-size="10" fill="#64748B">Optional: Poll GET /payment-intents/{id} every 2–3 s</text>
  <text x="186" y="340" text-anchor="middle" font-family="system-ui,sans-serif" font-size="10" fill="#64748B">as a fallback alongside webhooks</text>
</svg>
</p>

### Step 1 — Create a Payment Intent

**`POST /merchant/api/v1/payment-intents`**

**Headers:**
```
Authorization: Bearer {access_token}
Content-Type: application/json
```

**Request body:**
```json
{
  "amount": 29.99,
  "currency": "USD",
  "description": "Coffee x2",
  "expires_in": 300,
  "reference": "ORDER-1042",
  "callback_url": "https://your-backend.example.com/webhooks/fex",
  "metadata": { "table": "5", "cashier": "Maria" }
}
```

**Request fields:**

| Field | Type | Required | Description |
|-------|------|:--------:|-------------|
| `amount` | float | ✅ | Payment amount (must be positive) |
| `currency` | string | — | ISO 4217 code. Default: `USD` |
| `description` | string | — | Shown to customer in their wallet app |
| `expires_in` | integer | — | Seconds until QR expires. Default: `1800` |
| `reference` | string | — | Your internal order or invoice ID |
| `callback_url` | string | — | URL to receive payment webhook events |
| `counter_id` | string | — | Links intent to a specific POS terminal |
| `metadata` | object | — | Arbitrary key-value pairs, echoed in webhooks |

**Response `201 Created`:**
```json
{
  "success": true,
  "payment_intent": {
    "id": "pi_xxxx",
    "merchant_id": "a1b2c3d4-...",
    "merchant_name": "My Shop",
    "amount": 29.99,
    "currency": "USD",
    "description": "Coffee x2",
    "status": "pending",
    "qr_code_url": "data:image/png;base64,...",
    "qr_data": "fex://pay?intent=pi_xxxx&amount=29.99&currency=USD&merchant=a1b2c3d4-...",
    "expires_at": "2025-01-01T12:35:00Z",
    "reference": "ORDER-1042",
    "created_at": "2025-01-01T12:30:00Z"
  }
}
```

> **Displaying the QR code:** Use `qr_code_url` directly as an `<img src="...">` — it is a base64-encoded PNG. Alternatively, encode `qr_data` using your preferred QR library.

---

### Step 2 — Poll for Status (Optional)

**`GET /merchant/api/v1/payment-intents/{id}`**

Poll every 2–3 seconds while the QR is displayed. Stop when `status` reaches a terminal state.

**Response:**
```json
{
  "success": true,
  "payment_intent": {
    "id": "pi_xxxx",
    "status": "completed",
    "completed_at": "2025-01-01T12:31:55Z"
  }
}
```

**Payment intent statuses:**

| Status | Meaning |
|--------|---------|
| `pending` | Waiting for customer to scan and pay |
| `completed` | Payment received |
| `cancelled` | Cancelled by merchant |
| `expired` | QR code lifetime elapsed |
| `refunded` | Full refund issued |

> **Webhook vs polling:** Webhooks are the primary notification mechanism. Polling is a recommended fallback, not a replacement.

---

## 5. Reverse QR Flow (Counter Scan)

The POS terminal scans the **customer's** QR code. The customer receives a push notification in their wallet app and taps to approve.

<br />

<!-- DIAGRAM: Reverse QR Sequence -->
<p align="center">
<svg width="100%" viewBox="0 0 680 400" xmlns="http://www.w3.org/2000/svg">
  <defs>
    <marker id="arr4" viewBox="0 0 10 10" refX="8" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse">
      <path d="M2 1L8 5L2 9" fill="none" stroke="context-stroke" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
    </marker>
  </defs>

  {/* Swimlane headers */}
  <rect x="10" y="20" width="110" height="36" rx="6" fill="#F0FDF4" stroke="#22C55E" stroke-width="1"/>
  <text x="65" y="42" text-anchor="middle" font-family="system-ui,sans-serif" font-size="11" font-weight="600" fill="#166534">Customer</text>

  <rect x="200" y="20" width="110" height="36" rx="6" fill="#EEF2FF" stroke="#6366F1" stroke-width="1"/>
  <text x="255" y="42" text-anchor="middle" font-family="system-ui,sans-serif" font-size="11" font-weight="600" fill="#3730A3">POS Terminal</text>

  <rect x="380" y="20" width="100" height="36" rx="6" fill="#FFF7ED" stroke="#F97316" stroke-width="1"/>
  <text x="430" y="42" text-anchor="middle" font-family="system-ui,sans-serif" font-size="11" font-weight="600" fill="#9A3412">FEX API</text>

  <rect x="560" y="20" width="110" height="36" rx="6" fill="#F0FDF4" stroke="#22C55E" stroke-width="1"/>
  <text x="615" y="42" text-anchor="middle" font-family="system-ui,sans-serif" font-size="11" font-weight="600" fill="#166534">Wallet Service</text>

  {/* Lifelines */}
  <line x1="65"  y1="56" x2="65"  y2="380" stroke="#CBD5E1" stroke-width="1" stroke-dasharray="4 4"/>
  <line x1="255" y1="56" x2="255" y2="380" stroke="#CBD5E1" stroke-width="1" stroke-dasharray="4 4"/>
  <line x1="430" y1="56" x2="430" y2="380" stroke="#CBD5E1" stroke-width="1" stroke-dasharray="4 4"/>
  <line x1="615" y1="56" x2="615" y2="380" stroke="#CBD5E1" stroke-width="1" stroke-dasharray="4 4"/>

  {/* Step 1: Customer shows QR */}
  <line x1="65" y1="90" x2="246" y2="90" stroke="#22C55E" stroke-width="1.5" marker-end="url(#arr4)"/>
  <text x="155" y="82" text-anchor="middle" font-family="system-ui,sans-serif" font-size="10" fill="#22C55E">Shows wallet QR (scan_token)</text>

  {/* Step 2: POS scans */}
  <rect x="206" y="105" width="98" height="24" rx="5" fill="#EEF2FF" stroke="#6366F1" stroke-width="0.5"/>
  <text x="255" y="121" text-anchor="middle" font-family="system-ui,sans-serif" font-size="10" fill="#3730A3">Extract scan_token</text>

  {/* Step 3: POS → API */}
  <line x1="255" y1="148" x2="422" y2="148" stroke="#6366F1" stroke-width="1.5" marker-end="url(#arr4)"/>
  <text x="338" y="140" text-anchor="middle" font-family="system-ui,sans-serif" font-size="10" fill="#6366F1">POST /scan-payment</text>

  {/* Step 4: API validates */}
  <rect x="382" y="163" width="96" height="24" rx="5" fill="#FFF7ED" stroke="#F97316" stroke-width="0.5"/>
  <text x="430" y="179" text-anchor="middle" font-family="system-ui,sans-serif" font-size="10" fill="#9A3412">Validate counter</text>

  {/* Step 5: API → Wallet */}
  <line x1="430" y1="206" x2="608" y2="206" stroke="#F97316" stroke-width="1.5" marker-end="url(#arr4)"/>
  <text x="519" y="198" text-anchor="middle" font-family="system-ui,sans-serif" font-size="10" fill="#F97316">payment.approval_requested</text>

  {/* Step 6: Wallet → Customer push */}
  <line x1="615" y1="236" x2="74" y2="236" stroke="#22C55E" stroke-width="1.5" stroke-dasharray="5 3" marker-end="url(#arr4)"/>
  <text x="344" y="228" text-anchor="middle" font-family="system-ui,sans-serif" font-size="10" fill="#22C55E">Push notification</text>

  {/* Step 7: Customer approves */}
  <rect x="16" y="251" width="98" height="24" rx="5" fill="#F0FDF4" stroke="#22C55E" stroke-width="0.5"/>
  <text x="65" y="267" text-anchor="middle" font-family="system-ui,sans-serif" font-size="10" fill="#166534">Tap Approve</text>

  {/* Step 8: Customer → Wallet */}
  <line x1="65" y1="294" x2="608" y2="294" stroke="#22C55E" stroke-width="1.5" marker-end="url(#arr4)"/>
  <text x="344" y="286" text-anchor="middle" font-family="system-ui,sans-serif" font-size="10" fill="#22C55E">payment.completed</text>

  {/* Step 9: Webhook → POS */}
  <line x1="430" y1="324" x2="263" y2="324" stroke="#F97316" stroke-width="1.5" stroke-dasharray="5 3" marker-end="url(#arr4)"/>
  <text x="346" y="316" text-anchor="middle" font-family="system-ui,sans-serif" font-size="10" fill="#F97316">Webhook: payment.completed</text>

  {/* Step 10: Success */}
  <rect x="206" y="340" width="98" height="24" rx="5" fill="#F0FDF4" stroke="#22C55E" stroke-width="0.5"/>
  <text x="255" y="356" text-anchor="middle" font-family="system-ui,sans-serif" font-size="10" fill="#166534">Show success</text>
</svg>
</p>

### Prerequisites — Register a Counter (POS Terminal)

Each physical terminal should be registered once. This enables per-terminal reporting and ties scan-payment transactions to a specific terminal.

**`POST /merchant/api/v1/counters`**

```json
{
  "name": "Till 1",
  "location": "Main entrance"
}
```

**Response:**
```json
{
  "success": true,
  "counter": {
    "id": "ctr_xxxx",
    "merchant_id": "a1b2c3d4-...",
    "name": "Till 1",
    "location": "Main entrance",
    "status": "active",
    "webhook_secret": "whsec_xxxxxxxx"
  }
}
```

> **Save `webhook_secret`** — you will use it to verify incoming webhooks for this terminal. Treat it like a password.

---

### Initiate a Scan Payment

**`POST /merchant/api/v1/scan-payment`**

```json
{
  "scan_token":    "{value decoded from customer's wallet QR}",
  "counter_id":   "ctr_xxxx",
  "amount":       49.50,
  "currency":     "USD",
  "description":  "Lunch combo",
  "expires_in":   120,
  "reference":    "ORDER-2087",
  "callback_url": "https://your-backend.example.com/webhooks/fex"
}
```

**Request fields:**

| Field | Type | Required | Description |
|-------|------|:--------:|-------------|
| `scan_token` | string | ✅ | Value decoded from the customer's wallet QR |
| `counter_id` | string | ✅ | ID of the scanning terminal |
| `amount` | float | ✅ | Charge amount |
| `currency` | string | — | Default: `USD` |
| `expires_in` | integer | — | Seconds for customer to approve. Default: `300` |
| `description` | string | — | Shown to customer in push notification |
| `reference` | string | — | Your internal order or invoice ID |
| `callback_url` | string | — | URL to receive the `payment.completed` webhook |

**Response `201 Created`:**
```json
{
  "success": true,
  "payment_intent": {
    "id": "pi_yyyy",
    "status": "pending_approval",
    "amount": 49.50,
    "expires_at": "2025-01-01T12:37:00Z"
  }
}
```

The customer receives a push notification immediately. Their approval fires a `payment.completed` webhook to your `callback_url`.

---

## 6. Webhooks

The platform sends a signed HTTP POST to your `callback_url` whenever a payment changes state.

### Payload

```json
{
  "event": "payment.completed",
  "payment_intent_id": "pi_xxxx",
  "merchant_id": "a1b2c3d4-...",
  "amount": 29.99,
  "currency": "USD",
  "status": "completed",
  "completed_at": "2025-01-01T12:31:55Z",
  "reference": "ORDER-1042",
  "metadata": { "table": "5", "cashier": "Maria" }
}
```

### Webhook Events

| Event | Trigger |
|-------|---------|
| `payment.completed` | Customer payment confirmed |
| `payment.expired` | QR or approval window elapsed without payment |
| `payment.refunded` | Merchant issued a refund |

---

### Verifying the Signature

Every webhook delivery includes an `X-Webhook-Signature` header — an **HMAC-SHA256 hex digest** of the raw request body, signed with your counter's `webhook_secret`.

> **Always use constant-time comparison** (`hmac.compare_digest` / `timingSafeEqual`) to prevent timing attacks. Never use a simple string equality check.

**Python:**
```python
import hmac, hashlib

def verify_webhook(body: bytes, signature: str, secret: str) -> bool:
    expected = hmac.new(
        secret.encode(),
        body,
        hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(expected, signature)
```

**Node.js:**
```javascript
const crypto = require('crypto');

function verifyWebhook(body, signature, secret) {
  const expected = crypto
    .createHmac('sha256', secret)
    .update(body)
    .digest('hex');
  return crypto.timingSafeEqual(
    Buffer.from(expected),
    Buffer.from(signature)
  );
}
```

### Acknowledging Delivery

Respond with **HTTP 200** to acknowledge receipt. Deliveries returning a non-2xx status are retried **up to 5 times** with exponential backoff.

---

## 7. Cancellations & Refunds

### Cancel a Pending Intent

**`POST /merchant/api/v1/payment-intents/{id}/cancel`**

Only intents in `pending` status can be cancelled. Returns `200` on success.

---

### Issue a Refund

**`POST /merchant/api/v1/payment-intents/{id}/refund`**

Only intents in `completed` status can be refunded. The full amount is returned to the customer's wallet.

**Response:**
```json
{
  "success": true,
  "refund_initiated": true,
  "message": "Refund initiated successfully. Funds will be returned to the customer's wallet.",
  "payment_intent": {
    "id": "pi_xxxx",
    "status": "refunded",
    "refund_status": "initiated",
    "refund_ledger_tx_id": "ltx_zzzz"
  }
}
```

---

## 8. Transaction History

**`GET /merchant/api/v1/transactions?page=1&limit=20`**

Returns paginated ledger transactions for your merchant wallet.

**Response:**
```json
{
  "success": true,
  "merchant_id": "a1b2c3d4-...",
  "transactions": [
    {
      "id": "ltx_xxxx",
      "type": "payment",
      "amount": 29.99,
      "currency": "USD",
      "direction": "credit",
      "reference": "ORDER-1042",
      "created_at": "2025-01-01T12:31:55Z"
    }
  ],
  "page": 1,
  "limit": 20,
  "total": 142,
  "total_pages": 8
}
```

---

## 9. Counter Management

| Action | Method | Endpoint |
|--------|--------|----------|
| Create terminal | `POST` | `/merchant/api/v1/counters` |
| List terminals | `GET` | `/merchant/api/v1/counters` |
| Get terminal | `GET` | `/merchant/api/v1/counters/{id}` |
| Update terminal | `PUT` | `/merchant/api/v1/counters/{id}` |
| Deactivate terminal | `DELETE` | `/merchant/api/v1/counters/{id}` |
| Rotate webhook secret | `POST` | `/merchant/api/v1/counters/{id}/rotate-secret` |

> **Rotate the webhook secret** if it is ever exposed or compromised. All subsequent webhooks will use the new secret immediately.

---

## 10. Error Reference

All error responses follow this shape:

```json
{
  "success": false,
  "error": "human-readable message"
}
```

| HTTP Status | Meaning | Common cause |
|-------------|---------|--------------|
| `400` | Bad request | Missing or invalid field in request body |
| `401` | Unauthorized | Missing, expired, or malformed access token |
| `403` | Forbidden | Token does not have permission for this action |
| `404` | Not found | Resource ID does not exist |
| `409` | Conflict | e.g. refund already issued for this intent |
| `422` | Unprocessable | e.g. no customer wallet on file to refund |
| `500` | Platform error | Transient server fault — retry with exponential backoff |

> **On `5xx` errors:** implement retry logic with exponential backoff and jitter. Do not retry `4xx` errors without first fixing the request.

---

## 11. Quick-Start Checklist

Work through this list in order before going live.

**Credentials & Setup**
- [ ] Received `merchant_id`, `client_id`, and `client_secret` from platform admin
- [ ] Stored `client_secret` in a secrets manager (not in source code or environment variables)
- [ ] Tested token generation via the client credentials flow

**Terminal Registration**
- [ ] Created at least one counter (POS terminal) via `POST /merchant/api/v1/counters`
- [ ] Saved the counter's `webhook_secret` in your secrets manager

**Backend Integration**
- [ ] Implemented token refresh logic (refresh at least 30 s before expiry)
- [ ] Implemented webhook endpoint with HMAC-SHA256 signature verification
- [ ] Implemented retry/backoff logic for `5xx` responses

**End-to-End Testing**
- [ ] Tested Standard QR flow end-to-end in sandbox
- [ ] Tested Reverse QR flow end-to-end in sandbox
- [ ] Verified webhook delivery and signature validation

**Go Live**
- [ ] Contact platform admin to activate your live environment

---

*For sandbox access, base URLs, or credential issues, contact your platform administrator.*