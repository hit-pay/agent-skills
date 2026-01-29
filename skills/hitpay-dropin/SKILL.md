---
name: hitpay-dropin
description: Integrate HitPay Drop-in UI for embedded payment checkout. Use when user says "HitPay drop-in", "HitPay popup", "embedded checkout", "HitPay modal", or needs to display HitPay payment UI in their application.
license: MIT
metadata:
  author: hitpay
  version: "1.0.0"
---

# HitPay Drop-in UI Integration

Embed HitPay's payment checkout as a modal/popup in your application. Works with React, Vue, and vanilla JavaScript.

## When to Apply

Reference this skill when:
- Embedding HitPay checkout in a modal/popup
- Integrating payment UI without full page redirect
- Building React/Vue payment components
- Customizing drop-in UI behavior
- Troubleshooting drop-in UI issues

## Overview

The Drop-in UI allows customers to complete payment without leaving your site. It opens a secure modal/iframe containing HitPay's checkout.

### Flow

1. Backend creates payment request via API
2. Frontend initializes Drop-in with payment request ID
3. Customer completes payment in modal
4. Modal closes, frontend receives success callback
5. Backend confirms via webhook

---

## Setup

### Include HitPay Script

```html
<!-- Sandbox -->
<script src="https://sandbox.hit-pay.com/hitpay.js"></script>

<!-- Production -->
<script src="https://www.hit-pay.com/hitpay.js"></script>
```

### Environment URLs

| Environment | Script URL | API Domain |
|-------------|------------|------------|
| Sandbox | sandbox.hit-pay.com/hitpay.js | sandbox.hit-pay.com |
| Production | www.hit-pay.com/hitpay.js | hit-pay.com |

---

## Vanilla JavaScript

### Basic Implementation

```html
<!DOCTYPE html>
<html>
<head>
    <title>Payment</title>
    <!-- Include HitPay script -->
    <script src="https://sandbox.hit-pay.com/hitpay.js"></script>
</head>
<body>
    <button id="pay-button">Pay Now</button>

    <script>
        document.getElementById('pay-button').addEventListener('click', async () => {
            // 1. Create payment request via your backend
            const response = await fetch('/api/create-payment', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ amount: 100, currency: 'SGD' })
            });
            const { paymentRequestId, businessSlug } = await response.json();

            // 2. Build checkout URL
            const checkoutUrl = `https://securecheckout.sandbox.hit-pay.com/payment-request/@${businessSlug}/${paymentRequestId}/checkout`;

            // 3. Initialize Drop-in
            window.HitPay.init(checkoutUrl, {
                domain: 'sandbox.hit-pay.com',
                apiDomain: 'sandbox.hit-pay.com'
            }, {
                onClose: () => {
                    console.log('Payment modal closed');
                },
                onSuccess: (data) => {
                    console.log('Payment successful', data);
                    // Redirect to success page or show confirmation
                    window.location.href = '/payment/success?id=' + paymentRequestId;
                },
                onError: (error) => {
                    console.error('Payment error', error);
                    alert('Payment failed. Please try again.');
                }
            });
        });
    </script>
</body>
</html>
```

### Using Toggle Method

```javascript
// Alternative: Use toggle method
window.HitPay.toggle({
    paymentRequest: paymentRequestId,
    amount: '100.00',
    currency: 'SGD'
});
```

---

## React Integration

### Payment Button Component

```tsx
// components/HitPayButton.tsx
import { useCallback, useEffect } from 'react';

interface HitPayButtonProps {
    amount: number;
    currency: string;
    orderId: string;
    onSuccess?: (data: any) => void;
    onError?: (error: any) => void;
}

declare global {
    interface Window {
        HitPay: {
            init: (url: string, config: any, callbacks: any) => void;
            toggle: (options: any) => void;
        };
    }
}

