---
name: invoapi
description: Create, validate, read and convert German and European e-invoices (XRechnung, ZUGFeRD / Factur-X, EN 16931, UBL, CII, Peppol BIS) with the invoapi REST API. Use when the user wants to generate an e-invoice or E-Rechnung from data, check an incoming XML or ZUGFeRD PDF against the official rules (KoSIT validator, EN 16931 schematron), fix validation errors such as BR-DE-15 or BR-CO-25, extract invoice data from XML/PDF, convert between formats, or integrate e-invoicing into code (shop, ERP, CRM, script).
---

# invoapi: e-invoices from JSON

invoapi turns one JSON invoice model into any supported e-invoice format and validates documents with the official rule sets. Totals and the VAT breakdown are always calculated server-side. Invoice contents are processed per request and not stored.

- API base: `https://api.invoapi.de`
- Docs: https://invoapi.de/docs (OpenAPI: https://api.invoapi.de/openapi.json)
- Auth: `Authorization: Bearer $INVOAPI_API_KEY` (or header `X-API-Key`). Keys start with `inv_live_` or `inv_test_` and are created at https://invoapi.de/dashboard/api-keys.

Never print, log or commit the API key. Read it from the environment (`INVOAPI_API_KEY`); if it is missing, ask the user to create one in the dashboard and set the variable.

## Choose the format

| Need | Request |
|---|---|
| German public sector buyer (Behörde), Leitweg-ID | `"format": "xrechnung"` (UBL default, `"syntax": "cii"` possible) |
| B2B invoice a human also reads | `"format": "zugferd", "output": "pdf", "profile": "en16931"` (PDF/A-3 with embedded XML) |
| Cross-border / Peppol network | `"format": "peppol"` |
| Plain EN 16931 | `"format": "en16931"` |

## Endpoints

| Method | Path | Purpose |
|---|---|---|
| POST | `/api/v1/generate` | JSON invoice to XML or ZUGFeRD PDF, validated by default |
| POST | `/api/v1/validate` | Check XML / ZUGFeRD PDF (raw `application/xml`, `application/pdf`, multipart `file`, or JSON with `xml` / `content_base64` / `invoice`) |
| POST | `/api/v1/parse` | Any supported document into the canonical JSON model |
| POST | `/api/v1/convert` | Between JSON, UBL, CII, HTML, PDF (`target`: xml, pdf, json, html) |
| POST | `/api/v1/render` | Readable HTML or PDF of an XML invoice (returns the document itself) |
| POST | `/api/v1/detect` | Identify format, syntax and version without validating |
| POST | `/api/v1/calculate` | Totals and VAT breakdown only, no document |
| GET | `/api/v1/formats` | Supported formats and versions |

## Workflow for generating

1. Build the canonical invoice (see [references/invoice-model.md](references/invoice-model.md)). Required: `invoice_number`, `issue_date`, `seller`, `buyer`; add `line_items` and `payment`.
2. `POST /api/v1/generate` with `{"format": ..., "invoice": {...}}`. Validation runs by default (`"validate": true`).
3. Read the response:
   - `valid: true`: done. XML is in `document.content`, PDF in `document.content_base64`, name in `document.filename`.
   - `valid: false`: fix every issue with `severity: "error"` in `validation.issues`. Each issue names the `rule`, the canonical `field`, an `explanation` and a `suggestion`. Regenerate. See [references/errors.md](references/errors.md).
   - `valid: null`: no rule set could run. Never treat that as valid.
4. Also read `warnings`: the API reports values it filled in (for example the buyer reference fallback).

Do not compute totals yourself and do not send `totals` or `taxes`: the API recalculates them with exact decimal arithmetic. Use `/calculate` to preview amounts.

## Minimal XRechnung request

```bash
curl -s https://api.invoapi.de/api/v1/generate \
  -H "Authorization: Bearer $INVOAPI_API_KEY" -H "Content-Type: application/json" \
  -d @- <<'JSON'
{
  "format": "xrechnung",
  "invoice": {
    "invoice_number": "RE-2026-0001",
    "issue_date": "2026-10-29",
    "due_date": "2026-11-12",
    "buyer_reference": "04011000-12345-34",
    "seller": {
      "name": "Muster GmbH", "vat_id": "DE123456789",
      "email": "rechnung@muster.de", "phone": "+49 30 1234567",
      "address": {"street": "Musterstraße 1", "city": "Berlin", "postal_code": "10115", "country_code": "DE"}
    },
    "buyer": {
      "name": "Beispiel GmbH",
      "electronic_address": "einkauf@beispiel.de", "electronic_address_scheme": "EM",
      "address": {"street": "Teststraße 10", "city": "Hamburg", "postal_code": "20095", "country_code": "DE"}
    },
    "line_items": [
      {"name": "Beratung", "quantity": 8, "unit_code": "HUR", "unit_price": 145, "vat_category": "S", "vat_rate": 19}
    ],
    "payment": {"means_code": "58", "iban": "DE02120300000000202051"}
  }
}
JSON
```

More examples (ZUGFeRD PDF with logo, validating a file, parsing, converting): [references/examples.md](references/examples.md).

## Limits and errors

- HTTP errors use `{"error": {"code", "message", "details", "request_id"}}`.
- `402 QUOTA_EXHAUSTED`: the free plan's 100 invoices are used up; the user can upgrade at https://invoapi.de/preise.
- `429 RATE_LIMITED`: wait for the `Retry-After` header, then retry.
- `401`: wrong key, unconfirmed e-mail or disabled account.

## Other ways in

- MCP server for AI assistants: `https://api.invoapi.de/mcp` (OAuth). Setup: https://invoapi.de/mcp
- PHP SDK, n8n node, WooCommerce plugin: https://invoapi.de/integrationen
