# Common Errors Troubleshooting Guide

## API Errors

### 404 Not Found

**Symptoms:**
- Response: `{"message": "Not Found"}`
- HTTP status code 404

**Common Causes:**

| Cause | Solution |
|-------|----------|
| Wrong HTTP method | Use `POST` for creating payments, `GET` for status |
| Trailing slash in URL | Remove trailing slash: `https://api.hit-pay.com/v1/payment-requests` |
| Typo in endpoint | Check endpoint spelling |
| Wrong API version | Use `/v1/` prefix |

**Correct URLs:**
```
POST https://api.sandbox.hit-pay.com/v1/payment-requests     # Create payment
GET  https://api.sandbox.hit-pay.com/v1/payment-requests/{id} # Get status
POST https://api.sandbox.hit-pay.com/v1/refund                # Create refund
```

---

### 403 Forbidden

**Symptoms:**
- Response: `{"message": "Invalid business api key"}`
- HTTP status code 403

**Common Causes:**

| Cause | Solution |
|-------|----------|
| Production key with sandbox URL | Match key to environment |
| Sandbox key with production URL | Match key to environment |
| Missing API key header | Add `X-BUSINESS-API-KEY` header |
| Typo in header name | Use exact header: `X-BUSINESS-API-KEY` |
| Key from wrong account | Verify key in dashboard |

**Environment Matching:**
```
Sandbox:    https://api.sandbox.hit-pay.com + sandbox API key
Production: https://api.hit-pay.com         + production API key
```

**Correct Headers:**
```
X-BUSINESS-API-KEY: your_api_key_here
Content-Type: application/x-www-form-urlencoded  # or application/json
X-Requested-With: XMLHttpRequest
```

---

### 422 Unprocessable Entity

**Symptoms:**
- Response: `{"message": "The given data was invalid.", "errors": {...}}`
- HTTP status code 422

**Common Causes:**

| Error | Cause | Solution |
|-------|-------|----------|
| `amount` required | Missing amount | Include `amount` parameter |
| `currency` required | Missing currency | Include `currency` (e.g., "SGD") |
| Invalid amount | Amount too small | Minimum is 0.30 |
| Invalid email | Malformed email | Use valid email format |
| Invalid payment method | Wrong method code | Use exact codes like `paynow_online` |

**Minimum Amount:**
```
amount >= 0.30 (currency units, not cents)
```

**Valid Payment Methods:**
```
card, paynow_online, grabpay_direct, shopee_pay, wechat, alipay, fpx, promptpay
```

---

### 500 Internal Server Error

**Symptoms:**
- Response: Server error message
- HTTP status code 500

**Common Causes:**

| Cause | Solution |
|-------|----------|
| Stripe account issue | Reconnect Stripe in dashboard |
| Payment provider down | Check HitPay status page |
| Invalid Stripe configuration | Verify Stripe connection |

