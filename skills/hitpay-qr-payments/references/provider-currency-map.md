# Provider-Currency Map

Complete mapping of HitPay payment providers to payment methods, currencies, and settlement behavior. Verified against `get_account_status` API response (March 2026).

## How to Read This Document

- **Provider**: The internal provider name returned by `get_account_status` in `payment_providers.completed[]`
- **API Value**: The string to pass in the `payment_methods` parameter
- **Currency**: The currency code required in the API call
- **Settlement**: The currency you receive in your HitPay balance

## Singapore (SGD)

| Provider | Method | API Value | Currency | Settlement |
|----------|--------|-----------|----------|------------|
| `dbs_sg` | PayNow | `paynow_online` | SGD | SGD |
| `dbs_max_sg` | PayNow (DBS MAX) | `paynow_online` | SGD | SGD |
| `grabpay` | GrabPay | `grabpay` | SGD | SGD |
| `shopee_pay` | ShopeePay | `shopee_pay` | SGD | SGD |
| `fave_sg` | FavePay | `favepay` | SGD | SGD |
| `shopback_sg` | ShopBack PayLater | `shopback_paylater` | SGD | SGD |
| `atome_sg` | Atome | `atome` | SGD | SGD |
| `zip` | Zip | `zip` | SGD | SGD |
| `stripe_sg` | Cards (Visa, MC, Amex) | `card` | SGD | SGD |
| `stripe_sg` | Apple Pay | `apple_pay` | SGD | SGD |
| `stripe_sg` | Google Pay | `google_pay` | SGD | SGD |
| `airwallex_sg` | Cards (international) | `card` | SGD | SGD |

**Notes:**
- `dbs_sg` and `dbs_max_sg` both route to PayNow — having either enables it
- FavePay and ShopBack PayLater are BNPL (buy-now-pay-later), not QR
- Cards use redirect checkout, not QR — use `create_payment_request` instead

## Malaysia (MYR)

| Provider | Method | API Value | Currency | Settlement |
|----------|--------|-----------|----------|------------|
| `grabpay` | GrabPay | `grabpay` | MYR | MYR |
| `shopee_pay` | ShopeePay | `shopee_pay` | MYR | MYR |
| `touch_n_go` | Touch 'n Go eWallet | `touch_n_go` | MYR | MYR |
| — | FPX (bank transfer) | `fpx` | MYR | MYR |

**Notes:**
- DuitNow is **not available** — recommend GrabPay, ShopeePay, or Touch 'n Go
- FPX is bank transfer (redirect flow), not QR-based
- `touch_n_go` provider must be in `completed[]` — check `get_account_status`

## Philippines (PHP)

| Provider | Method | API Value | Currency | Settlement |
|----------|--------|-----------|----------|------------|
| `qrph_netbank` | QR Ph (InstaPay) | `qrph` | PHP | PHP |
| `shopee_pay` | ShopeePay | `shopee_pay` | PHP | PHP |
| `gcash` | GCash | `gcash` | PHP | PHP |

**Notes:**
- GCash may appear in `pending[]` — verify with `get_account_status` before using
- QR Ph uses the InstaPay rail and works with all Philippine bank apps
- `billease_ph` (BNPL) is a separate provider — not QR-based

## Thailand (THB)

| Provider | Method | API Value | Currency | Settlement |
|----------|--------|-----------|----------|------------|
| `opn` | PromptPay | `promptpay` | THB | THB |
| `opn` | TrueMoney | `truemoney` | THB | THB |

**Notes:**
- Both methods route through the `opn` provider (Opn Payments, formerly Omise)
- LINE Pay is **not available** — recommend PromptPay or TrueMoney
- RabbitLINE Pay is **not available**

## Vietnam (VND)

| Provider | Method | API Value | Currency | Settlement |
|----------|--------|-----------|----------|------------|
| `vietqr_payme` | VietQR | `vietqr` | VND | VND |
| `zalopay` | ZaloPay | `zalopay` | VND | VND |

**Notes:**
- VietQR works with all Vietnamese banking apps that support VietQR standard
- Momo is **not available** — recommend VietQR (covers most bank apps) or ZaloPay
- `vietqr_payme` is the provider name; the API value is just `vietqr`

## Indonesia (IDR)

