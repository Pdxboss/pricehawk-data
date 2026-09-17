# PriceHawk Site Integration

PriceHawk's public site should read its shared intelligence at runtime from this repository. Routine data changes must not require a ChatGPT Sites redeploy.

## Canonical public data URLs

- Deals: `https://raw.githubusercontent.com/Pdxboss/pricehawk-data/main/data/deals.json`
- Stores: `https://raw.githubusercontent.com/Pdxboss/pricehawk-data/main/data/stores.json`
- Activity: `https://raw.githubusercontent.com/Pdxboss/pricehawk-data/main/data/activity.json`

Do not put credentials or GitHub tokens in browser JavaScript. These files are intentionally public.

## Fetch behavior

The browser should request the three canonical JSON files at runtime. Use cache-busting or an equivalent no-stale strategy so PriceHawk does not continue showing old deal intelligence after a GitHub commit.

A safe pattern is to append a changing query parameter and request with cache disabled, for example:

```js
const bust = Date.now();
const url = `https://raw.githubusercontent.com/Pdxboss/pricehawk-data/main/data/deals.json?v=${bust}`;
const response = await fetch(url, { cache: 'no-store' });
```

The exact implementation may differ, but the functional requirement is that refreshing PriceHawk after a GitHub data commit retrieves the new JSON without republishing the site.

## Failure behavior

The site must not break if GitHub is temporarily unavailable or JSON is malformed.

Preferred behavior:

1. Try the external GitHub source.
2. If successful, render the external data and show its `updated_at` timestamp.
3. If it fails, use the bundled V1 JSON only as a fallback if available.
4. Clearly indicate that fallback/stale data is being shown.
5. Never silently present fallback data as current.

## Scope

Externalize only shared public intelligence for V1:

- `deals.json`
- `stores.json`
- `activity.json`

Keep private purchase/inventory information out of this public repository. The V1 browser-local inventory behavior can remain unchanged until a private data layer is intentionally designed.

## Acceptance test

1. Load PriceHawk and record a visible field from a deal.
2. Modify only `data/deals.json` in this repository.
3. Commit the change to `main`.
4. Refresh PriceHawk.
5. The changed field must appear without a ChatGPT Work/Sites rebuild or deployment.
6. Restore the intended value if the test mutation was artificial.

If this test succeeds, routine PriceHawk intelligence is decoupled from Work deployments.
