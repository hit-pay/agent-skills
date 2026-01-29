# HitPay Agent Skills

AI agent skills for integrating HitPay payment gateway into your applications.

## Available Skills

### hitpay

Integrate HitPay payment gateway for online payments in Next.js and JavaScript/TypeScript applications.

**Use when:**
- Integrating HitPay payment gateway
- Creating payment checkout flows
- Implementing QR code payments (PayNow, GrabPay, ShopeePay)
- Handling payment webhooks
- Processing refunds

**Features:**
- Payment request creation (redirect and embedded QR flows)
- Frontend options: redirect checkout, embedded QR, payment method selector
- Webhook handling with signature verification (vendor + event webhooks)
- Refunds API (full and partial)
- Common errors troubleshooting
- Sandbox testing guide
- Next.js and Express.js code examples

**Supported Payment Methods:**
- Cards (Visa, Mastercard, Amex)
- PayNow (Singapore)
- GrabPay
- ShopeePay
- WeChat Pay
- Alipay
- FPX (Malaysia)
- PromptPay (Thailand)
- And more...

---

### hitpay-php

Integrate HitPay payment gateway using PHP and cURL. No SDK dependencies required.

**Use when:**
- Building PHP applications with HitPay
- Creating Laravel payment integrations
- Handling webhooks in PHP
- Need pure cURL implementation without SDK

**Features:**
- Pure cURL implementation (works without SDK)
- Payment request creation with payment_methods array
- Vendor webhook HMAC-SHA256 validation
- Laravel service class pattern
- Laravel webhook controller with CSRF exception
- Payment status checking
- Refund API
- Common PHP errors and solutions

---

### hitpay-dropin

Integrate HitPay Drop-in UI for embedded payment checkout modals.

**Use when:**
- Embedding HitPay checkout in a modal/popup
- Integrating payment UI without full page redirect
- Building React/Vue payment components
- Customizing drop-in UI behavior

**Features:**
- Drop-in UI setup guide
- React integration component
- Vue integration component
- Vanilla JavaScript implementation
- Configuration options and callbacks
- Common issues troubleshooting
- Security best practices

## Installation

```bash
npx skills add hit-pay/agent-skills
```

## Usage

Once installed, skills are automatically available. Your AI coding agent will use the appropriate skill based on your request.

**Examples:**
```
Add HitPay payment integration to my Next.js app
```
```
Create a PayNow QR code checkout
```
```
Handle HitPay webhooks
```
```
Process a refund with HitPay
```
```
Add HitPay to my Laravel app
```
```
HitPay PHP webhook validation
```
```
Add HitPay drop-in popup to my React app
```
```
Embed HitPay checkout modal
```

## Skill Structure

```
skills/
├── hitpay/
│   ├── SKILL.md                     # Main skill instructions (Next.js/TS)
│   └── references/
│       ├── payment-request-api.md   # Full API reference
│       ├── webhook-events.md        # Webhook handling (vendor + event)
│       ├── refunds.md               # Refunds API reference
│       ├── common-errors.md         # Troubleshooting guide
│       └── sandbox-testing.md       # Testing guide
│
├── hitpay-php/
│   └── SKILL.md                     # PHP/Laravel integration
│
└── hitpay-dropin/
    └── SKILL.md                     # Drop-in UI guide
```

## Resources

- [HitPay API Docs](https://docs.hitpayapp.com)
- [HitPay Dashboard](https://dashboard.hit-pay.com)
- [Sandbox Dashboard](https://dashboard.sandbox.hit-pay.com)

## License

MIT
