---
name: hitpay-qr-payments
description: Operational guide for AI agents collecting QR payments via HitPay MCP tools. NOT for writing integration code — use the "hitpay" skill for that. Use when user says "collect PayNow payment", "QR for Indonesia", "accept GrabPay", "which payment methods in Malaysia", "generate QR code for payment", or "HitPay QR payment".
license: MIT
metadata:
  author: hitpay
  version: "1.0.0"
---

# HitPay QR Payments — MCP Operations Guide

Operational knowledge for AI agents using HitPay MCP tools (`create_embedded_qr`, `create_payment_request`, `get_account_status`) to collect QR-based payments across Southeast Asia, Australia, India, Korea, and Vietnam.

> **Writing code?** For API routes, React components, and webhook handlers, use the **hitpay** skill instead.

## When to Apply

- User asks to collect a payment via QR code (PayNow, GrabPay, QRIS, etc.)
- User asks which payment methods are available for a specific country
- User wants to generate a QR code for in-person or remote payment collection
- Agent needs to determine the correct `payment_methods` + `currency` combination for `create_embedded_qr`
- User asks about cross-currency settlement or FX behavior

## Step 1: Check Enabled Providers

Before generating any QR code, call `get_account_status` to confirm which providers are enabled.

```
Tool: get_account_status
```

Look at `payment_providers.completed[]` — only providers listed there can process payments. If a provider is in `pending[]`, inform the user it is not yet active.

**Key provider-to-method mapping (quick reference):**

| Provider | Payment Method(s) | Currency | Market |
|----------|-------------------|----------|--------|
| `dbs_sg` / `dbs_max_sg` | PayNow | SGD | Singapore |
| `grabpay` | GrabPay | SGD, MYR | SG, MY |
| `shopee_pay` | ShopeePay | SGD, MYR, PHP | SG, MY, PH |
| `opn` | PromptPay, TrueMoney | THB | Thailand |
| `qrph_netbank` | QR Ph (InstaPay) | PHP | Philippines |
| `touch_n_go` | Touch 'n Go eWallet | MYR | Malaysia |
| `ifpay` / `qfpay` / `wechat` | WeChat Pay | CNY→SGD | Cross-border |
| `ifpay` / `qfpay` | Alipay+ | CNY→SGD | Cross-border |
| `upi` | UPI | INR→SGD | India |
| `vietqr_payme` | VietQR | VND | Vietnam |
| `zalopay` | ZaloPay | VND | Vietnam |
| `monoova` | PayTo | AUD | Australia |
| `nhn_kcp` | Korean methods | KRW | Korea |

> Full country-by-country tables with settlement rules: see `references/provider-currency-map.md`

## Step 2: Map Country to Payment Methods

When the user specifies a country, use this lookup:

| Country | Recommended QR Methods | Currency |
|---------|----------------------|----------|
| **Singapore** | PayNow, GrabPay, ShopeePay | SGD |
| **Malaysia** | GrabPay, ShopeePay, Touch 'n Go | MYR |
| **Philippines** | QR Ph (InstaPay), ShopeePay, GCash | PHP |
| **Thailand** | PromptPay, TrueMoney | THB |
| **Vietnam** | VietQR, ZaloPay | VND |
| **Indonesia** | QRIS (via `qfpay`) | IDR |
| **India** | UPI | INR |
| **Australia** | PayTo | AUD |
| **Korea** | KCP methods | KRW |
| **Cross-border (Chinese tourists)** | WeChat Pay, Alipay+ | CNY→SGD |

If the user doesn't specify a country, ask. Never guess — the wrong currency will cause the API call to fail.

## Step 3: Validate Before Generating

Run this pre-flight checklist:

1. **Provider enabled?** — Check `get_account_status` result for the required provider
2. **Currency matches?** — Each method requires a specific currency (e.g., PayNow = SGD only)
3. **Amount reasonable?** — Minimum is typically 1.00 in local currency
4. **Method string correct?** — Use the exact `payment_methods` value (see table below)

### Payment Method Strings

