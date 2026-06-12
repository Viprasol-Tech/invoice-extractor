# Invoice Extractor — Agent Skill

> Turn any invoice, receipt, or bill into clean, validated JSON — vendor, line
> items, totals, tax, and payment terms — with the math checked and
> discrepancies flagged.

[![GitHub stars](https://img.shields.io/github/stars/Viprasol-Tech/invoice-extractor?style=social)](https://github.com/Viprasol-Tech/invoice-extractor)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Agent Skills](https://img.shields.io/badge/Agent%20Skills-Compatible-blueviolet)](https://github.com/Viprasol-Tech/invoice-extractor)
[![Output](https://img.shields.io/badge/Output-Validated%20JSON-green)](skill.md)
[![Validates](https://img.shields.io/badge/Math%20checked-Reconciled-blue)](skill.md)
[![Version](https://img.shields.io/badge/version-1.0.0-informational)](CHANGELOG.md)

**Works with:** Claude Code · OpenAI Codex · Cursor · GitHub Copilot · Gemini
CLI · 26+ tools that support portable agent skills.

---

## Quick Install

Clone the skill into your agent's skills directory:

```bash
# Claude Code
git clone https://github.com/Viprasol-Tech/invoice-extractor \
  ~/.claude/skills/invoice-extractor

# Cursor / Codex / other tools — clone anywhere your tool loads skills from, e.g.
git clone https://github.com/Viprasol-Tech/invoice-extractor \
  ~/.config/skills/invoice-extractor
```

The skill is a single self-contained `skill.md` (plus examples) — no
dependencies, no install step, nothing to build.

## Try It

Share an invoice, receipt, or bill (PDF, image, or pasted text) and ask:

```
Extract this invoice into JSON and tell me if it adds up.
```

The skill will classify the document, pull the vendor, line items, totals, tax,
and payment terms into a strict JSON object, normalize dates/amounts/currency,
reconcile the arithmetic, and return the JSON plus a short summary and a
flagged-issues list.

---

## Why This Exists

Keying invoices into a ledger by hand is slow, error-prone, and exactly the kind
of repetitive work an agent should do — but a naive "read the numbers" pass is
dangerous: OCR drops a digit, a stamped "PAID" overlaps a total, the subtotal
doesn't actually match the line items, and the bad number flows straight into
your books. This skill encodes the disciplined accounts-payable checklist:
classify the document, extract a fixed field set, normalize formats, and then
**reconcile the math** — line items to subtotal, subtotal to total, due to total
minus paid — flagging every discrepancy instead of silently "fixing" it.

## What It Does

1. **Identifies** the document type (invoice / receipt / PO / credit note /
   proforma) and notes quality (scanned, photo, handwritten, multi-page).
2. **Extracts** a fixed field set — vendor, bill-to, invoice number, dates,
   line items, totals, tax, and payment terms — using `null` for anything
   absent.
3. **Normalizes** dates to ISO 8601, amounts to plain decimals, currency to ISO
   4217, and maps synonyms ("Balance Due" → `amount_due`) to canonical fields.
4. **Reconciles** the arithmetic — line items → subtotal, subtotal − discount +
   tax + shipping → total, tax ≈ subtotal × rate, amount due → total − paid.
5. **Flags** every failed check and every low-confidence field instead of
   altering the numbers to balance.
6. **Outputs** a strict JSON object, a short human-readable summary, and a
   prioritized flagged-issues list.

## Features

- **Strict, predictable JSON** — a fixed shape with `vendor`, `bill_to`,
  `line_items[]`, `totals`, `payment`, `validation[]`, and per-field
  `confidence` so downstream code can rely on it.
- **Built-in reconciliation** — every invoice runs the full set of arithmetic
  checks with a rounding tolerance, emitting `expected` / `actual` /
  `discrepancy` on any mismatch.
- **Never silently "fixes" numbers** — a subtotal that doesn't match its line
  items is flagged, not overwritten.
- **Normalization that preserves ambiguity** — ISO dates and decimals, ISO 4217
  currency, locale-aware thousands/decimal handling, with the raw string kept
  whenever a transform could be wrong.
- **Confidence per field** — low-confidence reads (OCR ambiguity, handwriting,
  ambiguous dates) are surfaced for human review, not buried.
- **Handles the whole document family** — invoices, receipts, purchase orders,
  credit notes, and proformas, including multi-page and multi-currency.

### Output at a glance

| Block | What it holds |
|---|---|
| `vendor` / `bill_to` / `ship_to` | Names, addresses, tax/VAT id |
| `line_items[]` | description, quantity, unit_price, line_amount, tax_rate |
| `totals` | subtotal, discount, tax, shipping, grand_total, amount_paid, amount_due |
| `payment` | terms, methods, bank (IBAN / routing / SWIFT) |
| `validation[]` | each check: `pass` / `fail` / `skipped` + discrepancy |
| `confidence[]` | low/medium-confidence fields flagged for review |

*Numbers are JSON numbers, dates are ISO 8601, currency is an ISO 4217 code.*

See full worked reports in **[examples/](examples/)**:
- [SaaS subscription invoice](examples/saas-invoice.md) — a clean invoice where
  every validation check passes.
- [Multi-line goods invoice](examples/multi-line-goods-invoice.md) — per-line
  tax and a discount, with a deliberate subtotal mismatch the validation block
  flags.

---

## Verify Before You Pay

This skill is a **document data-entry aid, not financial, accounting, or legal
advice.** Extraction quality depends on the source — OCR misreads, skewed scans,
handwriting, and layout ambiguity all cause errors. Always verify extracted
values against the source document before posting to a ledger or paying, and
treat any low-confidence field as needing human review. The validation block
flags arithmetic discrepancies; it does not certify that a document is genuine
or that a charge is correct. **Not affiliated with or endorsed by Anthropic.**

---

## Contact — Viprasol Tech Private Limited
- 🌐 Website: [viprasol.com](https://viprasol.com)
- ✉️ Email: [support@viprasol.com](mailto:support@viprasol.com)
- 💬 Telegram: [t.me/viprasol_help](https://t.me/viprasol_help) · 📱 WhatsApp: +91 96336 52112
- 🐙 GitHub: [@Viprasol-Tech](https://github.com/Viprasol-Tech) · 💼 [LinkedIn](https://www.linkedin.com/in/viprasol/) · 𝕏 [@viprasol](https://twitter.com/viprasol)

> *Viprasol Tech — fintech software, AI agents, algorithmic trading systems, and B2B SaaS.*

## License
[MIT](LICENSE) © 2025 Viprasol Tech Private Limited
