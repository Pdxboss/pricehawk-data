# PriceHawk Data

Public structured-data repository for **PriceHawk — Retail Deal Intelligence**.

## Purpose

PriceHawk separates routine deal intelligence from application code. Normal ChatGPT scanners and conversations update the files in this repository. The PriceHawk site reads the structured data and renders it.

**Routine data flow:**

`scanner / ChatGPT -> this repository -> PriceHawk site`

ChatGPT Work should not be required for routine deal updates.

## Repository layout

- `data/deals.json` — current deal opportunities and their evidence
- `data/stores.json` — persistent public store directory
- `data/activity.json` — audit trail of meaningful changes
- `schemas/deal.schema.json` — machine-readable deal schema
- `schemas/store.schema.json` — machine-readable store schema
- `schemas/activity.schema.json` — machine-readable activity schema
- `docs/DATA_CONTRACT.md` — field definitions, truth layers, provenance, and mutation rules
- `docs/SCANNER_PLAYBOOK.md` — discovery, deduplication, confidence, verification, expiration, and alert logic

## Routine operations

Routine maintenance is limited to four operations:

1. **CREATE** — add a genuinely new opportunity
2. **UPDATE** — add fresh evidence or change current best-known values while preserving history
3. **VERIFY** — record a valid local price/store confirmation
4. **EXPIRE** — mark an opportunity no longer actionable without deleting history

Before writing data, read `docs/DATA_CONTRACT.md` and `docs/SCANNER_PLAYBOOK.md`.

## Public-data rule

This repository is intentionally public because the PriceHawk browser client will read it without exposing credentials.

Do **not** store private information here, including receipts, serial numbers, IMEIs, account data, private financial history, private purchase history, personal notes, API keys, tokens, or secrets.

## Governing principle

PriceHawk must clearly distinguish:

1. a deal reported somewhere,
2. a local inventory/store signal, and
3. a locally verified price.

Never promote a deal to locally verified without actual local confirmation.
