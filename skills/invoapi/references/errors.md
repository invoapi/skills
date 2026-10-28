# Fixing validation errors

Every issue in `validation.issues` carries `rule`, `severity`, `message`, `explanation`, `suggestion`, and when known `field` (the canonical field to change), `business_term`, `expected` and `actual`. Fix every `severity: "error"`, regenerate, repeat. Warnings do not make a document invalid.

Common XRechnung findings and their fix (all observed on the live API):

| Rule | Meaning | Fix |
|---|---|---|
| BR-DE-6 | Seller contact telephone missing | `seller.phone` or `seller.contact.phone` |
| BR-DE-1 / BR-DE-19 | Payment instructions incomplete | with `means_code` `58` add `payment.iban` |
| BR-CO-25 | Amount due but neither due date nor terms | `due_date` or `payment.terms` |
| PEPPOL-EN16931-R010 | Buyer electronic address missing | `buyer.electronic_address` + `buyer.electronic_address_scheme` (`EM` for e-mail) |
| BR-CO-26 / BR-DE-16 | Seller not identifiable for VAT | `seller.vat_id` or `seller.tax_number` |
| BR-S-02 | Standard-rate line without seller VAT ID | add `seller.vat_id` |
| BR-DE-15 | Buyer reference missing | `buyer_reference`; the API falls back to the invoice number and warns. For public buyers this must be the Leitweg-ID |

Filled in automatically (reported in `warnings`, not errors): the seller contact e-mail from `seller.email`, the seller electronic address from `seller.email`, and the buyer reference fallback above.

## Validating someone else's invoice

`POST /api/v1/validate` with the file. `result.status` is `valid`, `invalid` or `unsupported_validation`. Report the errors to the user with rule, explanation and the affected value; do not claim a document is valid when `valid` is `null`.

## HTTP errors

| Status | `error.code` | Meaning |
|---|---|---|
| 400 | `BAD_REQUEST`, `INVALID_XML`, `UNSUPPORTED_FORMAT`, `UNSUPPORTED_SYNTAX` | malformed request, unreadable XML or an unsupported format; `details` names the problem |
| 401 | `UNAUTHORIZED` | wrong key, e-mail not confirmed, account disabled |
| 402 | `QUOTA_EXHAUSTED` | free allowance used up |
| 413 | `PAYLOAD_TOO_LARGE` | document too big |
| 415 | `UNSUPPORTED_MEDIA_TYPE` | wrong `Content-Type` |
| 422 | `INVALID_INVOICE`, `CONVERSION_FAILED`, `DETECTION_FAILED` | the invoice data cannot be processed; `details` names the fields |
| 429 | `RATE_LIMITED` | retry after `Retry-After` seconds |
| 500 | `GENERATION_NOT_SCHEMA_VALID` | a defect in the service; report `request_id` to invoapi |
| 503 / 504 | `VALIDATOR_UNAVAILABLE`, `VALIDATION_TIMEOUT` | validator briefly unavailable; retry later |
