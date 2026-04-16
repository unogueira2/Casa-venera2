---
name: coa-parser
description: Extract and structure data from cannabis Certificate of Analysis (COA) PDFs — cannabinoid potency panels, pesticide/heavy metal/microbial/residual solvent results, batch numbers, lab info, and pass/fail status. Use this skill whenever the user uploads a COA PDF, mentions "lab report" or "lab test" for cannabis products, asks to parse or extract data from a certificate, needs to populate product pages with cannabinoid data automatically, or is building inventory/catalog systems that sync with lab results. Also trigger when verifying COA data against product labels for compliance. Critical: COA data drives both compliance display requirements and customer trust — parse conservatively and flag anomalies rather than guessing.
---

# COA Parser — Cannabis Certificate of Analysis Extraction

A Certificate of Analysis (COA) is a third-party lab report required for hemp products. It proves Δ9-THC is ≤0.3% (Farm Bill compliance) and documents the cannabinoid/terpene profile, contaminant screens, and batch traceability.

This skill handles: extracting structured data from COA PDFs, validating completeness, flagging compliance concerns, and preparing the data for product catalog or display.

## COA Anatomy (What to Extract)

Every COA should contain these fields. Structure your extraction around them:

```typescript
interface COA {
  // Identification
  lab_name: string
  lab_license: string
  lab_iso_accreditation?: string // ISO 17025

  sample_id: string
  batch_number: string
  product_name: string
  product_type: "flower" | "concentrate" | "edible" | "tincture" | "vape" | "pre-roll" | "other"

  client_name: string // grower / manufacturer
  date_received: string // ISO
  date_tested: string // ISO
  date_reported: string // ISO

  // Core potency panel (cannabinoids, % by weight OR mg/g OR mg/serving)
  cannabinoids: {
    delta_9_thc_pct?: number      // CRITICAL: must be ≤0.3 for hemp
    delta_9_thc_mg_per_g?: number
    thca_pct?: number
    thca_mg_per_g?: number
    delta_8_thc_pct?: number
    cbd_pct?: number
    cbda_pct?: number
    cbg_pct?: number
    cbga_pct?: number
    cbn_pct?: number
    cbc_pct?: number
    thcv_pct?: number
    cbdv_pct?: number
    thcp_pct?: number
    total_thc_pct?: number  // computed: Δ9 + (THCA × 0.877)
    total_cbd_pct?: number
    total_cannabinoids_pct?: number
  }

  // Terpenes (optional but valued)
  terpenes?: Record<string, number> // e.g., { "myrcene": 1.2, "limonene": 0.8 } in %

  // Contaminant screens (pass/fail + values)
  pesticides: { status: "PASS" | "FAIL" | "NOT_TESTED"; details?: any }
  heavy_metals: { status: "PASS" | "FAIL" | "NOT_TESTED"; details?: any }
  microbials: { status: "PASS" | "FAIL" | "NOT_TESTED"; details?: any }
  mycotoxins: { status: "PASS" | "FAIL" | "NOT_TESTED"; details?: any }
  residual_solvents: { status: "PASS" | "FAIL" | "NOT_TESTED"; details?: any }
  foreign_matter?: { status: "PASS" | "FAIL" | "NOT_TESTED" }
  water_activity?: { value: number; unit: "aw"; status: "PASS" | "FAIL" }
  moisture?: { value: number; unit: "%"; status?: "PASS" | "FAIL" }

  // Overall
  overall_status: "PASS" | "FAIL" | "PARTIAL"
  farm_bill_compliant: boolean  // Δ9 ≤ 0.3%

  // Source
  source_url?: string
  coa_pdf_hash: string // SHA-256 of the PDF, to detect re-uploads
}
```

## Extraction Strategy

COAs are NOT standardized. Every lab uses a different layout. Tactics by order of robustness:

### 1. Structured PDF text extraction (first pass)
Use `pdftotext`, `pdfplumber` (Python), or `pdf-parse` (Node). Works when the PDF is text-based (most modern COAs are).

```python
import pdfplumber

with pdfplumber.open("sample-coa.pdf") as pdf:
    text = "\n".join(page.extract_text() or "" for page in pdf.pages)
```

Then regex + heuristics to find the cannabinoid table. Structural anchors that work across labs:
- "Cannabinoid Profile" / "Potency Analysis" / "Cannabinoid Analysis"
- Row labels like "Δ9-THC", "Delta-9 THC", "THC-9", "d9-THC"
- "% w/w" or "mg/g" column header

### 2. Table extraction (better for tables)
Use `camelot-py` or `tabula-py` (Python) to pull tabular structure. Helps when cannabinoid rows are in a real table vs flowing text.

```python
import camelot
tables = camelot.read_pdf("sample-coa.pdf", pages="1-2")
for t in tables:
    df = t.df
    # locate cannabinoid table by header content
```

### 3. OCR fallback (image-based COAs)
If PDF is scanned/image-only, use Tesseract via `pytesseract`:

```python
from pdf2image import convert_from_path
import pytesseract

images = convert_from_path("scanned-coa.pdf", dpi=300)
text = "\n".join(pytesseract.image_to_string(img) for img in images)
```

OCR output is noisy — apply extra regex forgiveness (e.g., `O` vs `0` in numbers).

### 4. LLM extraction (when heuristics fail)
After raw text extraction, if the structure is unclear, send the text to a vision-capable model with a strict JSON schema to fill. This is the pragmatic escape hatch for messy COAs. Prompt template:

```
You are extracting data from a cannabis Certificate of Analysis.
Return ONLY valid JSON matching this schema: <paste interface>.
Rules:
- If a field is absent, omit it or use null — do NOT invent values.
- Percentages must be numbers (no "%" sign).
- Status fields must be "PASS", "FAIL", or "NOT_TESTED" only.
- If the value is "<LOD" (below detection limit) or ND, use 0.
Document text follows:
---
{extracted_text}
```

