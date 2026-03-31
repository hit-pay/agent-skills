# React QR Payment Page Template

Complete standalone React component for rendering a HitPay QR payment page. Replace the `PAYMENT_DATA` values with actual API response data from `create_embedded_qr`.

## Usage

1. Call `create_embedded_qr` via the HitPay MCP
2. Copy this template into a React artifact
3. Replace `PAYMENT_DATA` values with the actual response
4. Set `borderless` to `null` for domestic payments

## Template

```jsx
import { useState, useEffect, useRef } from "react";

// ============================================
// REPLACE THESE VALUES WITH API RESPONSE DATA
// ============================================
const PAYMENT_DATA = {
  id: "REPLACE_WITH_PAYMENT_ID",
  amount: "100.00",
  currency: "SGD",
  formattedAmount: "SGD 100.00",
  paymentMethod: "QRPH (InstaPay)",
  qrCode: "REPLACE_WITH_QR_CODE_STRING",
  qrCodeExpiry: null, // ISO string from qr_code_data.qr_code_expiry, or null for 15-min default
  checkoutUrl: "REPLACE_WITH_CHECKOUT_URL",
  purpose: "",
  referenceNumber: "",
  // Set to null for domestic payments
  borderless: {
    customerPays: "PHP 4,557.00",
    merchantReceives: "SGD 100.00",
    displayRate: "1 SGD = 45.57 PHP",
    feeNote: "1.5% cross-border processing fee applies",
  },
};

// ============================================
// BRAND COLORS — DO NOT MODIFY
// ============================================
const COLORS = {
  deepBlue: "#002771",
  logoBlue: "#0E2859",
  actionBlue: "#2465DE",
  white: "#FFFFFF",
  successGreen: "#4DAB80",
  textPrimary: "#03102F",
  textSecondary: "#61667C",
  danger: "#E5484D",
  cardBg: "rgba(14, 40, 89, 0.6)",
  cardBorder: "rgba(36, 101, 222, 0.2)",
};

// ============================================
// COUNTDOWN HOOK
// ============================================
function useCountdown(expiryIso) {
  const getTarget = () => {
    if (expiryIso) return new Date(expiryIso).getTime();
    return Date.now() + 15 * 60 * 1000; // 15-minute default
  };

  const [target] = useState(getTarget);
  const [remaining, setRemaining] = useState(
    Math.max(0, Math.floor((target - Date.now()) / 1000))
  );

  useEffect(() => {
    const interval = setInterval(() => {
      const diff = Math.max(0, Math.floor((target - Date.now()) / 1000));
      setRemaining(diff);
      if (diff <= 0) clearInterval(interval);
    }, 1000);
    return () => clearInterval(interval);
  }, [target]);

  const minutes = Math.floor(remaining / 60);
  const seconds = remaining % 60;
  const display = `${String(minutes).padStart(2, "0")}:${String(seconds).padStart(2, "0")}`;
  const isUrgent = remaining < 120 && remaining > 0;
  const isExpired = remaining <= 0;

  return { display, isUrgent, isExpired, remaining };
}

// ============================================
// QR CODE RENDERER
// ============================================
function QRCodeDisplay({ value, size = 200 }) {
  const containerRef = useRef(null);
  const [fallback, setFallback] = useState(false);

  useEffect(() => {
    if (!containerRef.current) return;

    // Check if the value is already a URL or base64 image
    if (value.startsWith("http") || value.startsWith("data:image")) {
      setFallback(true);
      return;
    }

    // Inject qrcode.js CDN and render
    const script = document.createElement("script");
    script.src = "https://cdn.jsdelivr.net/npm/qrcodejs@1.0.0/qrcode.min.js";
    script.onload = () => {
      if (containerRef.current && window.QRCode) {
        // Clear previous QR code children safely
        while (containerRef.current.firstChild) {
          containerRef.current.removeChild(containerRef.current.firstChild);
        }
        new window.QRCode(containerRef.current, {
          text: value,
          width: size,
          height: size,
          colorDark: COLORS.textPrimary,
          colorLight: COLORS.white,
          correctLevel: window.QRCode.CorrectLevel.M,
        });
      }
    };
    script.onerror = () => setFallback(true);
    document.head.appendChild(script);

    return () => {
      try { document.head.removeChild(script); } catch (e) {}
    };
  }, [value, size]);

  if (fallback) {
    const src = value.startsWith("data:image") ? value : value;
    return (
      <div style={{ display: "flex", justifyContent: "center" }}>
        <img src={src} alt="QR Code" style={{ width: size, height: size }} />
      </div>
    );
  }

  return <div ref={containerRef} style={{ display: "flex", justifyContent: "center" }} />;
}

// ============================================
// MAIN COMPONENT
// ============================================
export default function QRPaymentPage() {
  const { display, isUrgent, isExpired } = useCountdown(PAYMENT_DATA.qrCodeExpiry);

  return (
    <div
      style={{
        minHeight: "100vh",
        background: `linear-gradient(135deg, ${COLORS.deepBlue} 0%, ${COLORS.actionBlue} 100%)`,
        display: "flex",
        flexDirection: "column",
        alignItems: "center",
        justifyContent: "center",
        padding: "24px",
        fontFamily:
          "'Hauora', 'Manrope', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif",
      }}
    >
      {/* Main Card */}
      <div
        style={{
          width: "100%",
          maxWidth: 420,
          background: COLORS.cardBg,
          backdropFilter: "blur(20px)",
          borderRadius: 20,
          border: `1px solid ${COLORS.cardBorder}`,
          padding: "32px 28px",
          boxShadow: "0 24px 48px rgba(0, 0, 0, 0.3)",
        }}
      >
        {/* Header */}
        <div style={{ textAlign: "center", marginBottom: 24 }}>
          <div
            style={{
              fontSize: 12,
              fontWeight: 600,
              letterSpacing: "0.08em",
              textTransform: "uppercase",
              color: COLORS.actionBlue,
              marginBottom: 8,
            }}
          >
            {PAYMENT_DATA.paymentMethod}
          </div>
          <div
            style={{
              fontSize: 36,
              fontWeight: 700,
              color: COLORS.white,
              letterSpacing: "-0.02em",
            }}
          >
            {PAYMENT_DATA.formattedAmount}
          </div>
          {PAYMENT_DATA.purpose && (
            <div style={{ fontSize: 14, color: "rgba(255,255,255,0.6)", marginTop: 4 }}>
              {PAYMENT_DATA.purpose}
            </div>
          )}
        </div>

        {/* QR Code Container */}
        <div
          style={{
            background: COLORS.white,
            borderRadius: 16,
            padding: 24,
            marginBottom: 20,
          }}
        >
          {isExpired ? (
            <div
              style={{
                textAlign: "center",
                padding: "40px 20px",
                color: COLORS.danger,
                fontSize: 16,
                fontWeight: 600,
              }}
            >
              QR code expired. Please create a new payment.
            </div>
          ) : (
            <QRCodeDisplay value={PAYMENT_DATA.qrCode} size={220} />
          )}
        </div>

        {/* Timer */}
        <div style={{ textAlign: "center", marginBottom: 20 }}>
          <div
            style={{
              fontSize: 12,
              color: "rgba(255,255,255,0.5)",
              marginBottom: 4,
              textTransform: "uppercase",
              letterSpacing: "0.05em",
            }}
          >
            {isExpired ? "Expired" : "Expires in"}
          </div>
          <div
            style={{
              fontSize: 28,
              fontWeight: 700,
              fontVariantNumeric: "tabular-nums",
              color: isExpired
                ? COLORS.danger
                : isUrgent
                  ? COLORS.danger
                  : COLORS.white,
            }}
          >
            {display}
          </div>
        </div>

        {/* Borderless FX Info */}
        {PAYMENT_DATA.borderless && (
          <div
            style={{
              background: "rgba(255,255,255,0.08)",
              borderRadius: 12,
              padding: "16px 20px",
              marginBottom: 20,
              border: `1px solid ${COLORS.cardBorder}`,
            }}
          >
            <div
              style={{
                fontSize: 11,
                fontWeight: 600,
                textTransform: "uppercase",
                letterSpacing: "0.08em",
                color: COLORS.actionBlue,
                marginBottom: 12,
              }}
            >
              Cross-Border Payment
            </div>
            <div
              style={{
                display: "flex",
                justifyContent: "space-between",
                marginBottom: 8,
              }}
            >
              <span style={{ fontSize: 13, color: "rgba(255,255,255,0.6)" }}>
                Customer pays
              </span>
              <span style={{ fontSize: 14, fontWeight: 600, color: COLORS.white }}>
                {PAYMENT_DATA.borderless.customerPays}
              </span>
            </div>
            <div
              style={{
                display: "flex",
                justifyContent: "space-between",
                marginBottom: 8,
              }}
            >
              <span style={{ fontSize: 13, color: "rgba(255,255,255,0.6)" }}>
                You receive
              </span>
              <span
                style={{
                  fontSize: 14,
                  fontWeight: 600,
                  color: COLORS.successGreen,
                }}
              >
                {PAYMENT_DATA.borderless.merchantReceives}
              </span>
            </div>
            <div
              style={{
                display: "flex",
                justifyContent: "space-between",
                marginBottom: 8,
              }}
            >
              <span style={{ fontSize: 13, color: "rgba(255,255,255,0.6)" }}>
                Exchange rate
              </span>
              <span style={{ fontSize: 13, color: "rgba(255,255,255,0.8)" }}>
                {PAYMENT_DATA.borderless.displayRate}
              </span>
            </div>
            {PAYMENT_DATA.borderless.feeNote && (
              <div
                style={{
                  fontSize: 11,
                  color: "rgba(255,255,255,0.4)",
                  marginTop: 4,
                  fontStyle: "italic",
                }}
              >
                {PAYMENT_DATA.borderless.feeNote}
              </div>
            )}
          </div>
        )}

        {/* Open in Browser Link */}
        <a
          href={PAYMENT_DATA.checkoutUrl}
          target="_blank"
          rel="noopener noreferrer"
          style={{
            display: "block",
            textAlign: "center",
            padding: "12px",
            background: COLORS.actionBlue,
            color: COLORS.white,
            borderRadius: 10,
            textDecoration: "none",
            fontSize: 14,
            fontWeight: 600,
            marginBottom: 16,
            transition: "opacity 0.2s",
          }}
          onMouseOver={(e) => (e.currentTarget.style.opacity = "0.9")}
          onMouseOut={(e) => (e.currentTarget.style.opacity = "1")}
        >
          Open in Browser
        </a>

        {/* Footer */}
        <div style={{ textAlign: "center" }}>
          <div style={{ fontSize: 11, color: "rgba(255,255,255,0.3)" }}>
            Payment ID: {PAYMENT_DATA.id}
          </div>
          {PAYMENT_DATA.referenceNumber && (
            <div style={{ fontSize: 11, color: "rgba(255,255,255,0.3)", marginTop: 2 }}>
              Ref: {PAYMENT_DATA.referenceNumber}
            </div>
          )}
        </div>
      </div>

      {/* Powered by HitPay */}
      <div
        style={{
          marginTop: 20,
          fontSize: 12,
          color: "rgba(255,255,255,0.35)",
          textAlign: "center",
        }}
      >
        Powered by HitPay
      </div>
    </div>
  );
}
```

