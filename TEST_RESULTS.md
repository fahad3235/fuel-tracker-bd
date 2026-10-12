# fuel-tracker-bd \u2014 Test Results

Run #40 (13:00 slot, 2026-10-12). Everything below is real output from real runs.
All sources are pure ASCII: every Bengali string is emitted as `\uXXXX`
escapes by the build script (`/tmp/fuelbd/build.cjs`), so the push API's
ASCII-sensitive argv transport cannot corrupt them.

## File sizes

| File | Bytes |
|---|---|
| index.html | 37,810 |
| README.md | 3,973 |
| .gitignore | 123 |
| core.js (test support, local only) | 5,638 |

ASCII audit: `grep -rP '[^\x00-\x7F]' index.html README.md .gitignore` ->
zero matches.

## Unit tests \u2014 44/44 PASS (2026-10-12)

`node tests/unit.test.js` \u2014 pure math core (`core.js`), no DOM. Every expected
value was hand-computed before the run.

- sortRefills: unsorted input orders by date (r1,r2,r3).
- perRefillMileage: (1250-1000)/25 = 10.0 ; (1520-1250)/24 = 11.25 ; first = null.
- lifetimeMileage: 520 km / 49 L = 10.612244897959184 (within 1e-9); single/empty -> null.
- bestWorstMileage: best 11.25 / worst 10.0 ; empty -> {best:null, worst:null}.
- costPerKm: 8625 / 520 = 16.58653846153846 (within 1e-9); no-km -> null.
- totalSpend: 2500+3125+3000 = 8625.
- unitPrice: 3125/25 = 125.0 ; qty 0 -> null.
- last30DaysSpend: now 2026-10-12 -> 8625 (all inside); now 2026-11-15 -> 0
  (none inside); refill exactly on the cutoff day 2026-09-12 counts (1200).
- monthlySpend (ref 2026-10-12): 12 months Nov-2025..Oct-2026; Oct 2026 = 8625,
  Sep 2026 = 0; label format "10/26".
- bnDigits("123.45") = "\u09e7\u09e8\u09e9.\u09ea\u09eb"; commas preserved; fmt(8625,2) = "8,625.00";
  fmt(null,2) = "\u2014".
- validateRefill (today 2026-10-12): future date -> err_future; odo == last or
  lower -> err_odo; first-refill odo 0 allowed; negative odo -> err_odo_num;
  qty 0 -> err_qty; cost -5 -> err_cost; empty date -> err_date; valid -> [].
- validateImport: valid -> null; rejects garbage ("not-an-object"),
  missing arrays, blank name, bad fuel type, orphan refill, zero-qty refill.

## Smoke tests \u2014 48/48 PASS (2026-10-12)

`node tests/smoke.mjs` \u2014 real headless Chromium (puppeteer-core +
/opt/meta-chromium/chrome), `file://` URL with `?asof=2026-10-12` for
deterministic "today".

- Initial render: heading "Fuel Tracker BD", lang button shows the Bengali
  label, vehicles empty state visible, refill form hidden until a vehicle exists.
- Add vehicle "My Bike" (Octane): row renders, active badge, qty label
  "Quantity (litres)".
- 3 refills added via the real form; table rows hand-verified:
  row1 mileage "\u2014" (no previous), row2 "10.00 km/L", row3 "11.25 km/L",
  row2 unit price 125.00.
- Stats: lifetime avg "10.61 km/L" (520/49), best "11.25 km/L",
  worst "10.00 km/L", count 3.
- Cost dashboard: cost/km "16.59" (8625/520), total "8,625.00",
  last-30-day "8,625.00", SVG chart renders with "10/26" label.
- Validation fires inline: future date (2026-10-13) shows future-date error;
  odo 1520 (not increasing) cites last reading 1,520; qty 0 shows Quantity
  error; row count stays 3 after the three rejected adds.
- Export downloads a valid JSON (1 vehicle, 3 refills) -> clear-all wipes to
  empty -> import restores everything, "Import successful.", avg still 10.61.
- Malformed import ("{"junk": true") rejected with an error and all 3 rows
  intact.
- BN toggle: heading becomes "\u09ab\u09c1\u09af\u09bc\u09c7\u09b2 \u099f\u09cd\u09b0\u09cd\u09af\u09be\u0995\u09be\u09b0", avg renders "\u09e7\u09e6.\u09ec\u09e7 \u0995\u09bf\u09ae\u09bf/\u09b2\u09bf\u099f\u09be\u09b0";
  reload keeps BN language AND all data (localStorage persistence verified).
- 390px viewport: documentElement.scrollWidth <= 390 (no horizontal overflow).
- Second vehicle (CNG): qty label switches to the Bengali "\u09aa\u09b0\u09bf\u09ae\u09be\u09a3 (\u0995\u09c7\u099c\u09bf)"
  (page was in BN mode) \u2014 km/kg unit path exercised.
- 0 console errors, 0 page errors, 0 non-file network requests across the run.

## Repairs

- Repair cycle 1 (test-side only, 0 app bugs): two smoke expectations were
  wrong, not the app \u2014 (a) the CNG qty label was asserted in English while the
  page was in BN mode after the reload-persistence check; (b) the network
  counter included the two `file://` page loads (initial + reload) \u2014 counting
  only non-`file:`/non-`data:` requests gives 0. Both fixed in the harness;
  re-run 48/48 PASS.

No app logic, CSS, or markup changes were needed after the first smoke run.
