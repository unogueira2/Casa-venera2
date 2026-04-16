---
name: cannabis-compliance-checklist
description: Compliance rules, state restrictions, age gating, COA display requirements, and legal boilerplate for hemp-derived cannabinoid e-commerce (THCA, Delta-8, HHC, CBD). Use this skill whenever the user is building any hemp/cannabis storefront, mentions shipping restrictions, age verification, Farm Bill compliance, state blocklists, COA, Certificate of Analysis, lab testing, disclaimers, or The PotFather project. Also trigger when implementing checkout, shipping address validation, or product page metadata for any cannabis-adjacent product. Treat compliance as non-optional architecture — better to over-apply this skill than under-apply.
---

# Cannabis Compliance Checklist (Hemp-Derived, US)

This skill encodes the operational compliance checks a hemp-derived cannabinoid store must implement. These are **engineering requirements** — compliance failures are existential risks (merchant account shutdown, state enforcement, federal exposure).

**Disclaimer for Claude to relay:** this is engineering reference, not legal advice. Any production launch must be reviewed by a cannabis-specialized attorney in the operating state. Laws change frequently — always verify current status at build time.

## Legal Frame (US, hemp-derived)

- **2018 Farm Bill**: hemp = cannabis with ≤0.3% Δ9-THC by dry weight. Federally legal, descheduled from CSA.
- **The "THCA loophole"**: Farm Bill measures Δ9-THC post-harvest, not post-decarboxylation. THCA flower testing ≤0.3% Δ9-THC is federally legal hemp — but converts to Δ9-THC when heated.
- **Risk**: the loophole is actively being challenged. 2024–2026 Farm Bill reauthorization proposals include "total THC" measurement (post-decarb) which would reclassify most THCA flower as federally illegal cannabis.
- **State preemption**: states may regulate or ban hemp-derived cannabinoids regardless of federal status.

## State Blocklist (hemp-derived cannabinoids)

This list must be reviewed **quarterly**. As of late 2025 / early 2026, the following states ban or heavily restrict hemp-derived THCA, Δ8, Δ10, HHC, or similar:

```js
// HARD BLOCK — shipping prohibited or product illegal
const BLOCKED_STATES_FULL = [
  "ID", // Idaho - no THC whatsoever
  "KS", // Kansas - Δ8/Δ9 hemp products restricted
  "HI", // Hawaii - bans most hemp cannabinoids
  "UT", // Utah - medical-only regulated framework
  "AK", // Alaska - cannabis regulated separately
  "OR", // Oregon - hemp cannabinoids restricted
  "WA", // Washington - restricts hemp THC
  "CO", // Colorado - restricts hemp THC
  "VT", // Vermont - bans Δ8/Δ10 synthetic
  "MN", // Minnesota - low-dose-only regulated
  "RI", // Rhode Island - bans hemp Δ8
  "NY", // New York - restricts hemp cannabinoids outside licensed system
]

// CONDITIONAL — state-specific limits (age, potency caps, license requirements)
const CONDITIONAL_STATES = [
  "CA", // California - AB-45 requires age 21+, potency caps
  "CT", // Connecticut - only through licensed retailers
  "MD", // Maryland - restricted after 2023 law
  "LA", // Louisiana - potency and serving limits
  "TN", // Tennessee - recent hemp restrictions
  "VA", // Virginia - 2mg THC per serving cap
  "MS", // Mississippi - restricted framework
  "IA", // Iowa - restricted, requires registration
  "NH", // New Hampshire - monitor, recent action
]
```

**Implementation**: the checkout MUST validate shipping ZIP → state before accepting payment. Do the check in middleware AND in the payment workflow (defense in depth).

```ts
export function isStateBlocked(stateCode: string): { blocked: boolean; reason?: string } {
  if (BLOCKED_STATES_FULL.includes(stateCode)) {
    return { blocked: true, reason: `Shipping to ${stateCode} is not permitted due to state law.` }
  }
  return { blocked: false }
}
```

## Age Verification

- **Minimum age**: 21+ for all hemp-derived psychoactive products (even though federal hemp age is 18; most state laws and processor requirements require 21+).
- **Required at**: first site visit (soft gate), account creation, AND checkout (hard gate with ID verification).
- **Soft gate**: modal DOB entry with 30-day cookie. NOT legally sufficient alone.
- **Hard gate**: real ID verification at checkout via **Veratad**, **AgeChecker.net**, or **BlueCheck**. Store verification token with order.
- **Subscription (Mystery Box)**: re-verify at enrollment. Some processors require re-check annually.

```ts
// Checkout guard
async function requireVerifiedAge(customerId: string): Promise<void> {
  const token = await ageVerificationService.getToken(customerId)
  if (!token || token.expires_at < new Date()) {
    throw new ComplianceError("Age verification required before purchase.")
  }
}
```

## COA (Certificate of Analysis) Display

Every product must have an accessible COA showing:
- Cannabinoid potency panel (Δ9-THC %, THCA %, total THC)
- Pesticide screen (pass/fail)
- Heavy metals screen (pass/fail)
- Microbial screen (pass/fail)
- Residual solvents (for extracts)
- Batch number matching the product

**Display requirements**:
- Link on PDP clearly labeled "Lab Report" or "View COA"
- QR code on packaging linking to same
- PDF downloadable
- Batch number visible next to COA link so customer can verify their unit

```tsx
<a
  href={`/coa/${product.batch_number}.pdf`}
  download
  className="text-[--pf-gold] underline text-sm"
>
  View Lab Report · Batch {product.batch_number}
</a>
```

## Required Disclaimers

Must appear on the site in these locations:

