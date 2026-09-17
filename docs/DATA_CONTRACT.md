# PriceHawk Data Contract

This document defines the public structured data that powers PriceHawk.

## Core rule

PriceHawk separates three different levels of truth:

1. **Reported deal** — a credible deal occurred somewhere.
2. **Local signal** — a nearby store may have the item or may show an inventory signal.
3. **Local verification** — the exact local price was confirmed by a reliable local check.

Never collapse these into one state. A national deal can be real while the local price is still unknown.

## Files

### `data/deals.json`
Contains current opportunities and the evidence attached to them.

Top-level shape:

```json
{
  "schema_version": "1.0.0",
  "updated_at": "ISO-8601 timestamp",
  "deals": []
}
```

### `data/stores.json`
Persistent public store directory. Deal-specific inventory and price observations do not belong here; they belong in the deal's `store_signals` array.

### `data/activity.json`
Human-readable audit trail of meaningful deal changes.

## Stable identity and deduplication

When deciding whether a discovered item is new or already tracked, use the strongest stable identifier available in this order:

1. UPC
2. retailer-native SKU / Target DPCI
3. model number
4. normalized product identity

Multiple posts about the same physical product should become one deal with multiple sources, not duplicate cards.

## Deal fields

### Identity

- `id` — stable PriceHawk identifier. Prefer a retailer plus retailer-native identifier, e.g. `target-255-14-9998`.
- `retailer` — retailer name.
- `product` — human-readable exact product name.
- `brand` — manufacturer/brand when known.
- `category` — broad useful category.
- `image_url` — optional public image URL.
- `currency` — currently `USD`.

### Opportunity state

Allowed `status` values:

- `actionable` — fresh, significant opportunity with enough evidence to justify attention.
- `verify` — economics/evidence are promising, but an important confirmation is missing.
- `verified` — a relevant local price has been confirmed.
- `watching` — interesting but not yet actionable.
- `aging` — information is becoming stale.
- `dead` — no longer worth acting on.

`local_verification` is a separate boolean. Do not infer a local verification from `status` alone.

### Prices and economics

- `regular_price` — normal or reference retail price when reliably established.
- `reported_price` — current best-supported reported deal price. This is not automatically a local price.
- `discount_pct` — `(regular_price - reported_price) / regular_price * 100` when both prices are known.
- `estimated_resale` — realistic expected resale estimate, not an asking price presented as fact.
- `estimated_profit` — expected profit under the currently documented assumptions.
- `estimated_roi` — expected profit divided by buy cost, expressed as percent.
- `max_buy_price` — highest sensible purchase price under the user's minimum profit/ROI rules.

If the evidence does not support a value, use `null`. Never fabricate a number just to fill the card.

### Retail identifiers

Supported fields:

- `dpci`
- `tcin`
- `upc`
- `sku`
- `model`

Target deals should prioritize DPCI. Home Depot/Lowe's deals should prioritize their relevant SKU/item/model identifiers.

### Confidence and freshness

- `national_confidence` — `low`, `medium`, or `high`. This describes confidence that the reported deal/product identity is real somewhere, not that it is locally available.
- `first_seen` — first PriceHawk discovery time.
- `last_seen` — latest meaningful supporting observation.
- `reported_location` — location attached to the best-known reported deal when available.

Suggested freshness labels in the UI:

- under 6 hours: Fresh
- 6–24 hours: Recent
- 1–3 days: Aging
- 3+ days: Stale

## Sources

Every meaningful claim should retain provenance.

Each source should include:

- `id`
- `type`
- `url`
- `reported_price` when the source supports one
- `location` when known
- `observed_at`
- `evidence` — short factual description of what the source supports
- `is_primary`

Do not silently replace old evidence when a new price appears. Add the new source and preserve the old one.

## Store signals

Each deal may have store-specific observations in `store_signals`.

Supported inventory values:

- `in_stock`
- `limited`
- `out_of_stock`
- `not_sold`
- `unknown`

A store signal should include:

- `store_id`
- `inventory`
- `observed_price`
- `verified`
- `verification_type`
- `checked_at`
- `source`
- `notes`

Allowed verification types:

- `employee_phone_confirmation`
- `in_person_scan`
- `receipt`
- `shelf_tag`
- `other`

### Critical rule

`Limited stock`, `In stock`, or similar availability language is not proof of the local clearance price.

Only set `verified: true` when a reliable local price confirmation exists.

## Price history

Use `price_history` to preserve meaningful price observations instead of overwriting history.

Each entry should include:

- `price`
- `price_type`
- `observed_at`
- `source_id` when applicable

## Activity log

Routine mutations should create an activity event when the change would matter to the user. Examples:

- new deal created
- new source added
- price changed
- local store signal added/changed
- local price verified
- deal expired

## Public-data boundary

This repository is public. Do not store:

- receipts
- serial numbers
- IMEIs
- account information
- private financial history
- private purchase history
- personal notes
- secrets or API credentials

## Mutation operations

Routine ChatGPT/scanner writes should fit one of four operations:

1. **CREATE** — add a genuinely new opportunity.
2. **UPDATE** — add evidence or update current best-known fields while preserving history.
3. **VERIFY** — add a valid local confirmation and update the opportunity state accordingly.
4. **EXPIRE** — mark a deal `dead` while preserving its record and evidence.

Application code should not need to change for any of these operations.
