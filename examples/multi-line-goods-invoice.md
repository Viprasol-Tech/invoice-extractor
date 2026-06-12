# Example — Multi-Line Goods Invoice (Per-Line Tax, Discount, Subtotal Mismatch Flagged)

> **Illustrative example.** "Foundry Industrial Supply" and all figures below
> are fictional, used to demonstrate the skill's output format and how the
> validation block flags a discrepancy. Not a real invoice.

## Source document (paraphrased)

```
FOUNDRY INDUSTRIAL SUPPLY LTD                         INVOICE
Unit 7, Easton Trade Park, Birmingham B6 7EU, UK
VAT Reg: GB 482 7211 09          accounts@foundrysupply.co.uk

Invoice No: FIS-88204             Issue date: 12/03/2025
Bill to:  Halcyon Manufacturing   Due date:   11/04/2025
          14 Kiln Lane, Sheffield S9 1XH
PO: HM-5567

#  Description                  Qty   Unit price   VAT    Amount
1  Stainless sheet 2mm (m²)      40      18.00     20%    720.00
2  Mild steel angle 50mm (3m)    60       9.50     20%    570.00
3  Welding rods E6013 (5kg box)  25      14.00      5%    350.00
4  Cutting discs 230mm (pk/10)   30       7.20     20%    216.00

                              Subtotal (net)            1,900.00
                              Trade discount  -5%          -92.80
                              VAT                          335.04
                              Total due                  2,142.24

Terms: Net 30   ·   Pay by bank transfer
IBAN: GB29 NWBK 6016 1331 9268 19   ·   BIC: NWBKGB2L
```

## JSON output

```json
{
  "document_type": "invoice",
  "currency": "GBP",
  "vendor": {
    "name": "Foundry Industrial Supply Ltd",
    "address": "Unit 7, Easton Trade Park, Birmingham B6 7EU, UK",
    "tax_id": "GB 482 7211 09",
    "email": "accounts@foundrysupply.co.uk",
    "phone": null
  },
  "bill_to": { "name": "Halcyon Manufacturing", "address": "14 Kiln Lane, Sheffield S9 1XH" },
  "ship_to": { "name": null, "address": null },
  "invoice_number": "FIS-88204",
  "po_reference": "HM-5567",
  "issue_date": "2025-03-12",
  "due_date": "2025-04-11",
  "line_items": [
    { "description": "Stainless sheet 2mm (m²)", "quantity": 40, "unit_price": 18.00, "line_amount": 720.00, "tax_rate": "20%", "tax_category": "VAT" },
    { "description": "Mild steel angle 50mm (3m)", "quantity": 60, "unit_price": 9.50, "line_amount": 570.00, "tax_rate": "20%", "tax_category": "VAT" },
    { "description": "Welding rods E6013 (5kg box)", "quantity": 25, "unit_price": 14.00, "line_amount": 350.00, "tax_rate": "5%", "tax_category": "VAT" },
    { "description": "Cutting discs 230mm (pk/10)", "quantity": 30, "unit_price": 7.20, "line_amount": 216.00, "tax_rate": "20%", "tax_category": "VAT" }
  ],
  "totals": {
    "subtotal": 1900.00,
    "discount": 92.80,
    "tax": 335.04,
    "shipping": 0.00,
    "grand_total": 2142.24,
    "amount_paid": 0.00,
    "amount_due": 2142.24
  },
  "payment": {
    "terms": "Net 30",
    "methods": ["bank transfer"],
    "bank": {
      "bank_name": null,
      "iban": "GB29 NWBK 6016 1331 9268 19",
      "account_number": null,
      "routing_or_sort_code": null,
      "swift_bic": "NWBKGB2L"
    }
  },
  "validation": [
    { "check": "line_items_sum_to_subtotal", "status": "fail", "expected": 1856.00, "actual": 1900.00, "discrepancy": 44.00 },
    { "check": "line_math_quantity_times_unit_price", "status": "pass", "expected": 1856.00, "actual": 1856.00, "discrepancy": 0.00 },
    { "check": "subtotal_minus_discount_plus_tax_plus_shipping_equals_total", "status": "pass", "expected": 2142.24, "actual": 2142.24, "discrepancy": 0.00 },
    { "check": "amount_due_equals_total_minus_paid", "status": "pass", "expected": 2142.24, "actual": 2142.24, "discrepancy": 0.00 }
  ],
  "confidence": [
    { "field": "totals.subtotal", "level": "low", "note": "Line items sum to 1,856.00 (720+570+350+216) but the printed subtotal is 1,900.00 — a 44.00 discrepancy. Printed subtotal preserved; flagged for review. The rest of the document (discount, VAT, total) is internally consistent against the printed 1,900.00 subtotal." },
    { "field": "issue_date", "level": "medium", "note": "Source date 12/03/2025 read as DD/MM/YYYY (2025-03-12) per UK locale (GBP, GB VAT). If US MM/DD, this would be 2025-12-03." }
  ]
}
```

## Summary

Invoice **FIS-88204** from **Foundry Industrial Supply Ltd** to Halcyon
Manufacturing, **GBP 2,142.24** due **2025-04-11** on **Net 30** terms. Tax is
charged per line (20% on three lines, 5% on the welding rods). **One validation
check failed:** the printed subtotal does not match the sum of the line items.

## Flagged issues

1. **Subtotal mismatch (material).** The four line amounts sum to **1,856.00**,
   but the invoice prints a subtotal of **1,900.00** — a **+44.00** discrepancy.
   The discount, VAT, and total all reconcile against the *printed* 1,900.00, so
   either a line item is missing/understated or the printed subtotal is wrong.
   The number was **not** altered — confirm against the source before posting.
2. **Ambiguous date format (minor).** `12/03/2025` was read as 12 March 2025
   (UK locale); verify if the supplier uses US conventions.

*Illustrative; verify every flagged value against the source document before
posting or paying.*
