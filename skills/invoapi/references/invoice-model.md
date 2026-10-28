# The canonical invoice model

One JSON object for every format. Field names map to EN 16931 business terms (BT-…). Dates are `YYYY-MM-DD`; amounts are numbers or decimal strings. The full schema is `CanonicalInvoice` in https://api.invoapi.de/openapi.json.

## Document

| Field | Term | Notes |
|---|---|---|
| `invoice_number` | BT-1 | required |
| `issue_date` | BT-2 | required |
| `type` | | `invoice` (default) or `credit_note` |
| `due_date` | BT-9 | or `payment.terms`; one of them is required when money is due (BR-CO-25) |
| `currency` | BT-5 | default `EUR` |
| `buyer_reference` | BT-10 | Leitweg-ID for German public buyers; XRechnung needs it (BR-DE-15), otherwise the invoice number is used and a warning returned |
| `purchase_order_reference` | BT-13 | buyer's order number |
| `sales_order_reference` | BT-14 | seller's order number |
| `contract_reference`, `project_reference` | BT-12, BT-11 | |
| `invoice_period_start`, `invoice_period_end` | BT-73, BT-74 | service period |
| `preceding_invoice_references` | BG-3 | `[{"id", "issue_date"}]`, required for credit notes |
| `notes` | BG-1 | `[{"text"}]`, e.g. reverse charge or § 19 UStG note |
| `prepaid_amount` | BT-113 | already paid; amount due becomes total minus this |
| `delivery` | BG-13 | `{"delivery_date", "party_name", "address"}` |
| `allowances`, `charges` | BG-20, BG-21 | `[{"amount", "reason", "vat_category", "vat_rate"}]` |

## Party (`seller`, `buyer`, optional `payee`)

| Field | Term | Notes |
|---|---|---|
| `name` | BT-27/44 | required |
| `vat_id` | BT-31/48 | USt-IdNr., e.g. `DE123456789` |
| `tax_number` | BT-32 | Steuernummer, when the seller has no VAT ID |
| `company_id` | BT-30/47 | e.g. HRB number |
| `additional_legal_information` | BT-33 | e.g. register court, managing director |
| `email`, `phone` | | used to complete the seller contact |
| `electronic_address` + `electronic_address_scheme` | BT-34/49 | buyer: required for XRechnung (scheme `EM` for e-mail, `0204` for Leitweg-ID). Seller: filled from `email` when omitted |
| `address` | BG-5/8 | `street`, `additional_street`, `city`, `postal_code`, `state`, `country_code` (ISO alpha-2, default `DE`) |
| `contact` | BG-6/9 | `name`, `phone`, `email`; XRechnung requires seller contact phone and e-mail |

## Line item (`line_items[]`)

| Field | Term | Notes |
|---|---|---|
| `name` | BT-153 | required |
| `quantity` | BT-129 | default 1 |
| `unit_code` | BT-130 | UN/ECE Rec 20: `C62` piece (default), `H87` piece, `HUR` hour, `DAY`, `KGM`, `MTR`, `LS` lump sum |
| `unit_price` | BT-146 | net price per unit |
| `vat_category` | BT-151 | see below, default `S` |
| `vat_rate` | BT-152 | e.g. 19, 7, 0 |
| `description`, `seller_item_id`, `discount`, `charge`, `period_start`, `period_end` | | optional |

VAT categories: `S` standard rate, `Z` zero rated, `E` exempt (e.g. § 19 UStG small business, add a note), `AE` reverse charge (buyer in another EU country with VAT ID, rate 0), `K` intra-community supply, `G` export outside the EU, `O` not subject to VAT, `L`/`M` Canary Islands / Ceuta-Melilla.

## Payment (`payment`)

| Field | Term | Notes |
|---|---|---|
| `means_code` | BT-81 | UNTDID 4461: `58` SEPA transfer (default, needs `iban`), `59` SEPA direct debit (needs `mandate_reference`), `30` transfer, `48` card, `54` credit card, `68` online payment, `10` cash |
| `iban`, `bic`, `account_name` | BT-84, BT-86, BT-85 | |
| `payment_reference` | BT-83 | Verwendungszweck |
| `terms` | BT-20 | e.g. "Zahlbar innerhalb von 14 Tagen ohne Abzug." |

## Paid orders

For an invoice that was already paid (card, PayPal): set `prepaid_amount` to the total, put "Bereits bezahlt am …" into `payment.terms`, and leave out `due_date`. The amount due is then 0.