| Provider | Method | API Value | Currency | Settlement |
|----------|--------|-----------|----------|------------|
| `qfpay` | QRIS | `qris` | IDR | IDR |

**Notes:**
- QRIS is Indonesia's unified QR standard — works with GoPay, OVO, DANA, LinkAja, ShopeePay ID
- Individual wallets (DANA, OVO, GoPay) are **not available as separate methods** — use QRIS
- Routes through QFPay provider

## India (INR → SGD)

| Provider | Method | API Value | Currency | Settlement |
|----------|--------|-----------|----------|------------|
| `upi` | UPI | `upi` | SGD | SGD |

**Notes:**
- **Cross-currency**: Customer pays in INR, you receive SGD
- Set `currency: "SGD"` in the API call — the provider handles INR conversion
- FX rate applied at transaction time, not settlement time
- Works with all UPI apps (Google Pay India, PhonePe, Paytm, BHIM)

## Cross-Border Chinese Payments (CNY → SGD)

| Provider | Method | API Value | Currency | Settlement |
|----------|--------|-----------|----------|------------|
| `wechat` | WeChat Pay | `wechat` | SGD | SGD |
| `ifpay` | WeChat Pay | `wechat` | SGD | SGD |
| `qfpay` | WeChat Pay | `wechat` | SGD | SGD |
| `ifpay` | Alipay+ | `alipay` | SGD | SGD |
| `qfpay` | Alipay+ | `alipay` | SGD | SGD |

**Notes:**
- **Cross-currency**: Customer pays in CNY, you receive SGD
- Multiple providers can serve the same method — HitPay routes automatically
- Set `currency: "SGD"` — the provider handles CNY conversion on the customer side
- Primarily used for Chinese tourists paying at Singapore businesses

## Australia (AUD)

| Provider | Method | API Value | Currency | Settlement |
|----------|--------|-----------|----------|------------|
| `monoova` | PayTo | `payto` | AUD | AUD |
| `zip` | Zip | `zip` | AUD | AUD |

**Notes:**
- PayTo is a bank-initiated payment (not traditional QR)
- Zip is BNPL, not QR-based

## Korea (KRW)

| Provider | Method | API Value | Currency | Settlement |
|----------|--------|-----------|----------|------------|
| `nhn_kcp` | Korean card/bank methods | varies | KRW | KRW |

**Notes:**
- KCP (Korea Cyber Payment) aggregates Korean payment methods
- Specific method strings depend on KCP configuration
- Less commonly used for QR — primarily card and bank transfer

## Cryptocurrency

| Provider | Method | API Value | Currency | Settlement |
|----------|--------|-----------|----------|------------|
| `triple_a` | Bitcoin, ETH, USDT | `crypto` | SGD/USD | SGD |

**Notes:**
- Crypto payments settle in fiat (SGD)
- Not QR-based in the traditional sense — uses crypto wallet flow

## Known Gaps and Unavailable Methods

| Requested Method | Country | Status | Alternative |
|-----------------|---------|--------|-------------|
| DuitNow | Malaysia | Not available | GrabPay, ShopeePay, Touch 'n Go |
| DANA | Indonesia | Not separate | Use QRIS (covers DANA) |
| OVO | Indonesia | Not separate | Use QRIS (covers OVO) |
| GoPay | Indonesia | Not separate | Use QRIS (covers GoPay) |
| LinkAja | Indonesia | Not separate | Use QRIS (covers LinkAja) |
| LINE Pay | Thailand | Not available | PromptPay, TrueMoney |
| RabbitLINE Pay | Thailand | Not available | PromptPay, TrueMoney |
| Momo | Vietnam | Not available | VietQR, ZaloPay |
| KakaoPay | Korea | Not available | KCP methods |
| PayPay | Japan | Not available | — |
| Boost | Malaysia | Not available | GrabPay, ShopeePay, Touch 'n Go |

## Cross-Currency Settlement Summary

| Method | Customer Currency | Settlement Currency | FX Timing |
|--------|------------------|--------------------|-----------|
| WeChat Pay | CNY | SGD | At transaction |
| Alipay+ | CNY | SGD | At transaction |
| UPI | INR | SGD | At transaction |
| All others | Local | Local | No FX |

**Rule**: For cross-currency methods, always set `currency` to the **settlement currency** (SGD), not the customer's local currency. The payment provider handles the customer-facing conversion.