export function HitPayButton({ amount, currency, orderId, onSuccess, onError }: HitPayButtonProps) {
    // Load HitPay script
    useEffect(() => {
        const script = document.createElement('script');
        script.src = process.env.NODE_ENV === 'production'
            ? 'https://www.hit-pay.com/hitpay.js'
            : 'https://sandbox.hit-pay.com/hitpay.js';
        script.async = true;
        document.body.appendChild(script);

        return () => {
            document.body.removeChild(script);
        };
    }, []);

    const handlePayment = useCallback(async () => {
        try {
            // Create payment request via backend
            const response = await fetch('/api/payments/create', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ amount, currency, orderId })
            });

            const { paymentRequestId, checkoutUrl } = await response.json();

            // Initialize Drop-in
            const domain = process.env.NODE_ENV === 'production'
                ? 'hit-pay.com'
                : 'sandbox.hit-pay.com';

            window.HitPay.init(checkoutUrl, {
                domain,
                apiDomain: domain
            }, {
                onClose: () => {
                    console.log('Modal closed');
                },
                onSuccess: (data: any) => {
                    console.log('Payment success', data);
                    onSuccess?.(data);
                },
                onError: (error: any) => {
                    console.error('Payment error', error);
                    onError?.(error);
                }
            });
        } catch (error) {
            console.error('Failed to create payment', error);
            onError?.(error);
        }
    }, [amount, currency, orderId, onSuccess, onError]);

    return (
        <button onClick={handlePayment} className="pay-button">
            Pay ${amount} {currency}
        </button>
    );
}
```

### Usage in Page

```tsx
// pages/checkout.tsx
import { HitPayButton } from '@/components/HitPayButton';
import { useRouter } from 'next/router';

export default function CheckoutPage() {
    const router = useRouter();

    return (
        <div>
            <h1>Checkout</h1>
            <HitPayButton
                amount={100}
                currency="SGD"
                orderId="ORDER-123"
                onSuccess={(data) => {
                    router.push('/payment/success');
                }}
                onError={(error) => {
                    alert('Payment failed');
                }}
            />
        </div>
    );
}
```

### Next.js API Route

```ts
// app/api/payments/create/route.ts
import { NextResponse } from 'next/server';

export async function POST(request: Request) {
    const { amount, currency, orderId } = await request.json();

    const response = await fetch('https://api.sandbox.hit-pay.com/v1/payment-requests', {
        method: 'POST',
        headers: {
            'X-BUSINESS-API-KEY': process.env.HITPAY_API_KEY!,
            'Content-Type': 'application/json',
        },
        body: JSON.stringify({
            amount,
            currency,
            reference_number: orderId,
            webhook: `${process.env.NEXT_PUBLIC_APP_URL}/api/webhooks/hitpay`,
        }),
    });

    const data = await response.json();

    // Return checkout URL for drop-in
    return NextResponse.json({
        paymentRequestId: data.id,
        checkoutUrl: data.url
    });
}
```

---

## Vue Integration

### Payment Component

```vue
<!-- components/HitPayPayment.vue -->
<template>
    <button @click="initiatePayment" :disabled="loading">
        {{ loading ? 'Processing...' : `Pay $${amount} ${currency}` }}
    </button>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue';

interface Props {
    amount: number;
    currency: string;
    orderId: string;
}

const props = defineProps<Props>();
const emit = defineEmits(['success', 'error', 'close']);

const loading = ref(false);
let scriptElement: HTMLScriptElement | null = null;

// Load HitPay script
onMounted(() => {
    scriptElement = document.createElement('script');
    scriptElement.src = import.meta.env.PROD
        ? 'https://www.hit-pay.com/hitpay.js'
        : 'https://sandbox.hit-pay.com/hitpay.js';
    scriptElement.async = true;
    document.body.appendChild(scriptElement);
});

onUnmounted(() => {
    if (scriptElement) {
        document.body.removeChild(scriptElement);
    }
});

async function initiatePayment() {
    loading.value = true;

    try {
        // Create payment request via backend
        const response = await fetch('/api/payments/create', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
                amount: props.amount,
                currency: props.currency,
                orderId: props.orderId
            })
        });

        const { checkoutUrl } = await response.json();

        const domain = import.meta.env.PROD
            ? 'hit-pay.com'
            : 'sandbox.hit-pay.com';

        // Initialize Drop-in
        (window as any).HitPay.init(checkoutUrl, {
            domain,
            apiDomain: domain
        }, {
            onClose: () => {
                loading.value = false;
                emit('close');
            },
            onSuccess: (data: any) => {
                loading.value = false;
                emit('success', data);
            },
            onError: (error: any) => {
                loading.value = false;
                emit('error', error);
            }
        });
    } catch (error) {
        loading.value = false;
        emit('error', error);
    }
}
</script>
```

### Usage

```vue
<template>
    <HitPayPayment
        :amount="100"
        currency="SGD"
        orderId="ORDER-123"
        @success="handleSuccess"
        @error="handleError"
    />
