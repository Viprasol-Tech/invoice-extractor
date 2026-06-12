---
name: invoice-extractor
description: Extract structured data from invoices, receipts, and bills into clean validated JSON — vendor, line items, totals, tax, and payment terms. Validates the math and flags discrepancies. Use for accounts-payable automation, bookkeeping, expense processing, or document data entry.
version: 1.0.0
---

# Invoice Extractor

A structured methodology for turning an invoice, receipt, or bill — a PDF, a
scanned image, a phone photo, or pasted text — into one clean, **validated**
JSON object: vendor and bill-to details, line items, totals, tax/VAT, and
payment terms. The skill **checks the arithmetic** (line items sum to subtotal,
subtotal − discount + tax + shipping = total, amount due = total − paid) and
emits a `validation` block that flags every discrepancy instead of silently
"fixing" the numbers.

> **Verify before you pay.** Extracted values can be wrong — OCR misreads a
> digit, a stamped "PAID" overlaps a number, a multi-page total is split. Always
> verify extracted values against the source document before posting to your
> ledger or paying. Treat any field marked low-confidence as needing human
> review. This is a data-entry aid, not financial or legal advice.

---

## When to Activate

Activate this skill when the user:

- Shares an **invoice, receipt, bill, purchase order, credit note, or proforma**
  as a PDF, image (PNG/JPG/HEIC), scan, photo, or pasted text and asks to
  extract, parse, digitize, read, or "pull the data out of" it.
- Asks for accounts-payable automation, bookkeeping data entry, expense
  processing, or "turn this receipt into a spreadsheet/JSON".
- Wants line items, totals, tax, or vendor details lifted into a structured
  format, or asks "does this invoice add up?".

If the document is unreadable, partly cut off, or a key total is illegible, say
so and extract what is legible — **never fabricate a number to make the math
balance.** Mark anything guessed or derived with low confidence.

---

## Step 1: Identify Document Type

Before extracting, classify the document and note its quality — both change how
you read it and how much to trust each field.

**Document type** (state which, and why):

| Type | Tell |
|---|---|
| **Invoice** | "Invoice", an invoice number, an amount **due** and due date — a demand for payment. |
| **Receipt** | Proof of a **completed** payment — "Paid", tender/change, often a register/POS layout. |
| **Purchase order (PO)** | Buyer-issued, a "PO number", **ordered** quantities, often no tax/amount-due. |
| **Credit note** | Negative/refund document — "Credit", "Credit memo", reverses a prior invoice. |
| **Proforma** | Quote/estimate issued **before** the sale — "Proforma", "Quotation", not yet payable. |

**Quality flags** (carry into `confidence`): scanned vs native PDF, phone photo
(skew/glare/shadow), handwritten fields, faint thermal receipt, multi-page (note
which page totals live on), non-English or mixed-language, multiple currencies.

---

## Step 2: Field Extraction

Pull these fields. Anything absent → `null` (do not invent). Anything derived,
guessed, or OCR-ambiguous → record a `confidence` entry (Step 5).

**Header**
- Vendor / supplier **name**, **address**, **tax / VAT / GST id**, phone/email.
- **Bill-to** (and **ship-to** if different): name + address.
- **Invoice / document number**, **PO reference** (if any).
- **Issue date**, **due date**.
- **Currency** (the symbol or code shown on the document).

**Line items** (one object per line):
- **Description**, **quantity**, **unit price**, **line amount**.
- **Tax rate** / **tax category** per line, if shown (e.g. `20%`, `VAT`,
  `Exempt`, `Zero-rated`). If tax is only stated at the document level, leave
  line-level tax `null` and capture it in `totals`.

**Totals**
- **Subtotal** (pre-tax net), **discount**, **tax / VAT**, **shipping /
  freight**, **grand total**.