These are the exact values to pass in the `payment_methods` parameter:

| Display Name | API Value | Notes |
|-------------|-----------|-------|
| PayNow | `paynow_online` | SGD only |
| GrabPay | `grabpay` | SGD or MYR |
| ShopeePay | `shopee_pay` | SGD, MYR, or PHP |
| Touch 'n Go | `touch_n_go` | MYR only |
| FPX | `fpx` | MYR only, bank transfer not QR |
| PromptPay | `promptpay` | THB only |
| TrueMoney | `truemoney` | THB only |
| VietQR | `vietqr` | VND only |
| ZaloPay | `zalopay` | VND only |
| WeChat Pay | `wechat` | Settles in SGD |
| Alipay+ | `alipay` | Settles in SGD |
| UPI | `upi` | Settles in SGD |
| QR Ph | `qrph` | PHP only |
| GCash | `gcash` | PHP only |
| QRIS | `qris` | IDR only |
| PayTo | `payto` | AUD only |

## Step 4: Generate the QR Code

Use `create_embedded_qr` for all QR payment collection:

```
Tool: create_embedded_qr
Parameters:
  amount: "10.00"
  currency: "SGD"
  payment_methods: ["paynow_online"]
  name: "Customer Name"           # optional
  email: "customer@example.com"   # optional
  purpose: "Order #1234"          # optional
  reference_number: "ORD-1234"    # optional
```

The response includes a `qr_code_data` field (base64 PNG) and a `url` for the checkout page.

### When to Use Which Tool

| Scenario | Tool | Why |
|----------|------|-----|
| Collect a one-time QR payment | `create_embedded_qr` | Returns QR image directly |
| Collect card + QR payment | `create_payment_request` | Returns checkout URL with method selector |
| Permanent in-store QR | `create_static_qr` | Rare — POS terminals only |

## Pitfalls and Edge Cases

### Cross-Currency Settlement

These methods accept payment in the customer's local currency but **settle in SGD**:

| Method | Customer Pays | You Receive | FX Applied |
|--------|--------------|-------------|------------|
| WeChat Pay | CNY | SGD | At transaction time |
| Alipay+ | CNY | SGD | At transaction time |
| UPI | INR | SGD | At transaction time |

When using these, set `currency: "SGD"` in the API call — the provider handles the customer-side conversion.

### Known Unavailable Methods

These are commonly requested but **not currently available** as QR methods on HitPay:

- **DuitNow** (Malaysia) — Use GrabPay, ShopeePay, or Touch 'n Go instead
- **DANA** (Indonesia) — Use QRIS instead (covers DANA wallets)
- **OVO** (Indonesia) — Use QRIS instead
- **LINE Pay** (Thailand) — Use PromptPay or TrueMoney instead
- **Momo** (Vietnam) — Use VietQR or ZaloPay instead

### Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| "Payment method not available" | Provider not enabled | Check `get_account_status`, enable in dashboard |
| "Invalid currency" | Currency doesn't match method | Use the correct currency from the table above |
| "Amount too low" | Below minimum threshold | Minimum is usually 1.00 in local currency |
| Provider in `pending[]` | KYC not complete for that provider | Complete verification in HitPay dashboard |

### Multiple Methods in One Request

You can pass multiple QR methods in a single request to let the customer choose:

```
payment_methods: ["paynow_online", "grabpay", "shopee_pay"]
currency: "SGD"
```

All methods in the array **must support the same currency**. You cannot mix SGD and MYR methods.

## Quick Decision Flowchart

```
User wants QR payment
  │
  ├─ Know the country? ──No──→ Ask user which country
  │
  Yes
  │
  ├─ Look up methods for that country (Step 2 table)
  │
  ├─ Call get_account_status
  │   │
  │   ├─ Provider in completed[]? ──No──→ "Provider not enabled. Enable in dashboard."
  │   │
  │   Yes
  │   │
  │   ├─ Call create_embedded_qr with correct method + currency
  │   │
  │   └─ Return QR code to user
  │
  └─ Done
```
