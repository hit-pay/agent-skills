---
name: hitpay-php
description: Integrate HitPay payment gateway using PHP and cURL. Use when user says "HitPay PHP", "HitPay Laravel", "HitPay cURL", "PHP payment integration", or needs PHP code for HitPay.
license: MIT
metadata:
  author: hitpay
  version: "1.0.0"
---

# HitPay PHP Integration

Payment gateway integration for PHP applications using pure cURL (no SDK dependencies). Works with Laravel, WordPress, Magento, and vanilla PHP.

## When to Apply

Reference this skill when:
- Integrating HitPay with PHP applications
- Using cURL for HitPay API calls
- Building Laravel payment integrations
- Handling HitPay webhooks in PHP
- Troubleshooting PHP SDK issues

## API Basics

| Environment | Base URL |
|-------------|----------|
| Sandbox | `https://api.sandbox.hit-pay.com` |
| Production | `https://api.hit-pay.com` |

## Create Payment Request

### Basic cURL Implementation

```php
<?php
function createHitPayPayment($amount, $currency, $email, $redirectUrl, $webhookUrl, $referenceNumber = null, $paymentMethods = []) {
    $apiKey = getenv('HITPAY_API_KEY'); // or use your config
    $baseUrl = getenv('HITPAY_ENV') === 'production'
        ? 'https://api.hit-pay.com'
        : 'https://api.sandbox.hit-pay.com';

    $data = [
        'amount' => $amount,
        'currency' => $currency,
        'email' => $email,
        'redirect_url' => $redirectUrl,
        'webhook' => $webhookUrl,
    ];

    if ($referenceNumber) {
        $data['reference_number'] = $referenceNumber;
    }

    // Payment methods array - IMPORTANT: use payment_methods[] format
    if (!empty($paymentMethods)) {
        foreach ($paymentMethods as $method) {
            $data['payment_methods[]'] = $method;
        }
    }

    $curl = curl_init();
    curl_setopt_array($curl, [
        CURLOPT_URL => $baseUrl . '/v1/payment-requests',
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_POST => true,
        CURLOPT_POSTFIELDS => http_build_query($data),
        CURLOPT_HTTPHEADER => [
            'X-BUSINESS-API-KEY: ' . $apiKey,
            'Content-Type: application/x-www-form-urlencoded',
            'X-Requested-With: XMLHttpRequest'
        ],
        CURLOPT_TIMEOUT => 30,
    ]);

    $response = curl_exec($curl);
    $httpCode = curl_getinfo($curl, CURLINFO_HTTP_CODE);
    $error = curl_error($curl);
    curl_close($curl);

    if ($error) {
        throw new Exception("cURL Error: " . $error);
    }

    $result = json_decode($response, true);

    if ($httpCode !== 200 && $httpCode !== 201) {
        $errorMsg = $result['message'] ?? 'Unknown error';
        throw new Exception("HitPay API Error ({$httpCode}): {$errorMsg}");
    }

    return $result;
}

// Usage example
try {
    $payment = createHitPayPayment(
        100.00,                              // amount
        'SGD',                               // currency
        'customer@example.com',              // email
        'https://yoursite.com/payment/success', // redirect
        'https://yoursite.com/webhook/hitpay', // webhook
        'ORDER-12345',                       // reference number
        ['paynow_online', 'card']            // payment methods
    );

    // Redirect customer to payment page
    header('Location: ' . $payment['url']);
    exit;
} catch (Exception $e) {
    echo "Payment creation failed: " . $e->getMessage();
}
```

### Payment Methods Array (Common Issue)

The most common PHP integration issue is passing `payment_methods` as an array. Use one of these approaches:

**Option 1: Multiple fields**
```php
$data = [
    'amount' => 100,
    'currency' => 'SGD',
    // ... other fields
];

// Add payment methods as separate entries
$postFields = http_build_query($data);
$postFields .= '&payment_methods[]=paynow_online';
$postFields .= '&payment_methods[]=card';

curl_setopt($curl, CURLOPT_POSTFIELDS, $postFields);
```

**Option 2: PHP array syntax**
```php
$data = [
    'amount' => 100,
    'currency' => 'SGD',
    'payment_methods' => ['paynow_online', 'card'], // Will become payment_methods[0], payment_methods[1]
];

// Use http_build_query with PHP_QUERY_RFC1738
curl_setopt($curl, CURLOPT_POSTFIELDS, http_build_query($data));
```

## Webhook Handling

HitPay sends two types of webhooks:

1. **Vendor Webhook** (form-encoded) - sent per payment request
2. **Event Webhook** (JSON) - sent based on dashboard settings

### Vendor Webhook (Most Common)

This is received as form-encoded POST data with an `hmac` field for verification.