- **Amount paid**, **amount due / balance** (for receipts, paid = total and due
  = 0; for invoices, due is what's owed).

**Payment**
- **Terms** (e.g. `Net 30`, `Due on receipt`, `2/10 Net 30`).
- **Bank details** if present: bank name, **IBAN**, **account / routing /
  sort code**, SWIFT/BIC. **Accepted payment methods** (card, ACH, wire, etc.).

---

## Step 3: Normalization Rules

Normalize as you extract so the JSON is machine-clean — but **preserve the
original string** wherever a transform could be wrong.

- **Dates → ISO 8601** (`YYYY-MM-DD`). When the format is ambiguous (e.g.
  `03/04/2025` — is it Mar 4 or Apr 3?), use document locale/currency as a hint,
  flag it low-confidence, and keep the raw string in `confidence.note`.
- **Amounts → plain decimals.** Strip thousands separators and currency symbols;
  keep the decimal point. `$1,234.50` → `1234.50`. Respect locale decimal commas
  (`1.234,50` → `1234.50`). Negative/refund amounts stay negative.
- **Currency → ISO 4217 code** (`$` → `USD` or `CAD`/`AUD` per context, `€` →
  `EUR`, `£` → `GBP`). If the symbol is ambiguous, keep the symbol and flag it.
- **Map synonyms to the canonical field:** "Total Due" / "Balance Due" /
  "Amount Payable" / "Please Pay" → `amount_due`; "Sub-total" / "Net" →
  `subtotal`; "VAT" / "GST" / "Sales Tax" → `tax`; "Bill To" / "Sold To" →
  `bill_to`.
- **Preserve original strings where ambiguous** — never overwrite a raw value
  you transformed with low confidence; keep both.

---

## Step 4: Validation & Reconciliation

Run every applicable check and record the result. **Never silently adjust a
number to make a check pass — flag it.** Allow a small rounding tolerance
(±0.02 of the document currency, or per-line rounding) before calling a
mismatch.

| Check | Rule |
|---|---|
| **Line items → subtotal** | Σ(line amounts) = subtotal |
| **Line math** | quantity × unit_price = line_amount (per line) |
| **Subtotal → total** | subtotal − discount + tax + shipping = grand_total |
| **Tax** | tax ≈ subtotal × tax_rate (within rounding), where a rate is given |
| **Amount due** | amount_due = grand_total − amount_paid |

For each check emit a `validation` entry: the `check` name, `status` of `pass`
or `fail`, and on failure the `expected`, `actual`, and `discrepancy` (signed
difference) so a human can see exactly what's off. If a check can't run (a
needed field is missing), mark it `status: "skipped"` with a `reason`.

---

## Step 5: Output Format

Emit, in this order: (1) a strict JSON object, (2) a short human-readable
summary, (3) a flagged-issues list.

### JSON shape

```json
{
  "document_type": "invoice | receipt | purchase_order | credit_note | proforma",
  "currency": "ISO 4217 code",
  "vendor": {
    "name": "string|null",
    "address": "string|null",
    "tax_id": "string|null",
    "email": "string|null",
    "phone": "string|null"
  },
  "bill_to": { "name": "string|null", "address": "string|null" },
  "ship_to": { "name": "string|null", "address": "string|null" },
  "invoice_number": "string|null",
  "po_reference": "string|null",
  "issue_date": "YYYY-MM-DD|null",
  "due_date": "YYYY-MM-DD|null",
  "line_items": [
    {
      "description": "string",
      "quantity": 0,
      "unit_price": 0.00,
      "line_amount": 0.00,
      "tax_rate": "string|null",
      "tax_category": "string|null"
    }
  ],
  "totals": {
    "subtotal": 0.00,
    "discount": 0.00,
    "tax": 0.00,
    "shipping": 0.00,
    "grand_total": 0.00,
    "amount_paid": 0.00,
    "amount_due": 0.00
  },
  "payment": {
    "terms": "string|null",
    "methods": ["string"],
    "bank": {
      "bank_name": "string|null",
      "iban": "string|null",
      "account_number": "string|null",
      "routing_or_sort_code": "string|null",
      "swift_bic": "string|null"
    }
  },
  "validation": [
    {
      "check": "line_items_sum_to_subtotal",
      "status": "pass | fail | skipped",
      "expected": 0.00,
      "actual": 0.00,
      "discrepancy": 0.00,
      "reason": "string (only when skipped)"
    }
  ],
  "confidence": [
    { "field": "dotted.path", "level": "high | medium | low", "note": "why" }
  ]
}
```

Rules: omit `confidence` entries for fields read cleanly — list only low/medium
ones (and any field you transformed under ambiguity). Use `null` for absent
fields, never `""` or `0` as a stand-in for "unknown". Keep numbers as JSON
numbers, not strings.

### Inline example A — clean receipt (all checks pass)

```json
{
  "document_type": "receipt",
  "currency": "USD",
  "vendor": { "name": "Blue Bottle Coffee", "address": "12 Mint St, San Francisco, CA 94103", "tax_id": null, "email": null, "phone": null },
  "bill_to": { "name": null, "address": null },
  "invoice_number": "R-558210",
  "issue_date": "2025-05-18",
  "due_date": null,
  "line_items": [
    { "description": "Cappuccino", "quantity": 2, "unit_price": 4.50, "line_amount": 9.00, "tax_rate": null, "tax_category": null },
    { "description": "Almond croissant", "quantity": 1, "unit_price": 5.00, "line_amount": 5.00, "tax_rate": null, "tax_category": null }
  ],
  "totals": { "subtotal": 14.00, "discount": 0.00, "tax": 1.23, "shipping": 0.00, "grand_total": 15.23, "amount_paid": 15.23, "amount_due": 0.00 },
  "payment": { "terms": "Paid", "methods": ["card"], "bank": null },
  "validation": [
    { "check": "line_items_sum_to_subtotal", "status": "pass", "expected": 14.00, "actual": 14.00, "discrepancy": 0.00 },
    { "check": "subtotal_minus_discount_plus_tax_plus_shipping_equals_total", "status": "pass", "expected": 15.23, "actual": 15.23, "discrepancy": 0.00 },
    { "check": "amount_due_equals_total_minus_paid", "status": "pass", "expected": 0.00, "actual": 0.00, "discrepancy": 0.00 }
  ]
}
```

### Inline example B — invoice with a flagged subtotal mismatch

```json
{
  "document_type": "invoice",
  "currency": "EUR",
  "vendor": { "name": "Hektar Supplies GmbH", "address": "Industriestr. 9, 80807 München", "tax_id": "DE811209227", "email": "billing@hektar.de", "phone": null },
  "bill_to": { "name": "Acme Robotics Ltd", "address": "Unit 4, Park Road, Leeds LS1 2AB" },
  "invoice_number": "2025-0412",
  "po_reference": "PO-7781",
  "issue_date": "2025-04-03",
  "due_date": "2025-05-03",
  "line_items": [
    { "description": "M6 hex bolts (box/500)", "quantity": 10, "unit_price": 12.00, "line_amount": 120.00, "tax_rate": "19%", "tax_category": "VAT" },
    { "description": "Threadlocker 50ml", "quantity": 4, "unit_price": 8.50, "line_amount": 34.00, "tax_rate": "19%", "tax_category": "VAT" }
  ],
  "totals": { "subtotal": 160.00, "discount": 0.00, "tax": 29.26, "shipping": 0.00, "grand_total": 183.26, "amount_paid": 0.00, "amount_due": 183.26 },
  "payment": { "terms": "Net 30", "methods": ["wire"], "bank": { "bank_name": "Stadtsparkasse München", "iban": "DE89370400440532013000", "swift_bic": "SSKMDEMM", "account_number": null, "routing_or_sort_code": null } },
  "validation": [
    { "check": "line_items_sum_to_subtotal", "status": "fail", "expected": 154.00, "actual": 160.00, "discrepancy": 6.00 },
    { "check": "subtotal_minus_discount_plus_tax_plus_shipping_equals_total", "status": "fail", "expected": 189.26, "actual": 183.26, "discrepancy": -6.00 },
    { "check": "tax_equals_subtotal_times_rate", "status": "fail", "expected": 30.40, "actual": 29.26, "discrepancy": -1.14 }
  ],
  "confidence": [
    { "field": "totals.subtotal", "level": "low", "note": "Line items sum to 154.00 but printed subtotal is 160.00 — printed subtotal kept; mismatch flagged for review." }
  ]
}
```

### Human-readable summary

After the JSON, add 2–4 lines: document type, vendor, total + currency, due
date/terms, and whether validation passed. Then a **flagged-issues list** —
every failed/skipped check and every low-confidence field, most material first,
each with the numbers behind it.

---

## Disclaimer

This skill is a **document data-entry aid, not financial, accounting, tax, or
legal advice.** Extraction depends on document quality: OCR misreads, skewed
scans, handwriting, overlapping stamps, and layout ambiguity all cause errors.
**Verify extracted values against the source document before posting to a ledger
or paying anything**, and treat low-confidence fields as requiring human review.
The validation block flags arithmetic discrepancies — it does not certify that a
document is genuine or that a charge is correct. This skill is **not affiliated
with or endorsed by Anthropic** or any accounting-software provider.