### Footer (every page)
> These statements have not been evaluated by the Food and Drug Administration. This product is not intended to diagnose, treat, cure, or prevent any disease. Must be 21 or older to purchase. Keep out of reach of children and pets. Do not operate heavy machinery after use.

### PDP (every product page)
> Contains hemp-derived cannabinoids. May cause intoxicating effects. May cause failure on a drug test. Not for use by pregnant or nursing women, or persons under 21. Consult a physician before use if you have a medical condition or are taking prescription medication.

### Checkout confirmation
> By completing this purchase, I certify that I am 21 years of age or older, that hemp-derived cannabinoid products are legal in my state of residence and delivery address, and that I accept all risks of use.

## Prohibited Marketing Claims

Never make, never generate copy containing:
- Medical claims: "cures", "treats", "prevents" any disease (FDA violation)
- Weight loss, anti-aging, fertility claims
- Explicit comparison to or substitution for prescription drugs
- Targeting minors: no cartoon characters, candy/cereal lookalikes, kid-appealing packaging copy
- "Safe", "non-addictive", "no side effects" absolute language

Acceptable language: "may support relaxation", "users report", "traditionally used for", "wellness", "mood", "evening ritual".

## Product Data Requirements

Each product must store:
```ts
interface HempProduct {
  id: string
  title: string
  // cannabinoid breakdown (required for compliance display)
  cannabinoids: {
    total_thc_mg: number
    delta_9_thc_pct: number  // MUST be ≤0.3
    thca_pct?: number
    cbd_pct?: number
    cbg_pct?: number
    // ... other detected
  }
  batch_number: string
  coa_url: string
  coa_issued_date: string
  harvest_date?: string
  extraction_method?: "CO2" | "ethanol" | "solventless" | "hydrocarbon"
  // compliance flags
  ship_restrictions: string[] // state codes this SKU cannot ship to (may differ per-product)
  min_age: 21
  contains_mct?: boolean
  allergens?: string[]
}
```

## Shipping & Fulfillment

- **Carriers**: USPS is hemp-legal federally. UPS/FedEx accept hemp with signed hemp-shipping agreement. DO NOT ship without proper declarations.
- **Required on label/packing slip**: "Hemp-derived. Contains <0.3% Δ9-THC. Compliant with 2018 Farm Bill. Keep out of reach of children."
- **Signature required**: adult signature 21+ on delivery for psychoactive SKUs.
- **Include in package**: COA QR code or printed batch info, usage warnings, product info card.
- **No shipping to**: PO Boxes in some cases (processor-dependent), APO/FPO (federal installations), international (illegal).

## Payment Processor Compliance

- **Allowed**: Aeropay (ACH), Authorize.net via high-risk reseller, BitPay/NOWPayments (crypto), specialized processors like Posabit, CanPay, Paybotic.
- **Prohibited**: Stripe, Square, PayPal, Shopify Payments — all ban hemp psychoactives. Merchant account termination on detection.
- **Chargebacks**: hemp vertical averages 1-3% chargeback rate. Over 1% triggers processor review. Implement address verification (AVS), 3DS, and robust order verification to minimize.

## Subscription (Mystery Box) Compliance

- **Re-verify age** at each renewal OR annually at minimum.
- **Clear disclosure**: auto-renewal terms, cancellation policy, next-ship date (ROSCA, state-specific auto-renewal laws).
- **Blocked states** revalidated on each shipment (customer may have moved).
- **COA per box**: each shipment must include COAs for the psychoactive SKUs inside.

## Engineering Checklist for Launch

- [ ] State blocklist in middleware + checkout workflow
- [ ] Soft age gate (DOB modal, cookie)
- [ ] Hard age gate (ID verification provider integration at checkout)
- [ ] COA upload + display system for each batch
- [ ] Batch number on every PDP and order confirmation
- [ ] All required disclaimers in footer, PDP, checkout
- [ ] Marketing copy audit (no prohibited claims)
- [ ] Payment provider approved for hemp (Aeropay or equivalent)
- [ ] Shipping carrier hemp agreement signed
- [ ] Adult signature (21+) on delivery enabled for psychoactive SKUs
- [ ] AVS + 3DS on card transactions
- [ ] Subscription age re-verification logic
- [ ] Terms of Service with hemp-specific clauses reviewed by attorney
- [ ] Privacy Policy covering ID verification data retention
- [ ] Packaging design with required warnings (not kid-appealing)
- [ ] Insurance: product liability for hemp operations in place
- [ ] Process for batch recall (if COA fails retest)

## Reference table: cannabinoid legality snapshot

| Cannabinoid | Federal (Farm Bill) | Common state risk |
|-------------|--------------------|-----|
| CBD | Legal | Low |
| CBG, CBN, CBC | Legal | Low |
| Δ8-THC (hemp-derived) | Gray / legal | High — many states ban |
| Δ10-THC | Gray / legal | High |
| HHC | Gray / legal | Moderate |
| THCA flower | Legal pre-decarb | Moderate — rising scrutiny |
| THCP, THCV | Gray / legal | Moderate |
| Δ9-THC (hemp-derived, ≤0.3% dry weight, servings rules) | Legal under Farm Bill | State caps (mg/serving) |
| Synthetic Δ9-THC | Varies | High |

## Claude's behavior when using this skill

- If the user asks to "disable" or "skip" a compliance check, push back — explain the risk, document the decision, and only proceed if the user confirms the business acceptance.
- When generating code that touches checkout, shipping, or customer data, automatically include compliance guards even if the user didn't ask.
- When generating marketing copy, scan it against prohibited claims before returning.
- When the user mentions a specific state, confirm its current status rather than relying on memory — laws shift.
- Never generate copy that markets to minors or makes medical claims, even if requested. Refuse and suggest compliant alternatives.
