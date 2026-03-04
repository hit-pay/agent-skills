---
name: hitpay-qr-checkout
description: Generate a visual QR payment page artifact using HitPay MCP tools. Use when user says "create a QR payment page", "embedded QR checkout", "QR payment page for PayNow", "QRPH payment page", "QR checkout artifact", "build a QR payment UI", "show QR code to collect payment", or "QR page for [country] customer".
license: MIT
metadata:
  author: hitpay
  version: "1.0.0"
---

# HitPay QR Checkout — Payment Page Generator

Generate a self-contained React artifact that displays a branded QR payment page. Orchestrates `create_embedded_qr` → extracts response data → injects into a React template.

> **Collecting QR payments without a visual page?** Use the **hitpay-qr-payments** skill instead.
> **Writing backend integration code?** Use the **hitpay** skill instead.

## When to Apply

- User wants a **visual QR payment page** (React artifact, not just raw QR data)
- User says "create a QR payment page", "QR checkout page", "show a QR to collect payment"
- User mentions a specific payment method + amount and wants a rendered UI

## Step 1: Parse Natural Language

Map the user's request to `create_embedded_qr` parameters:

### Payment Method Mapping

| User Says | `payment_methods` | Default Currency |
|-----------|-------------------|-----------------|
| PayNow | `paynow_online` | SGD |
| QRPH / InstaPay | `qrph_netbank` | PHP |
| GCash | `gcash_qr` | PHP |
| PromptPay | `opn_prompt_pay` | THB |
| TrueMoney | `opn_true_money_qr` | THB |
| QRIS | `ifpay_qris` | IDR |
| UPI | `upi_qr` | SGD |
| ShopeePay | `shopee_pay` | SGD |
| GrabPay | `grabpay_direct` | SGD |
| Touch 'n Go / TNG | `touch_n_go` | MYR |

### Customer Country Mapping

| User Says | Recommended Method | Code |
|-----------|--------------------|------|
| Filipino / Philippine customer | QR Ph (InstaPay) | `qrph_netbank` |
| Thai customer | PromptPay | `opn_prompt_pay` |
| Indonesian customer | QRIS | `ifpay_qris` |
| Malaysian customer | Touch 'n Go | `touch_n_go` |
| Singapore customer | PayNow | `paynow_online` |

## Step 2: Detect Borderless (Cross-Border)

If the customer's country differs from the merchant's currency country:

1. Set `currency` to the **merchant's** currency (e.g., `SGD` for a Singapore merchant)
2. Use the **borderless method code** from the table above (e.g., `gcash_qr` not `gcash`)
3. The response will include `borderless_fx` with FX conversion details

**Example:** Singapore merchant + Filipino customer → `currency: "SGD"`, `payment_methods: ["qrph_netbank"]`

## Step 3: Call MCP Tool

```
Tool: create_embedded_qr
Parameters:
  amount: "<amount>"
  currency: "<resolved currency>"
  payment_methods: ["<resolved method>"]
  purpose: "<optional description>"
  reference_number: "<optional reference>"
```

### Expected Response Shape

```json
{
  "id": "abc-123",
  "amount": "SGD 100.00",
  "status": "pending",
  "payment_methods": ["qrph_netbank"],
  "checkout_url": "https://securecheckout.hit-pay.com/...",
  "qr_code_data": {
    "qr_code": "<raw QR payload string or base64>",
    "qr_code_expiry": "2025-01-01T12:15:00Z"
  },
  "borderless_fx": {
    "customer_pays": "PHP 4,557.00",
    "merchant_receives": "SGD 100.00",
    "fx_rate": "45.57",
    "display_rate": "1 SGD = 45.57 PHP",
    "fee_note": "1.5% cross-border processing fee applies"
  }
}
```

> `borderless_fx` is only present for cross-border payments. For domestic payments it will be null/absent.

## Step 4: Generate React Artifact

Read the template from `references/react-template.md` and inject the API response values:

1. Replace `PAYMENT_DATA` constant values with actual response data
2. Set `borderless` to `null` for domestic payments, or populate with `borderless_fx` values
3. The template is fully self-contained — inline styles, CDN QR library, no external dependencies

### Key Template Features

- QR code rendered via `qrcode.js` CDN (injected in `useEffect`) with base64 image fallback
- Countdown timer using `qr_code_expiry` (or 15-minute default), turns red under 2 minutes
- Borderless FX info card (only renders when borderless data is present)
- "Open in browser" fallback link to `checkout_url`
- HitPay brand colors throughout

## Error Handling

| Situation | Action |
|-----------|--------|
| Missing amount | Ask user: "What amount should I create the QR for?" |
| Missing payment method | Ask user: "Which payment method? (PayNow, QRPH, GCash, etc.)" |
| Ambiguous method (e.g., "QR payment" with no method) | Default to PayNow (SGD) or ask if multi-market |
| Provider not enabled | Tell user: "The [method] provider is not enabled on your account. Enable it in the HitPay dashboard." |
| API error | Surface the error message and suggestion from the MCP response |

## Brand Colors

| Token | Hex | Usage |
|-------|-----|-------|
| Deep Blue | `#002771` | Dark gradient background |
| Logo Blue | `#0E2859` | Card background |
| Action Blue | `#2465DE` | Accents, links, badges |
| White | `#FFFFFF` | QR container, text on dark |
| Success Green | `#4DAB80` | Merchant receives amount |
| Text Primary | `#03102F` | Primary text on light backgrounds |
| Text Secondary | `#61667C` | Secondary/muted text |

## Skill Boundaries

- **This skill:** Orchestrates MCP call → React artifact (visual QR page)
- **hitpay-qr-payments:** Full provider-currency mapping, pre-flight validation, decision flowcharts (operational, no UI)
- **hitpay:** Backend integration code — API routes, webhook handlers, React components for app embedding
