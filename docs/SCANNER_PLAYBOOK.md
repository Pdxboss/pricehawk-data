# PriceHawk Scanner Playbook

This file defines how normal ChatGPT scanners should discover, evaluate, and maintain PriceHawk opportunities without modifying application code.

## Goal

PriceHawk scanners should answer one question:

> Is this opportunity real enough, fresh enough, local enough, and profitable enough to deserve the user's attention?

The scanner is not a generic deals feed. It should reject noise.

## Current focus

### Target electronics scanner

Priority categories:

- Apple products, especially iPhone 16 / iPhone 16 Pro
- gaming hardware and games
- TVs
- headphones
- computers
- cameras
- smart-home products

Default threshold: prioritize deals at least 40% below normal retail, but allow strategically important products through when likely profit is strong.

### Power tool scanner

Priority brands:

- Milwaukee
- DeWalt
- Makita
- Ridgid
- Bosch
- Metabo HPT

Priority retailers:

- Home Depot
- Lowe's
- Acme Tools and other major authorized retailers when useful

Prefer opportunities with at least 40% expected ROI or at least $75 expected profit when the evidence supports a resale estimate.

## Geographic weighting

Give extra weight to evidence relevant to:

- Portland
- Beaverton
- Hillsboro
- Tigard
- Fairview
- Hayden Island
- Vancouver, WA

National reports remain useful for discovering what to look for, but they do not become local verification automatically.

## Discovery sources

Useful public sources include:

- retailer product pages
- Reddit deal communities
- Slickdeals
- publicly indexed social posts
- credible deal communities

Preserve direct source URLs whenever possible.

## Extraction checklist

For every potential opportunity, extract as many supported fields as possible:

- exact product name
- retailer
- brand/category
- regular/reference price
- reported deal price
- discount percentage
- DPCI / TCIN / UPC / SKU / model
- carrier/lock status when applicable
- report location
- report time
- source URL
- evidence type
- local inventory/store signals when actually available

Use `null` for unknown fields. Do not invent missing identifiers or prices.

## Deduplication

Before CREATE, inspect current `data/deals.json`.

Match in this order:

1. UPC
2. retailer SKU / DPCI
3. model
4. normalized product identity

If the item already exists, perform UPDATE instead of CREATE.

## CREATE

Create a new deal when:

- product identity is sufficiently clear,
- the opportunity is relevant to PriceHawk,
- the evidence is current enough to matter,
- and no matching deal already exists.

A new deal should always include at least one source and timestamps.

## UPDATE

Update an existing deal when new evidence materially changes or strengthens the opportunity.

Examples:

- additional independent confirmation
- deeper reported markdown
- new regional evidence
- newly discovered exact identifier
- local inventory/store signal
- better resale evidence

When updating:

- preserve prior sources,
- append new price history when price changes,
- update `last_seen`,
- recalculate derived fields only when supported,
- add an activity event for user-relevant changes.

Do not create activity spam for insignificant metadata cleanup.

## VERIFY

A local deal becomes verified only when the exact local price has been credibly confirmed.

Accepted verification examples:

- employee provides scan price over the phone
- in-person scanner/register result
- receipt
- shelf tag when clearly tied to the exact item
- another reliable direct local confirmation

Inventory language alone is not verification.

When VERIFY occurs:

- update/add the correct `store_signals` entry,
- set `observed_price`,
- set `verified: true`,
- set `verification_type`,
- set `checked_at`,
- set top-level `local_verification: true`,
- normally set `status: verified`,
- append price history if appropriate,
- create a `local_verified` activity event.

## EXPIRE

Set `status: dead` when the opportunity is no longer reasonably actionable, for example:

- retailer markdown ended
- repeated fresh checks show normal price
- item is no longer available and no useful inventory signal remains
- original report proved incorrect
- resale economics deteriorated enough to fail the user's thresholds

Do not delete the historical record merely because the deal died.

## Confidence logic

Confidence describes whether the reported deal is credibly real somewhere.

### Low

Typical characteristics:

- vague social post
- no exact identifier
- no corroboration
- stale screenshot
- unclear price context

### Medium

Typical characteristics:

- exact product/identifier established
- at least one credible fresh report
- evidence is plausible but not strongly corroborated

### High

Typical characteristics:

- exact identifier confirmed
- multiple independent reports and/or strong receipt/scan evidence
- fresh timestamps
- credible source agreement

A high national confidence deal can still have `local_verification: false`.

## Local store signals

Store availability signals should be factual and timestamped.

Examples:

- Target app: Limited stock
- retailer site: In stock
- third-party checker: possible stock

Preserve the source name and time.

Do not convert an inventory signal into a local price unless the source actually provides a credible price for that store.

## Economics

Do not rank solely by percent off.

When credible resale evidence is available, prioritize:

- realistic quick-sale value
- expected resale value
- expected profit
- expected ROI
- maximum sensible buy price

Avoid treating active asking prices as guaranteed realized prices.

If resale evidence is weak, leave estimates null rather than inventing precision.

## Alerting

Notify the user only for meaningful changes, such as:

- significant new opportunity
- major price drop
- strong new local signal
- local price verification
- evidence that materially upgrades confidence

If an hourly scan finds nothing significant, do not notify.

## Public repo safety

Never write private or sensitive information into this repository.

Do not store:

- receipts
- serial numbers / IMEIs
- user account data
- financial history
- private purchase records
- secrets
- tokens
- private personal notes

## Routine scanner sequence

1. Search fresh sources.
2. Extract supported structured facts.
3. Read current `data/deals.json`.
4. Deduplicate by stable identifiers.
5. Choose CREATE, UPDATE, VERIFY, or EXPIRE.
6. Preserve provenance and history.
7. Update `updated_at` and relevant timestamps.
8. Add activity only for meaningful changes.
9. Validate structure against the documented contract.
10. Commit with a concise message describing the mutation.
11. Notify the user only if the change is actionable or significant.
