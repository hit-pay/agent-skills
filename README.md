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
- Webhook handling with signature verification
- Refunds API (full and partial)
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

## Installation

```bash
npx skills add hit-pay/agent-skills
```

## Usage

Once installed, the skill is automatically available. Your AI coding agent will use it when you mention HitPay integration tasks.

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

## Skill Structure

```
skills/hitpay/
├── SKILL.md                 # Main skill instructions
├── references/
│   ├── payment-request-api.md   # API reference
│   ├── webhook-events.md        # Webhook handling
│   └── refunds.md               # Refunds API
└── scripts/
    └── verify-webhook.sh        # Code samples
```

## Resources

- [HitPay API Docs](https://docs.hitpayapp.com)
- [HitPay Dashboard](https://dashboard.hit-pay.com)
- [Sandbox Dashboard](https://dashboard.sandbox.hit-pay.com)

## License

MIT
