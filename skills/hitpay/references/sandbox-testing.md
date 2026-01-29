# Sandbox Testing Guide

## Getting Started

### Create Sandbox Account

1. Go to [dashboard.sandbox.hit-pay.com](https://dashboard.sandbox.hit-pay.com)
2. Register with your email
3. Complete basic business information (can use dummy data)
4. Get API keys from Settings → Payment Gateway → API Keys

### Environment Configuration

| Setting | Sandbox | Production |
|---------|---------|------------|
| Dashboard | dashboard.sandbox.hit-pay.com | dashboard.hit-pay.com |
| API Base URL | api.sandbox.hit-pay.com | api.hit-pay.com |
| Checkout URL | securecheckout.sandbox.hit-pay.com | securecheckout.hit-pay.com |
| Drop-in JS | sandbox.hit-pay.com/hitpay.js | www.hit-pay.com/hitpay.js |

---

## Sandbox Availability by Country

| Country | Sandbox Available | Alternative |
|---------|-------------------|-------------|
| Singapore | Yes | - |
| Malaysia | Yes | - |
| Thailand | Yes | - |
| Vietnam | Yes | - |
| Indonesia | No | Test with small live transactions |
| Philippines | No | Test with small live transactions |

**For countries without sandbox:** Create production account and test with minimum amounts (e.g., 1 SGD).

---

## Testing Card Payments

### Connect Stripe (Required for Cards)

1. Go to Dashboard → Settings → Payment Methods
2. Click "Connect Stripe" under Credit Cards
3. Follow Stripe's sandbox setup flow

### Test Card Numbers

From [Stripe Testing](https://stripe.com/docs/testing):

| Scenario | Card Number | Expiry | CVC |
|----------|-------------|--------|-----|
| **Success** | 4242 4242 4242 4242 | Any future | Any 3 digits |
| **Declined** | 4000 0000 0000 0002 | Any future | Any 3 digits |
| **Insufficient funds** | 4000 0000 0000 9995 | Any future | Any 3 digits |
| **3D Secure required** | 4000 0027 6000 3184 | Any future | Any 3 digits |

### ZIP Code Requirement

Some test cards (especially US cards) require ZIP code. Enter any 5-6 digit postal code.

---

## Testing PayNow QR

**This is the most common point of confusion!**

### Correct Process

1. Create payment request with `payment_methods: ['paynow_online']`
2. Open the checkout URL or render the QR code
3. **Scan QR code with your phone's CAMERA APP** (not a banking app!)
4. The QR contains a URL that opens a mock payment page
5. Click the "Pay" button on the mock page
6. Payment is simulated as successful

### Common Mistakes

| Mistake | Why It Fails | Solution |
|---------|--------------|----------|
| Using banking app to scan | Banking apps expect real PayNow data | Use camera app |
| Scanning but nothing happens | QR is URL, not PayNow payload | Open the URL in browser |
| Trying to pay real money | It's sandbox mode | Just click mock "Pay" button |

### Step-by-Step Screenshots Flow

```
1. Generate QR → 2. Scan with camera → 3. Open URL → 4. Click "Pay" → 5. Webhook received
```

---

## Testing Webhooks

### Option 1: Use webhook.site

1. Go to [webhook.site](https://webhook.site)
2. Copy your unique URL
3. Use this URL as `webhook` parameter in payment request
4. Complete test payment
5. View received webhook data on webhook.site

### Option 2: Use ngrok for localhost

```bash
# Install ngrok
npm install -g ngrok

# Expose local port 3000
ngrok http 3000

# Use the https URL as webhook
# Example: https://abc123.ngrok.io/webhook/hitpay
```

### Webhook Testing Checklist

- [ ] Webhook URL is publicly accessible
- [ ] URL uses HTTPS
- [ ] Endpoint returns HTTP 200
- [ ] HMAC validation is working
- [ ] Payment status is processed correctly

---

## Testing Drop-in UI

### Setup

```html
<!-- Sandbox -->
<script src="https://sandbox.hit-pay.com/hitpay.js"></script>

<!-- Production -->
<script src="https://www.hit-pay.com/hitpay.js"></script>
```

### Test Flow

```javascript
// Create payment request via your backend first
const { paymentRequestId } = await createPayment();

// Open drop-in
window.HitPay.init(
    'https://securecheckout.sandbox.hit-pay.com/payment-request/@your-business/' + paymentRequestId + '/checkout',
    {
        domain: 'sandbox.hit-pay.com', // Use 'hit-pay.com' for production
        apiDomain: 'sandbox.hit-pay.com'
    },
    {
        onClose: () => console.log('Closed'),
        onSuccess: (data) => console.log('Success', data),
        onError: (error) => console.log('Error', error)
    }
);
```

### Drop-in UI Limitations

| Feature | Supported |
|---------|-----------|
| Card payments | Yes |
| PayNow | Yes |
| GrabPay | Yes |
| ShopeePay | Yes |
| DOKU methods (Indonesia) | **No** |
| Custom styling | Limited |

---

## Testing Refunds

### Via API

```bash
curl -X POST https://api.sandbox.hit-pay.com/v1/refund \
  -H "X-BUSINESS-API-KEY: your_sandbox_key" \
  -d "payment_id=payment_id_from_completed_payment" \
  -d "amount=50.00"  # Optional: partial refund
```

### Refund Rules

- Can only refund completed payments
- Full refund: omit `amount` parameter
- Partial refund: specify `amount` less than original
- Multiple partial refunds allowed until fully refunded

---

## Testing Recurring Billing

### Create Subscription

```bash
curl -X POST https://api.sandbox.hit-pay.com/v1/recurring-billing \
  -H "X-BUSINESS-API-KEY: your_sandbox_key" \
  -d "plan_id=your_plan_id" \
  -d "customer_email=test@example.com" \
  -d "start_date=2024-01-01" \
  -d "redirect_url=https://yoursite.com/subscription/success"
```

### Test Saved Cards

1. Create subscription with card
2. Customer enters test card details
3. Card is saved for future charges
4. Subsequent charges happen automatically

---

## Common Sandbox Issues

### "Payment method not available"

**Cause:** Payment provider not connected or not enabled

**Solution:**
1. Check Dashboard → Settings → Payment Methods
2. Connect Stripe for cards
3. Enable PayNow
4. Verify all providers show "Active"

### 419 Page Expired

**Cause:** Token/session conflict between sandbox and production

**Solution:**
1. Use private/incognito browser for sandbox
2. Clear cookies
3. Don't switch between sandbox/production in same browser session

### QR Code Won't Scan

**Cause:** QR payload is large, hard to scan

**Solution:**
1. Increase QR code display size
2. Ensure good lighting
3. Hold phone steady
4. Try a QR scanner app instead of camera

### Webhook Not Received

**Checklist:**
1. Is URL publicly accessible? (not localhost)
2. Does URL return 200 status?
3. Is HMAC validation passing?
4. Check firewall/Cloudflare settings
5. Use webhook.site to verify HitPay is sending

---

## Moving to Production

### Checklist

- [ ] Create production account at dashboard.hit-pay.com
- [ ] Complete business verification
- [ ] Connect production Stripe account
- [ ] Update API base URL to `api.hit-pay.com`
- [ ] Update API key to production key
- [ ] Update webhook URL to production endpoint
- [ ] Update Drop-in JS URL
- [ ] Test with small real transaction

### Code Changes

```javascript
// Change from sandbox to production
const BASE_URL = process.env.NODE_ENV === 'production'
    ? 'https://api.hit-pay.com'
    : 'https://api.sandbox.hit-pay.com';

const API_KEY = process.env.NODE_ENV === 'production'
    ? process.env.HITPAY_PRODUCTION_KEY
    : process.env.HITPAY_SANDBOX_KEY;
```
