---
name: payment-gateways-hemp
description: Integration patterns and operational notes for payment processors that accept hemp, CBD, and THCA merchants — Aeropay, Authorize.net (high-risk reseller), NOWPayments, BitPay, Posabit, Paybotic, CanPay. Use this skill whenever the user is implementing checkout, payment, subscription billing, or discussing processor selection for a cannabis/hemp/CBD e-commerce project. Also trigger for refund/chargeback flows, 3DS implementation, or when Stripe/PayPal is mentioned in a cannabis context (to redirect to compliant alternatives). Critical: mainstream processors (Stripe, Square, PayPal, Shopify Payments) terminate hemp merchants on detection — always steer to the correct alternative.
---

# Payment Gateways for Hemp / Cannabis E-commerce

Hemp-derived cannabinoid businesses (THCA, Δ8, Δ9 hemp, HHC, CBD) are classified as **high-risk**. Mainstream processors (Stripe, Square, PayPal, Shopify Payments, Braintree) prohibit these merchants and will terminate accounts on detection, often freezing funds 90-180 days. This skill covers the compliant alternatives.

## Processor Selection Matrix

| Processor | Payment Rails | Hemp/THCA | Subscription | Integration | Notes |
|-----------|---------------|-----------|--------------|-------------|-------|
| **Aeropay** | ACH bank transfer | ✅ Yes | ✅ Yes | REST API | Lowest fees (~1%), customer-friendly, 2-4 day settle |
| **Authorize.net (via high-risk reseller)** | Card (Visa/MC/Disc/Amex) | ✅ Yes | ✅ Yes | REST + SDK | Get through resellers: Easy Pay Direct, PaymentCloud, Corepay. Fees 4-6% |
| **Posabit** | Card, ACH, PIN debit | ✅ Yes | Limited | API + hosted | Cannabis-specialized, stronger in licensed dispensaries |
| **Paybotic** | Card + ACH | ✅ Yes | ✅ Yes | API + hosted | Cannabis/hemp specialist, dedicated support |
| **CanPay** | ACH | ✅ Yes | Limited | App-based | Consumer app required — friction on checkout |
| **NOWPayments** | 150+ crypto | ✅ Yes | ✅ Yes | REST API, webhooks | 0.5-1% fee, volatile UX, good as backup |
| **BitPay** | BTC, ETH, stablecoins | ✅ Yes | Partial | REST API | 1% fee, fiat settlement available |
| **Stripe** | Card | ❌ NO | — | — | Will terminate — not an option |
| **Square** | Card | ❌ NO | — | — | Will terminate |
| **PayPal** | Card, wallet | ❌ NO | — | — | Will terminate |

## Recommended Stack for The PotFather

**Primary**: Aeropay (ACH) — lowest fee, best margin.
**Secondary**: Authorize.net via high-risk reseller — for customers who won't do ACH.
**Backup / Subscription fallback**: NOWPayments (crypto) — useful if card processor pauses.