</template>

<script setup>
import HitPayPayment from '@/components/HitPayPayment.vue';
import { useRouter } from 'vue-router';

const router = useRouter();

function handleSuccess(data) {
    router.push('/payment/success');
}

function handleError(error) {
    alert('Payment failed');
}
</script>
```

---

## Configuration Options

### init() Parameters

```javascript
window.HitPay.init(checkoutUrl, config, callbacks);
```

**checkoutUrl:** The payment checkout URL from API response

**config:**
| Option | Type | Description |
|--------|------|-------------|
| `domain` | string | HitPay domain (`sandbox.hit-pay.com` or `hit-pay.com`) |
| `apiDomain` | string | API domain (same as domain) |

**callbacks:**
| Callback | Description |
|----------|-------------|
| `onClose` | Called when modal is closed (any reason) |
| `onSuccess` | Called when payment succeeds |
| `onError` | Called when payment fails |

### toggle() Parameters

```javascript
window.HitPay.toggle({
    paymentRequest: 'payment-request-id',
    amount: '100.00',
    currency: 'SGD',
    method: 'paynow_online'  // Optional: pre-select payment method
});
```

---

## Common Issues

### Payment Methods Not Showing

**Symptoms:** Drop-in opens but shows no payment methods

**Causes & Solutions:**

| Cause | Solution |
|-------|----------|
| Payment methods not enabled | Enable in Dashboard → Settings → Payment Methods |
| Stripe not connected | Connect Stripe for card payments |
| DOKU methods (Indonesia) | DOKU not supported in drop-in - use redirect instead |
| Provider pending approval | Wait for provider to be activated |

### Modal Not Opening

**Symptoms:** Nothing happens when calling init() or toggle()

**Checklist:**
1. Is HitPay script loaded? Check console for errors
2. Is the checkout URL correct?
3. Is the payment request ID valid?
4. Check browser console for JavaScript errors

### "Powered by HitPay" Badge

The badge cannot be removed. This is required for all drop-in implementations.

### Close Timer (5 seconds)

After payment success, modal waits 5 seconds before closing. This is currently not configurable.

---

## Limitations

| Feature | Status |
|---------|--------|
| Card payments | Supported |
| PayNow | Supported |
| GrabPay | Supported |
| ShopeePay | Supported |
| **DOKU (Indonesia)** | **Not Supported** |
| Custom styling | Limited |
| Remove HitPay badge | Not allowed |
| Configurable close timer | Not available |

**For DOKU methods:** Use redirect flow instead of drop-in.

---

## Security Notes

1. **Never expose API keys** in frontend code
2. **Always validate payments** via webhook, not frontend callbacks
3. **Frontend callbacks** are for UX only, not payment confirmation
4. **Create payment requests** from your backend, not frontend

```javascript
// WRONG: Creating payment from frontend (exposes API key)
const response = await fetch('https://api.hit-pay.com/v1/payment-requests', {
    headers: { 'X-BUSINESS-API-KEY': 'your-key' }  // Exposed!
});

// CORRECT: Create via your backend
const response = await fetch('/api/payments/create', {
    method: 'POST',
    body: JSON.stringify({ amount: 100 })
});
```

---

## Testing

### Sandbox Testing

1. Use sandbox script: `https://sandbox.hit-pay.com/hitpay.js`
2. Set domain: `sandbox.hit-pay.com`
3. Create payment via sandbox API
4. Use test cards (4242 4242 4242 4242)

### Production Checklist

- [ ] Switch to production script
- [ ] Update domain to `hit-pay.com`
- [ ] Use production API key
- [ ] Test with small real transaction
- [ ] Verify webhook works in production