```php
<?php
function handleHitPayWebhook() {
    $salt = getenv('HITPAY_SALT');

    // Get POST data
    $data = $_POST;

    // Extract and remove hmac from data
    $receivedHmac = $data['hmac'] ?? '';
    unset($data['hmac']);

    // Sort keys alphabetically
    ksort($data);

    // Build signature string: key1value1key2value2...
    $signatureString = '';
    foreach ($data as $key => $value) {
        $signatureString .= $key . $value;
    }

    // Calculate expected HMAC (SHA256)
    $calculatedHmac = hash_hmac('sha256', $signatureString, $salt);

    // Verify signature
    if (!hash_equals($calculatedHmac, $receivedHmac)) {
        http_response_code(401);
        echo json_encode(['error' => 'Invalid signature']);
        exit;
    }

    // Process the payment
    $paymentId = $data['payment_id'];
    $paymentRequestId = $data['payment_request_id'];
    $status = $data['status'];
    $amount = $data['amount'];
    $currency = $data['currency'];
    $referenceNumber = $data['reference_number'] ?? '';

    if ($status === 'completed') {
        // Mark order as paid in your database
        // updateOrderStatus($referenceNumber, 'paid');

        // Log successful payment
        error_log("Payment completed: {$paymentId} - {$amount} {$currency}");
    }

    // Return 200 OK - IMPORTANT: HitPay retries if non-200
    http_response_code(200);
    echo json_encode(['received' => true]);
}

// Call the handler
handleHitPayWebhook();
```

### Webhook Payload Fields

| Field | Description |
|-------|-------------|
| `payment_id` | Unique payment ID |
| `payment_request_id` | Payment request ID (from create API) |
| `phone` | Customer phone (may be empty) |
| `amount` | Payment amount |
| `currency` | Currency code |
| `status` | `completed`, `failed`, `pending` |
| `reference_number` | Your order reference |
| `hmac` | HMAC-SHA256 signature |

### HMAC Validation Details

**IMPORTANT:** Include ALL fields except `hmac` in the signature, even if empty:

```php
// Example webhook data
$data = [
    'payment_id' => '9e2d6dc0-dd6d-4443-95a2-b68b3a1eef2f',
    'payment_request_id' => '9e2d6dab-53d6-4f83-baf0-8f3d69e58baa',
    'phone' => '',  // Empty but still included
    'amount' => '2.00',
    'currency' => 'MYR',
    'status' => 'completed',
    'reference_number' => '',  // Empty but still included
    'hmac' => '7e8c948dd7037b4ec5013a5224886c0bced6ea06f5ad4842db0c89552ab2cc7c'
];

// Remove hmac
$hmac = $data['hmac'];
unset($data['hmac']);

// Sort alphabetically
ksort($data);

// Build string: amountXXXcurrencyXXXpayment_idXXX...
$str = '';
foreach ($data as $k => $v) {
    $str .= $k . $v;
}

// Result: amount2.00currencyMYRpayment_id9e2d6dc0-...payment_request_id9e2d6dab-...phonereference_numberstatuscompleted

$expected = hash_hmac('sha256', $str, $salt);
```

## Get Payment Status

```php
<?php
function getHitPayPaymentStatus($paymentRequestId) {
    $apiKey = getenv('HITPAY_API_KEY');
    $baseUrl = getenv('HITPAY_ENV') === 'production'
        ? 'https://api.hit-pay.com'
        : 'https://api.sandbox.hit-pay.com';

    $curl = curl_init();
    curl_setopt_array($curl, [
        CURLOPT_URL => $baseUrl . '/v1/payment-requests/' . $paymentRequestId,
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_HTTPHEADER => [
            'X-BUSINESS-API-KEY: ' . $apiKey,
            'X-Requested-With: XMLHttpRequest'
        ],
    ]);

    $response = curl_exec($curl);
    curl_close($curl);

    return json_decode($response, true);
}
```

## Refund Payment

```php
<?php
function refundHitPayPayment($paymentId, $amount = null) {
    $apiKey = getenv('HITPAY_API_KEY');
    $baseUrl = getenv('HITPAY_ENV') === 'production'
        ? 'https://api.hit-pay.com'
        : 'https://api.sandbox.hit-pay.com';

    $data = [];
    if ($amount !== null) {
        $data['amount'] = $amount; // Partial refund
    }

    $curl = curl_init();
    curl_setopt_array($curl, [
        CURLOPT_URL => $baseUrl . '/v1/refund',
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_POST => true,
        CURLOPT_POSTFIELDS => http_build_query(array_merge($data, [
            'payment_id' => $paymentId
        ])),
        CURLOPT_HTTPHEADER => [
            'X-BUSINESS-API-KEY: ' . $apiKey,
            'Content-Type: application/x-www-form-urlencoded',
            'X-Requested-With: XMLHttpRequest'
        ],
    ]);

    $response = curl_exec($curl);
    curl_close($curl);

    return json_decode($response, true);
}
```