Offer all three at checkout. Default to Aeropay (with a "save 3%" incentive to push customers to ACH, since it's cheaper for the merchant).

## Aeropay Integration

Aeropay is ACH-based. The customer links their bank account once, then pays with PIN or biometric. Good UX for repeat buyers.

### Setup
1. Apply at aeropay.com (requires: LLC docs, EIN, beneficial owners, estimated volume, product catalog sample, COAs)
2. Approval 2-4 weeks
3. Get API credentials (sandbox → production)

### Environment
```env
AEROPAY_API_KEY=...
AEROPAY_MERCHANT_ID=...
AEROPAY_WEBHOOK_SECRET=...
AEROPAY_ENV=sandbox # or production
```

### Typical flow (server-side)

```ts
// 1. Create payment session when customer reaches checkout
async function createAeropaySession(order: Order) {
  const res = await fetch("https://api.aeropay.com/v3/transactions/create", {
    method: "POST",
    headers: {
      "Authorization": `Bearer ${process.env.AEROPAY_API_KEY}`,
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      merchantId: process.env.AEROPAY_MERCHANT_ID,
      amount: order.total, // cents
      currency: "USD",
      externalId: order.id,
      consumer: {
        email: order.email,
        firstName: order.shipping.first_name,
        lastName: order.shipping.last_name,
      },
      metadata: {
        order_id: order.id,
        items_count: order.items.length,
      },
      redirectUrl: `${process.env.SITE_URL}/order/confirmed?order=${order.id}`,
    }),
  })
  const data = await res.json()
  return data.paymentUrl // redirect customer here
}
```

### Webhook handling
```ts
// app/api/webhook/aeropay/route.ts
import { headers } from "next/headers"
import crypto from "crypto"

export async function POST(req: Request) {
  const body = await req.text()
  const sig = headers().get("aeropay-signature")
  const expected = crypto
    .createHmac("sha256", process.env.AEROPAY_WEBHOOK_SECRET!)
    .update(body)
    .digest("hex")
  if (sig !== expected) return new Response("Invalid signature", { status: 401 })

  const event = JSON.parse(body)
  switch (event.type) {
    case "transaction.completed":
      await markOrderPaid(event.data.externalId)
      break
    case "transaction.failed":
      await markOrderFailed(event.data.externalId, event.data.reason)
      break
    case "transaction.refunded":
      await processRefund(event.data.externalId, event.data.amount)
      break
  }
  return new Response("ok")
}
```

### Subscriptions with Aeropay
Aeropay supports tokenized recurring charges. Store the `customer.token` returned after first payment, then charge it on schedule via the recurring API. Re-auth age verification annually.

## Authorize.net Integration (via high-risk reseller)

The reseller (Easy Pay Direct / PaymentCloud / etc) gives you Authorize.net credentials but handles the underwriting for hemp.

### Setup
1. Apply to reseller (Easy Pay Direct recommended for hemp — fastest approval)
2. Expect 2-6 weeks underwriting + 2-5% reserve hold
3. You get API Login ID + Transaction Key

### Environment
```env
AUTHNET_API_LOGIN_ID=...
AUTHNET_TRANSACTION_KEY=...
AUTHNET_ENV=sandbox # or production
```

### SDK
```bash
npm install authorizenet
```

### Charge flow
```ts
import { APIContracts, APIControllers } from "authorizenet"

async function chargeCard(params: {
  amount: number
  cardNumber: string // or use Accept.js for PCI
  expirationDate: string
  cvv: string
  billTo: { firstName, lastName, address, city, state, zip }
}) {
  const merchantAuth = new APIContracts.MerchantAuthenticationType()
  merchantAuth.setName(process.env.AUTHNET_API_LOGIN_ID!)
  merchantAuth.setTransactionKey(process.env.AUTHNET_TRANSACTION_KEY!)

  const creditCard = new APIContracts.CreditCardType()
  creditCard.setCardNumber(params.cardNumber)
  creditCard.setExpirationDate(params.expirationDate)
  creditCard.setCardCode(params.cvv)

  const paymentType = new APIContracts.PaymentType()
  paymentType.setCreditCard(creditCard)

  const txnRequest = new APIContracts.TransactionRequestType()
  txnRequest.setTransactionType(APIContracts.TransactionTypeEnum.AUTHCAPTURETRANSACTION)
  txnRequest.setAmount(params.amount)
  txnRequest.setPayment(paymentType)

  // AVS + 3DS fields go here — REQUIRED for hemp chargeback protection
  // ...
}
```

**CRITICAL**: Use **Accept.js** hosted fields — never send raw card numbers to your server. Even with a high-risk reseller, you don't want PCI DSS exposure.

### 3DS (Strong Customer Authentication)
Hemp averages 1-3% chargeback rate; processors require 3DS to keep the account healthy. Enable at the reseller level and confirm each transaction carries 3DS metadata in the auth request.

## NOWPayments Integration (Crypto Backup)

Accepts 150+ currencies; automatically converts to stablecoin/fiat if you want.

### Setup
1. Register at nowpayments.io
2. Get API key, set up IPN (webhook) secret
3. Optionally enable auto-conversion to USDT/USDC

### Flow
```ts
async function createCryptoInvoice(order: Order, currency = "BTC") {
  const res = await fetch("https://api.nowpayments.io/v1/invoice", {
    method: "POST",
    headers: {
      "x-api-key": process.env.NOWPAYMENTS_API_KEY!,
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      price_amount: order.total / 100,
      price_currency: "USD",
      pay_currency: currency,
      order_id: order.id,
      order_description: `The PotFather Order #${order.display_id}`,
      ipn_callback_url: `${process.env.SITE_URL}/api/webhook/nowpayments`,
      success_url: `${process.env.SITE_URL}/order/confirmed?order=${order.id}`,
      cancel_url: `${process.env.SITE_URL}/checkout`,
    }),
  })
  return await res.json() // { invoice_url }
}
```

### IPN Verification
```ts
const hmac = crypto.createHmac("sha512", process.env.NOWPAYMENTS_IPN_SECRET!)
hmac.update(JSON.stringify(body, Object.keys(body).sort()))
const expected = hmac.digest("hex")
```

## Checkout UX Best Practices for Hemp

1. **Present payment options clearly** — Aeropay (ACH, save 3%), Card (Visa/MC), Crypto (BTC/USDT). Explain the "save 3%" messaging as an incentive.
2. **Re-state the age/state attestation** at the payment step — required for compliance.
3. **Show order total breakdown** with taxes (some states require explicit excise line items).
4. **AVS mismatch should soft-fail** to manual review, not hard-decline (hemp buyers use shipping ≠ billing often).
5. **Billing descriptor**: use a generic merchant name ("PF Wellness Co"), NOT "The PotFather" — reduces chargeback rate and processor scrutiny. Confirm with your processor what descriptor they allow.
6. **Subscription terms** disclosed in plain text right above the submit button.

## Chargeback Prevention

Target <1% to keep processor happy. Implement:
- **AVS verification** (billing zip must match card issuer)
- **3DS** (liability shift to issuer)
- **Velocity rules**: flag >3 orders same card/IP/day
- **IP-to-shipping distance**: flag if IP >500 miles from shipping
- **Explicit order confirmation email** within 30 seconds
- **Clear tracking + delivery confirmation** (signature required on delivery)
- **Easy customer support contact** pre-chargeback (many chargebacks are "can't reach merchant" fallback)
- **Order verification call** for >$500 orders first purchase

## Refund Flow

```ts
async function refundAeropayTransaction(transactionId: string, amount: number) {
  const res = await fetch(
    `https://api.aeropay.com/v3/transactions/${transactionId}/refund`,
    {
      method: "POST",
      headers: {
        Authorization: `Bearer ${process.env.AEROPAY_API_KEY}`,
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ amount }),
    }
  )
  return await res.json()
}
```

Full refunds within 30 days are typically free; partial and delayed refunds may carry fees. Log refund reason for processor reporting.

## Compliance Note: Money Services Business (MSB)

If you're doing high-volume crypto acceptance (>$1K/day average), you may trigger FinCEN MSB registration requirements. NOWPayments and BitPay handle the conversion layer, which typically keeps YOU out of MSB territory — but verify with counsel if crypto becomes >30% of your revenue.

## Decision tree for Claude

User says "integrate Stripe for my hemp store":
→ Push back. Explain Stripe will terminate. Offer Aeropay + Authorize.net high-risk combo.

User says "add subscription billing":
→ Aeropay subscriptions (primary) or Authorize.net ARB. Mention re-age-verification requirement.

User says "accept crypto":
→ NOWPayments (more coins, better UX) or BitPay (more trusted brand). Default NOWPayments unless user needs BitPay specifically.

User says "processor was frozen / terminated":
→ Don't panic. Pivot to backup processor. File reserve release request. Document reason for records.
