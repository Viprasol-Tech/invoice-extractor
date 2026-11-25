# Example — SaaS Subscription Invoice (Clean, All Checks Pass)

> **Illustrative example.** "Cloudwright Inc." and all figures below are
> fictional, used to demonstrate the skill's output format. Not a real invoice.

## Source document (paraphrased)

```
CLOUDWRIGHT INC.                              INVOICE
500 Market St, Suite 1200, San Francisco, CA 94105
VAT/Tax ID: US-94-3771208        billing@cloudwright.io

Invoice #: CW-2025-04417            Issue date: 06 May 2025
Bill to:   Northwind Analytics LLC   Due date:   05 Jun 2025
           220 Wacker Dr, Chicago, IL 60606
PO: NW-3391

Description                         Qty   Unit price   Amount
Pro plan — monthly seats            25      40.00      1,000.00
Premium support add-on               1     150.00        150.00
Overage — API calls (per 1k)        80       0.25         20.00

                                   Subtotal           1,170.00
                                   Discount (annual)    -117.00
                                   Sales tax (8.75%)      92.14
                                   Total due          1,145.14

Terms: Net 30    Pay by ACH or card
Bank: First Republic  ·  Routing 321081669  ·  Acct 0042118730
```

## JSON output

```json
{
  "document_type": "invoice",
  "currency": "USD",
  "vendor": {
    "name": "Cloudwright Inc.",
    "address": "500 Market St, Suite 1200, San Francisco, CA 94105",
    "tax_id": "US-94-3771208",
    "email": "billing@cloudwright.io",
    "phone": null
  },
  "bill_to": { "name": "Northwind Analytics LLC", "address": "220 Wacker Dr, Chicago, IL 60606" },
  "ship_to": { "name": null, "address": null },
  "invoice_number": "CW-2025-04417",
  "po_reference": "NW-3391",
  "issue_date": "2025-05-06",
  "due_date": "2025-06-05",
  "line_items": [
    { "description": "Pro plan — monthly seats", "quantity": 25, "unit_price": 40.00, "line_amount": 1000.00, "tax_rate": "8.75%", "tax_category": "Sales tax" },
    { "description": "Premium support add-on", "quantity": 1, "unit_price": 150.00, "line_amount": 150.00, "tax_rate": "8.75%", "tax_category": "Sales tax" },
    { "description": "Overage — API calls (per 1k)", "quantity": 80, "unit_price": 0.25, "line_amount": 20.00, "tax_rate": "8.75%", "tax_category": "Sales tax" }
  ],
  "totals": {
    "subtotal": 1170.00,
    "discount": 117.00,
    "tax": 92.14,
    "shipping": 0.00,
    "grand_total": 1145.14,
    "amount_paid": 0.00,
    "amount_due": 1145.14
  },
  "payment": {
    "terms": "Net 30",
    "methods": ["ACH", "card"],
    "bank": {
      "bank_name": "First Republic",
      "iban": null,
      "account_number": "0042118730",
      "routing_or_sort_code": "321081669",
      "swift_bic": null
    }
  },
  "validation": [
    { "check": "line_items_sum_to_subtotal", "status": "pass", "expected": 1170.00, "actual": 1170.00, "discrepancy": 0.00 },
    { "check": "subtotal_minus_discount_plus_tax_plus_shipping_equals_total", "status": "pass", "expected": 1145.14, "actual": 1145.14, "discrepancy": 0.00 },
    { "check": "tax_equals_subtotal_times_rate", "status": "pass", "expected": 92.14, "actual": 92.14, "discrepancy": 0.00 },
    { "check": "amount_due_equals_total_minus_paid", "status": "pass", "expected": 1145.14, "actual": 1145.14, "discrepancy": 0.00 }
  ]
}
```

Note on the tax check: tax is applied to the post-discount net
(1,170.00 − 117.00 = 1,053.00 × 8.75% = 92.1375, which rounds to the printed
92.14, so `pass`). The line-level `tax_rate` is recorded because a single rate
applies to the whole invoice.

## Summary

Invoice **CW-2025-04417** from **Cloudwright Inc.** to Northwind Analytics LLC,
**USD 1,145.14** due **2025-06-05** on **Net 30** terms. **All validation checks
pass** — line items reconcile to the subtotal, the discount and tax roll up to
the total, and the amount due equals the total (unpaid).

## Flagged issues

None. Document is a clean native invoice; all fields read with high confidence
and the arithmetic reconciles. *Illustrative; verify against the source before
posting.*
