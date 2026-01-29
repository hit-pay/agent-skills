# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, Cursor, Copilot, etc.) when working with code in this repository.

## Repository Overview

HitPay Agent Skills - AI agent skills for integrating HitPay payment gateway into applications.

## Available Skills

### hitpay

Integrate HitPay payment gateway for online payments in Next.js and JavaScript/TypeScript applications.

**Triggers:**
- "Add HitPay"
- "HitPay checkout"
- "HitPay payments"
- "HitPay webhook"
- "HitPay QR code"
- "PayNow integration"

**Capabilities:**
- Payment request creation (redirect and embedded QR flows)
- Frontend options: redirect checkout, embedded QR, payment method selector
- Webhook handling with signature verification (vendor + event webhooks)
- Refunds API (full and partial)
- Common errors troubleshooting
- Sandbox testing guide

---

### hitpay-php

Integrate HitPay payment gateway using PHP and cURL. No SDK dependencies required.

**Triggers:**
- "HitPay PHP"
- "HitPay Laravel"
- "HitPay cURL"
- "PHP payment integration"
- "PHP webhook validation"

**Capabilities:**
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

**Triggers:**
- "HitPay drop-in"
- "HitPay popup"
- "HitPay modal"
- "embedded checkout"
- "HitPay iframe"

**Capabilities:**
- Drop-in UI setup guide
- React integration component
- Vue integration component
- Vanilla JavaScript implementation
- Configuration options
- Common issues (methods not showing, DOKU limitations)
- Security best practices

---

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

## Installation

```bash
npx skills add hit-pay/agent-skills
```

## Creating/Modifying Skills

### Directory Structure

```
skills/
  {skill-name}/           # kebab-case directory name
    SKILL.md              # Required: skill definition
    scripts/              # Executable scripts
    references/           # Supporting documentation
  {skill-name}.zip        # Packaged for distribution
```

### SKILL.md Format

```markdown
---
name: {skill-name}
description: {One sentence describing when to use this skill. Include trigger phrases.}
license: MIT
metadata:
  author: hitpay
  version: "1.0.0"
---

# {Skill Title}

{Brief description and instructions}
```

### Best Practices

- **Keep SKILL.md under 500 lines** — put detailed reference material in separate files
- **Write specific descriptions** — helps the agent know exactly when to activate the skill
- **Use progressive disclosure** — reference supporting files that get read only when needed

### Creating the Zip Package

```bash
cd skills
zip -r {skill-name}.zip {skill-name}/
```
