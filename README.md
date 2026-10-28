# invoapi Agent Skills

Agent Skills for [invoapi](https://invoapi.de), the API for German and European e-invoices (XRechnung, ZUGFeRD / Factur-X, EN 16931, UBL, CII, Peppol BIS).

With the skill installed, coding agents such as Claude Code, Cursor, Codex or GitHub Copilot can generate e-invoices from your data, validate incoming XML and ZUGFeRD PDFs against the official rules (KoSIT validator, EN 16931 schematron), explain and fix validation errors, and wire invoapi into a shop, ERP or script.

## Install

```bash
npx skills add invoapi/skills
```

Add `-g` to install it for all projects. Then set your API key, created at [invoapi.de/dashboard/api-keys](https://invoapi.de/dashboard/api-keys):

```bash
export INVOAPI_API_KEY=inv_live_...
```

The skill reads the key from the environment only and never writes it anywhere.

## Skills

| Skill | What it does |
|---|---|
| [invoapi](skills/invoapi/SKILL.md) | Generate, validate, parse and convert e-invoices with the invoapi REST API |

## Links

- Documentation: https://invoapi.de/docs
- MCP server for AI assistants: https://invoapi.de/mcp
- All integrations: https://invoapi.de/integrationen