## Cannabinoid Math Rules

### Total THC
```
total_thc = delta_9_thc + (thca × 0.877)
```
The 0.877 factor accounts for THCA losing a CO2 group when it decarboxylates to Δ9-THC (molecular weight ratio).

### Total CBD
```
total_cbd = cbd + (cbda × 0.877)
```

### Farm Bill check
```
farm_bill_compliant = delta_9_thc_pct <= 0.3
```
NOTE: The Farm Bill measures Δ9-THC only, NOT total THC. A flower with 25% THCA and 0.2% Δ9-THC is federally compliant even though total THC post-decarb is ~22%. This is the "THCA loophole" — document it carefully, and note this is under regulatory challenge.

### Cross-verification
If the COA reports both components and "Total THC", verify:
- `|reported_total_thc - computed_total_thc| < 0.1%` — else flag
- If discrepancy, trust component values, flag the COA for human review

## Pass/Fail Logic

Don't just copy the lab's status — compute your own from the raw data when possible:

```typescript
function computeOverallStatus(coa: COA): "PASS" | "FAIL" | "PARTIAL" {
  const screens = [
    coa.pesticides.status,
    coa.heavy_metals.status,
    coa.microbials.status,
    coa.residual_solvents.status,
  ]
  if (screens.includes("FAIL")) return "FAIL"
  if (!coa.farm_bill_compliant) return "FAIL"
  if (screens.includes("NOT_TESTED")) return "PARTIAL"
  return "PASS"
}
```

## Red Flags to Surface

When parsing, raise warnings for:
- Δ9-THC > 0.3% (not federally hemp — possible mislabel)
- Missing pesticide OR heavy metals panel (concentrates MUST have both)
- Date of testing > 12 months old (most states require recent COA)
- Lab name not recognized (maintain an allowlist of reputable labs)
- Total cannabinoids > 40% in flower (unusual — possible test error)
- Residual solvents "FAIL" on a solventless product (data entry error likely)
- No batch number (compliance violation)

## Trusted Lab Allowlist (sample)

Maintain an internal list. Examples of labs commonly accepted as reputable:
- SC Labs
- Anresco
- CannaSafe
- Infinite Chemical Analysis Labs
- ACS Laboratory
- Desert Valley Testing
- Encore Labs
- Green Scientific Labs
- Kaycha Labs

Flag COAs from unknown labs for manual review before publishing to product page.

## Implementation Patterns

### Node/TypeScript pipeline
```ts
import pdf from "pdf-parse"
import { readFileSync } from "fs"
import crypto from "crypto"

async function parseCOA(pdfPath: string): Promise<COA> {
  const buf = readFileSync(pdfPath)
  const hash = crypto.createHash("sha256").update(buf).digest("hex")
  const parsed = await pdf(buf)
  const text = parsed.text

  // 1. Extract batch number
  const batchMatch = text.match(/Batch\s*#?:?\s*([A-Z0-9\-]+)/i)
  // 2. Extract cannabinoids via regex table detection
  const cannabinoids = extractCannabinoids(text)
  // 3. Extract contaminant statuses
  const screens = extractScreens(text)
  // 4. Validate + compute
  const coa = assembleCOA({ hash, batch: batchMatch?.[1], cannabinoids, screens, text })
  validate(coa)
  return coa
}
```

### Python pipeline (more mature PDF ecosystem)
```python
from pathlib import Path
import hashlib
import pdfplumber
import re

def parse_coa(pdf_path: Path) -> dict:
    raw = pdf_path.read_bytes()
    sha = hashlib.sha256(raw).hexdigest()
    with pdfplumber.open(pdf_path) as pdf:
        text = "\n".join(p.extract_text() or "" for p in pdf.pages)
    return assemble({
        "coa_pdf_hash": sha,
        "batch_number": _find_batch(text),
        "cannabinoids": _extract_cannabinoids(text),
        "pesticides": _extract_screen(text, "pesticide"),
        # ...
    })
```

## Integration Patterns

### Auto-populate PDP on COA upload
When a new COA is uploaded by admin:
1. Parse and validate
2. Match batch number to product variant
3. If match: update product variant with `cannabinoids.*`, `batch_number`, `coa_url`
4. If no match: queue for manual link
5. Notify if `overall_status = FAIL` or compliance red flag

### Public COA lookup
Customers should be able to paste a batch number into a search and pull up the COA. Implement `/coa/[batch]` route that queries your COA table by batch number, renders a readable summary, links to the raw PDF.

### QR code on packaging
Every product ships with a QR linking to `https://thepotfather.com/coa/{batch_number}`. Customers scan, see lab report, trust increases, compliance satisfied.

## Handling ambiguous data

Never guess. If the parser can't determine a value with confidence:
- Return `null` for that field
- Log the ambiguity to an `extraction_notes` array
- Flag for human review rather than publishing false data
- Prefer silence to fabrication (false cannabinoid data = customer lawsuit risk)

## Claude's behavior with this skill

- When a user uploads a COA PDF, use the appropriate extraction tools (`pdfplumber` / `pdf-parse` for text, OCR for scans, LLM for messy cases) rather than guessing.
- Always return structured JSON matching the schema, even if partial.
- Surface red flags prominently, don't bury them.
- Never invent cannabinoid values to "fill out" a response — null is better than wrong.
- When building COA display UIs, design for clarity: batch number, date, overall status, cannabinoid table, screen statuses, link to raw PDF. No marketing fluff on the COA display page.
- Remember: a flower can be Farm Bill compliant (Δ9 ≤ 0.3%) AND have very high total THC post-decarb — this is the THCA loophole, not an error.
