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
- Webhook handling with signature verification
- Refunds API (full and partial)

### hitpay-qr-payments

Operational guide for AI agents collecting QR payments via HitPay MCP tools. NOT for writing integration code.

**Triggers:**
- "collect PayNow payment"
- "QR for Indonesia"
- "accept GrabPay"
- "which payment methods in Malaysia"
- "generate QR code for payment"
- "HitPay QR payment"

**Capabilities:**
- Provider-currency mapping for 10+ markets (SG, MY, PH, TH, VN, ID, IN, AU, KR, cross-border)
- Pre-flight account validation via `get_account_status`
- Correct `payment_methods` + `currency` parameter selection
- Cross-currency settlement rules (WeChat, Alipay, UPI)
- Known gaps and alternative recommendations

## Skill Structure

```
skills/hitpay/
├── SKILL.md                     # Main skill instructions
├── references/
│   ├── payment-request-api.md   # Full API reference
│   ├── webhook-events.md        # Webhook handling guide
│   └── refunds.md               # Refunds API reference
└── scripts/
    └── verify-webhook.sh        # Code samples for webhook verification
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
