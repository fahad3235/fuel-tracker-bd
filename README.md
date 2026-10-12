# Fuel Tracker BD

An offline, bilingual (English / Bangla) vehicle fuel tracker for Bangladesh.
Log every refill, watch your mileage (km/L, or km/kg for CNG), and see exactly
what each kilometre costs you -- all in a single HTML file with zero network
requests.

## Problem

Vehicle owners in Bangladesh track fuel costs in notebooks or memory. Pump
receipts pile up, mileage drifts silently, and nobody knows their real
cost-per-kilometre. Fuel Tracker BD turns a phone browser into a simple
refill ledger with honest math: per-refill mileage, lifetime averages, and a
cost dashboard -- no accounts, no servers, no internet.

## Features

1. **Vehicle profiles** -- add any number of vehicles with a fuel-type selector
   (Octane / Petrol / Diesel / CNG). Mileage units are km/L, switching to km/kg
   automatically for CNG vehicles.
2. **Refill log** -- date, odometer (km), fuel quantity (L or kg), total cost
   (BDT). Validation: date required and not in the future, odometer must be
   strictly increasing, quantity and cost must be greater than zero. Inline
   error messages, per-row delete.
3. **Mileage stats** -- mileage between consecutive refills (delta km / fuel)
   on every table row, plus lifetime average, best and worst mileage.
4. **Cost dashboard** -- cost per km, total spend, last-30-day spend, refill
   count, and a monthly-spend SVG line chart for the last 12 months.
5. **Export / import / clear** -- full JSON export and import with strict schema
   validation (malformed files are rejected and existing data stays intact),
   plus clear-all behind a confirmation dialog.
6. **English / Bangla toggle** -- persisted in localStorage; Bengali digit
   rendering throughout; responsive 390px mobile layout with no horizontal
   overflow; empty states for "no vehicle" and "no refills".

## Stack

- Single static HTML file: inline CSS + vanilla JavaScript, no frameworks.
- Data: browser localStorage only. Works from `file://` and any static host.
- Node.js (tests only), puppeteer-core + system Chromium (smoke tests only).

## Prerequisites

- A modern browser (Chrome, Edge, Firefox, Safari).
- Node.js 18+ only if you want to run the test suite.

## Install

No install step. Copy `index.html` anywhere and open it:

```sh
# or serve locally
npx serve .
```

## Dev commands

```sh
node tests/unit.test.js     # math-core unit tests (no DOM needed)
node tests/smoke.mjs        # headless-Chrome smoke tests (needs chromium)
```

## Env

None. There is no backend, no API key, and no build step.

## Testing

- `tests/unit.test.js`: pure-logic tests over `core` functions -- mileage
  between refills, lifetime average, best/worst, cost/km, monthly aggregation,
  Bengali-digit conversion, refill validation, import validation. All expected
  values are hand-computed; see `TEST_RESULTS.md`.
- `tests/smoke.mjs`: real headless-Chrome run against `index.html` via
  `file://` -- add vehicle, add 3 refills with hand-checked math, validation
  errors, export -> wipe -> import round-trip, BN toggle + reload persistence,
  390px no-overflow, 0 console/page errors, 0 network requests.

## Deployment (GitHub Pages)

1. Create a repo named `fuel-tracker-bd`.
2. Commit `index.html` (+ README, .gitignore).
3. Enable Pages on the `main` branch (root). The app is live at
   `https://<user>.github.io/fuel-tracker-bd/`.

## Limitations

- Data lives in the browser's localStorage: clearing site data wipes it.
  Export JSON regularly as a backup.
- Mileage is computed from odometer deltas between consecutive refills; the
  first refill for a vehicle has no mileage value.
- Multi-vehicle support is one vehicle at a time on screen (use the "Use"
  button to switch the active vehicle).
- This is a personal tracker, not official data, built for educational/demo
  purposes. No fuel prices are shipped with the app; every price field is
  entered by you per refill.

## Status

Built and tested 2026-10-12. See `TEST_RESULTS.md` for the full run.
