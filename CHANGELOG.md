# Changelog

All notable changes to this project are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2025-06-12

### Added
- Initial release of the **invoice-extractor** Agent Skill.
- `skill.md` methodology with a 5-step workflow: identify document type,
  field extraction, normalization, validation & reconciliation, and a strict
  JSON output format with a human-readable summary and flagged-issues list.
- Document-type classification across **invoice, receipt, purchase order,
  credit note, and proforma**, with quality flags (scanned, photo,
  handwritten, multi-page, multi-currency).
- Full field set — vendor, bill-to/ship-to, invoice number, PO reference,
  dates, line items, totals, tax/VAT, and payment terms (bank/IBAN/routing).
- **Normalization rules** — ISO 8601 dates, decimal amounts with separators
  stripped, ISO 4217 currency, and synonym mapping, preserving original
  strings where ambiguous.
- **Validation & reconciliation** — line items → subtotal, subtotal − discount
  + tax + shipping = total, tax ≈ subtotal × rate, amount due = total − paid,
  each emitted as a `pass`/`fail`/`skipped` check with the discrepancy; numbers
  are flagged, never silently "fixed".
- Strict JSON schema with `line_items`, `totals`, `validation`, and per-field
  `confidence` blocks.
- Two worked example reports: a clean SaaS subscription invoice (all checks
  pass) and a multi-line goods invoice with per-line tax, a discount, and a
  flagged subtotal mismatch.
- "Verify before you pay" disclaimer in both `skill.md` and `README.md`.
- CI validator (`scripts/validate_skill.py`) and GitHub Actions workflow
  (`.github/workflows/validate.yml`).

[1.0.0]: https://github.com/Viprasol-Tech/invoice-extractor/releases/tag/v1.0.0
