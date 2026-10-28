# Examples

All requests need `-H "Authorization: Bearer $INVOAPI_API_KEY"`.

## ZUGFeRD PDF with logo, saved to disk

```bash
LOGO=$(base64 < logo.png | tr -d '\n')
jq -n --arg logo "$LOGO" --slurpfile inv invoice.json \
  '{format: "zugferd", output: "pdf", profile: "en16931", logo: $logo, invoice: $inv[0]}' |
curl -s https://api.invoapi.de/api/v1/generate \
  -H "Authorization: Bearer $INVOAPI_API_KEY" -H "Content-Type: application/json" -d @- > response.json
jq -r '.valid' response.json
jq -r '.document.content_base64' response.json | base64 --decode > "$(jq -r '.document.filename' response.json)"
```

`logo` is optional: PNG or JPEG, base64 or data URI, at most 512 KB, only used for PDF output.

## Validate an incoming invoice

```bash
curl -s https://api.invoapi.de/api/v1/validate \
  -H "Authorization: Bearer $INVOAPI_API_KEY" -F "file=@eingang.xml" |
jq '{valid, errors: [.result.issues[] | select(.severity=="error") | {rule, explanation, field}]}'
```

A ZUGFeRD PDF works the same way (`-F "file=@rechnung.pdf"`).

## Read an invoice into JSON

```bash
curl -s https://api.invoapi.de/api/v1/parse \
  -H "Authorization: Bearer $INVOAPI_API_KEY" -F "file=@rechnung.pdf"
```

## Convert UBL to CII

```bash
jq -n --rawfile xml rechnung-ubl.xml '{xml: $xml, target: "xml", format: "xrechnung", syntax: "cii"}' |
curl -s https://api.invoapi.de/api/v1/convert \
  -H "Authorization: Bearer $INVOAPI_API_KEY" -H "Content-Type: application/json" -d @-
```

Information the target cannot express is listed in `warnings`.

## Python

```python
import os, requests

r = requests.post(
    "https://api.invoapi.de/api/v1/generate",
    headers={"Authorization": f"Bearer {os.environ['INVOAPI_API_KEY']}"},
    json={"format": "xrechnung", "invoice": invoice},
    timeout=90,
)
r.raise_for_status()
body = r.json()
if body["valid"] is not True:
    for issue in body["validation"]["issues"]:
        if issue["severity"] == "error":
            print(issue["rule"], issue.get("field"), issue.get("suggestion"))
else:
    open(body["document"]["filename"], "w", encoding="utf-8").write(body["document"]["content"])
```

## PHP

Use the SDK: https://invoapi.de/integrationen/php

```php
$invoapi = new Invoapi\Client(getenv('INVOAPI_API_KEY'));
$rechnung = $invoapi->generate($daten, format: 'xrechnung');
if ($rechnung->isValid()) {
    $rechnung->saveTo($rechnung->filename());
}
```