**Solution:**
1. Check [HitPay Status](https://status.hitpayapp.com)
2. Reconnect Stripe: Dashboard → Settings → Payment Methods → Credit Cards
3. Contact support if persists

---

## Webhook Issues

### Webhook Not Received

**Checklist:**

| Check | Solution |
|-------|----------|
| URL is localhost | Use public URL or ngrok |
| URL not HTTPS | Use HTTPS for production |
| Firewall blocking | Whitelist HitPay IPs |
| Cloudflare blocking | Check firewall rules |
| Webhook URL empty | Include `webhook` parameter in API call |

**Testing Webhooks:**
1. Use [webhook.site](https://webhook.site) for testing
2. Use ngrok for localhost: `ngrok http 3000`
3. Check HitPay logs in dashboard

---

### HMAC Validation Fails

**Most Common Issue!** This accounts for ~25% of developer support questions.

**Checklist:**

| Check | Solution |
|-------|----------|
| Wrong algorithm | Use SHA256, not SHA1 |
| Keys not sorted | Sort alphabetically with `ksort()` or `.sort()` |
| Empty fields excluded | Include ALL fields, even empty ones |
| Using API key as salt | Salt is separate - get from dashboard |
| `hmac` not removed | Remove `hmac` field before calculating |
| Wrong data type | Ensure all values are strings |

**Correct Calculation:**

```php
// PHP - CORRECT
$hmac = $data['hmac'];
unset($data['hmac']);      // 1. Remove hmac
ksort($data);              // 2. Sort keys

$str = '';
foreach ($data as $k => $v) {
    $str .= $k . $v;       // 3. Concatenate key+value
}

$expected = hash_hmac('sha256', $str, $salt);  // 4. SHA256
return hash_equals($expected, $hmac);          // 5. Compare
```

```javascript
// Node.js - CORRECT
const { hmac, ...rest } = data;
const str = Object.keys(rest)
    .sort()
    .map(k => `${k}${rest[k]}`)
    .join('');

const expected = crypto
    .createHmac('sha256', salt)
    .update(str)
    .digest('hex');
```

**Example Calculation:**

Given data:
```
payment_id=abc123
amount=100.00
currency=SGD
status=completed
phone=
reference_number=ORDER-1
hmac=received_hash
```

Signature string (after sorting, excluding hmac):
```
amount100.00currencySGDpayment_idabc123phonereference_numberORDER-1statuscompleted
```

---

### Webhook Returns Non-200

**Symptoms:**
- HitPay retries webhook multiple times
- Payment shows as "webhook failed" in dashboard

**Common Causes:**

| Cause | Solution |
|-------|----------|
| Throwing exception | Catch all errors, return 200 |
| Slow processing | Process async, return 200 quickly |
| Database error | Handle gracefully |
| Missing return statement | Always return response |

**Best Practice:**

```php
try {
    // Process webhook
    processPayment($data);
} catch (Exception $e) {
    // Log error but still return 200
    error_log("Webhook error: " . $e->getMessage());
}

// Always return 200 to acknowledge receipt
http_response_code(200);
echo json_encode(['received' => true]);
```

---

## Payment Method Issues

### Payment Methods Not Showing

**For API:**

| Cause | Solution |
|-------|----------|
| Method not enabled | Enable in Dashboard → Settings → Payment Methods |
| Stripe not connected | Connect Stripe for card payments |
| Wrong method code | Use exact codes (e.g., `paynow_online` not `paynow`) |
| Array format wrong | Use `payment_methods[]` in form data |

**For Drop-in UI:**

| Cause | Solution |
|-------|----------|
| Provider not enabled | Enable provider in dashboard |
| DOKU methods | DOKU not supported in drop-in UI |
| Provider pending | Wait for provider approval |

---

### Payment Methods Array Format

This is a common issue when passing multiple payment methods.

**PHP (cURL):**
```php
// Option 1: Separate fields
$postFields = 'amount=100&currency=SGD';
$postFields .= '&payment_methods[]=paynow_online';
$postFields .= '&payment_methods[]=card';

// Option 2: http_build_query
$data = [
    'amount' => 100,
    'currency' => 'SGD',
    'payment_methods' => ['paynow_online', 'card']
];
$postFields = http_build_query($data);
```

**Node.js:**
```javascript
const qs = require('qs');
const data = {
    amount: 100,
    currency: 'SGD',
    payment_methods: ['paynow_online', 'card']
};
const body = qs.stringify(data, { arrayFormat: 'brackets' });
```

**C#:**
```csharp
request.AddParameter("payment_methods[]", "paynow_online");
request.AddParameter("payment_methods[]", "card");
```

---

## Sandbox Issues

### Can't Create Sandbox Account

**By Country:**

| Country | Solution |
|---------|----------|
| Indonesia | Sandbox not available - test with small production transactions |
| Philippines | Sandbox not available - test with small production transactions |
| Other | Register at dashboard.sandbox.hit-pay.com |

### Sandbox PayNow QR Not Working

**Correct Process:**
1. Generate PayNow QR in sandbox
2. Scan with **phone camera app** (NOT banking app)
3. Open the link in browser
4. Click "Pay" button to simulate payment

**Do NOT:**
- Scan with banking app (will fail)
- Try to actually pay (it's sandbox)

### 419 Page Expired (Sandbox)

**Cause:** Session/token issue when switching between sandbox and production

**Solutions:**
1. Use private/incognito browser
2. Clear browser cookies
3. Use separate browser for sandbox

---

## CORS Errors

**Symptoms:**
```
Access to fetch blocked by CORS policy: No 'Access-Control-Allow-Origin' header
```

**Cause:** Calling HitPay API from frontend JavaScript

**Solution:**
- **Never call HitPay API from frontend** - exposes API key
- Create backend endpoint that calls HitPay
- Frontend calls your backend, backend calls HitPay

```javascript
// WRONG - frontend directly calling HitPay
const response = await fetch('https://api.hit-pay.com/v1/payment-requests', {
    headers: { 'X-BUSINESS-API-KEY': apiKey } // Exposed!
});

// CORRECT - frontend calls your backend
const response = await fetch('/api/payments/create', {
    method: 'POST',
    body: JSON.stringify({ amount, orderId })
});
```