## Data Injection Guide

When generating the artifact, replace the `PAYMENT_DATA` values as follows:

| Template Field | API Response Field | Notes |
|----------------|-------------------|-------|
| `id` | `id` | Payment request ID |
| `amount` | Raw amount string (e.g., `"100.00"`) | Numeric amount without currency |
| `currency` | Currency code (e.g., `"SGD"`) | From the request |
| `formattedAmount` | Formatted amount (e.g., `"SGD 100.00"`) | From the response |
| `paymentMethod` | Display name (e.g., `"QRPH (InstaPay)"`) | Map from API code to display name |
| `qrCode` | `qr_code_data.qr_code` | Raw QR payload string or base64 |
| `qrCodeExpiry` | `qr_code_data.qr_code_expiry` | ISO string, or `null` for 15-min default |
| `checkoutUrl` | `checkout_url` or `url` | The hosted checkout page URL |
| `purpose` | From user input | Optional description |
| `referenceNumber` | From user input | Optional reference |
| `borderless` | `borderless_fx` object | Set to `null` for domestic payments |
| `borderless.customerPays` | `borderless_fx.customer_pays` | e.g., `"PHP 4,557.00"` |
| `borderless.merchantReceives` | `borderless_fx.merchant_receives` | e.g., `"SGD 100.00"` |
| `borderless.displayRate` | `borderless_fx.display_rate` | e.g., `"1 SGD = 45.57 PHP"` |
| `borderless.feeNote` | `borderless_fx.fee_note` | e.g., `"1.5% cross-border processing fee applies"` |

## Payment Method Display Names

| API Code | Display Name |
|----------|-------------|
| `paynow_online` | PayNow |
| `qrph_netbank` | QRPH (InstaPay) |
| `gcash_qr` | GCash |
| `opn_prompt_pay` | PromptPay |
| `opn_true_money_qr` | TrueMoney |
| `ifpay_qris` | QRIS |
| `doku_qris` | QRIS (DOKU) |
| `upi_qr` | UPI |
| `shopee_pay` | ShopeePay |
| `grabpay_direct` | GrabPay |
| `touch_n_go` | Touch 'n Go |
| `wechat_pay` | WeChat Pay |
| `atome` | Atome |