## Laravel Integration

### Service Class

```php
<?php
// app/Services/HitPayService.php

namespace App\Services;

use Illuminate\Support\Facades\Http;
use Illuminate\Support\Facades\Log;

class HitPayService
{
    private string $apiKey;
    private string $salt;
    private string $baseUrl;

    public function __construct()
    {
        $this->apiKey = config('services.hitpay.api_key');
        $this->salt = config('services.hitpay.salt');
        $this->baseUrl = config('services.hitpay.sandbox')
            ? 'https://api.sandbox.hit-pay.com'
            : 'https://api.hit-pay.com';
    }

    public function createPaymentRequest(array $params): array
    {
        $response = Http::withHeaders([
            'X-BUSINESS-API-KEY' => $this->apiKey,
            'X-Requested-With' => 'XMLHttpRequest',
        ])->asForm()->post($this->baseUrl . '/v1/payment-requests', $params);

        if (!$response->successful()) {
            Log::error('HitPay API error', [
                'status' => $response->status(),
                'body' => $response->body(),
            ]);
            throw new \Exception('HitPay API error: ' . $response->body());
        }

        return $response->json();
    }

    public function validateWebhook(array $data): bool
    {
        $hmac = $data['hmac'] ?? '';
        unset($data['hmac']);
        ksort($data);

        $signatureString = '';
        foreach ($data as $key => $value) {
            $signatureString .= $key . $value;
        }

        $calculatedHmac = hash_hmac('sha256', $signatureString, $this->salt);

        return hash_equals($calculatedHmac, $hmac);
    }
}
```

### Config

```php
<?php
// config/services.php

return [
    // ... other services

    'hitpay' => [
        'api_key' => env('HITPAY_API_KEY'),
        'salt' => env('HITPAY_SALT'),
        'sandbox' => env('HITPAY_SANDBOX', true),
    ],
];
```

### Controller

```php
<?php
// app/Http/Controllers/HitPayWebhookController.php

namespace App\Http\Controllers;

use App\Services\HitPayService;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Log;

class HitPayWebhookController extends Controller
{
    public function handle(Request $request, HitPayService $hitpay)
    {
        $data = $request->all();

        // Validate webhook signature
        if (!$hitpay->validateWebhook($data)) {
            Log::warning('Invalid HitPay webhook signature', $data);
            return response()->json(['error' => 'Invalid signature'], 401);
        }

        // Process based on status
        if ($data['status'] === 'completed') {
            // Update your order
            // Order::where('reference', $data['reference_number'])
            //     ->update(['status' => 'paid', 'payment_id' => $data['payment_id']]);
        }

        return response()->json(['received' => true]);
    }
}
```

### Route (disable CSRF for webhooks)

```php
<?php
// routes/web.php
Route::post('/webhook/hitpay', [HitPayWebhookController::class, 'handle'])
    ->withoutMiddleware([\App\Http\Middleware\VerifyCsrfToken::class]);
```

## Common Errors

### 404 Not Found

**Cause:** Wrong URL, method, or missing trailing content
**Solution:**
```php
// Wrong
CURLOPT_URL => 'https://api.sandbox.hit-pay.com/v1/payment-requests/'

// Correct
CURLOPT_URL => 'https://api.sandbox.hit-pay.com/v1/payment-requests'
```

### Invalid Business API Key

**Cause:** Using production key with sandbox URL or vice versa
**Solution:** Ensure API key matches environment:
- Sandbox: `https://api.sandbox.hit-pay.com` + sandbox API key
- Production: `https://api.hit-pay.com` + production API key

### 422 The given data was invalid

**Cause:** Missing required fields or invalid format
**Solution:** Check amount (must be > 0.30), currency, and email format

### HMAC Validation Fails

**Common causes:**
1. Using SHA1 instead of SHA256
2. Not sorting keys alphabetically
3. Not including empty fields
4. Not removing `hmac` before calculating
5. Using wrong salt (API key vs salt)

### Webhook Not Received

1. Webhook URL must be publicly accessible (not localhost)
2. Use https://webhook.site for testing
3. Return HTTP 200 status code
4. Check firewall/Cloudflare settings

## Sandbox Testing

### Test Cards
| Result | Number | Expiry | CVC |
|--------|--------|--------|-----|
| Success | 4242 4242 4242 4242 | Any future | Any 3 digits |
| Declined | 4000 0000 0000 0002 | Any future | Any 3 digits |

### Testing PayNow QR
1. Create payment with `paynow_online` method
2. Scan QR code with your phone's **camera app** (not banking app)
3. Click the link to open mock payment page
4. Click "Pay" to simulate successful payment

## Environment Variables

```bash
HITPAY_API_KEY=your_api_key_here
HITPAY_SALT=your_webhook_salt_here
HITPAY_ENV=sandbox  # or production
```
