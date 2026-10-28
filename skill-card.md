# 时光梳理·每日行程梳理 / Daily Location Timeline

Reconstruct a verified, day-by-day city/location timeline across years — from the messy traces a person actually leaves behind: payment records, travel tickets, navigation GPS, and photo EXIF GPS. Works for mainland-China users **and** international users.

## What it does

Given a multi-year archive of personal records, this skill produces a **daily "which city was I in" table** (date → primary city / all cities / evidence source / notes), plus city summary stats and stay-segment views, delivered as XLSX + CSV.

Core pipeline (8 stages): data inventory → source tiering → travel-leg extraction → **time-aware daily judgment** → photo EXIF GPS evidence → corrections & audits → delivery → post-mortem.

## Key methodology (why it's not just "group by date")

- **Time-aware judgment engine**: each day's primary city = the city with the longest *effective-time* stay within 07:00–24:00 local (sleep window excluded). Late-night departures and early-morning arrivals are attributed correctly — the #1 mistake of naive "arrival-first" engines.
- **Evidence hierarchy**: GPS/EXIF (L0) > travel legs (arrival/departure) > on-site spend (hotels/tickets) > chain-gap continuity > generic spend > inference. Conflicts are surfaced, never silently resolved.
- **Merchant-location trap**: online payment platforms bill under the *merchant's registered city*, not yours. Configurable keyword rules + "city only from product field, never from counterparty" guard rails are built in.
- **Photo EXIF GPS pipeline** (`photo_gps_judge.py`): batch-extract DateTimeOriginal + GPS, reverse-geocode via a pluggable city-coordinate library, then judge by time-weighted dwell with **distance-overrun, single-point enclave, and timezone-offset audits** (every one of these bit us in production).
- **Double safety gates** on chain-gap filling: max gap (default 14 days) AND chain continuity check — a 5-year data hole must never be "filled" with the last known city.
- **Cross-timezone aware**: all legs carry local times + offset; a Bangkok 23:40 departure lands you in Bangkok for that day, not your home city.

## Inputs

- Payment/bill exports with location columns (any format: WeChat/Alipay/UnionPay, bank CSV, card statements, or international equivalents)
- Travel legs JSON: flights / trains / buses / ferry / ride-hailing with from/to/dates/times (template included: `scripts/travel_legs.json`)
- Optional but powerful: photo/video EXIF GPS (extracted with exiftool, one JSON)
- Configuration block in `scripts/rebuild_final.py`: home cities, transit cities, corrections dict, merchant hint rules, coordinate library

## Outputs

- `每日所在城市_最终整合版.xlsx` / daily-cities XLSX: per-day primary city, all cities, evidence source, notes (multi-sheet with stay segments + city summary)
- Daily detail CSV for programmatic use
- `photo_daily_judgments.json`: per-day photo-GPS judgment with warnings for human review
- Audit report: corrections applied, gaps filled, anomalies flagged

## Typical use cases

- "Where was I on any given day of the last 10 years?" — tax, insurance, visa, immigration, or personal-history queries
- Rebuilding a life travel map from scattered multi-source records
- Auditing an existing hand-built timeline for merchant-location contamination

## Risks & limits (read before trusting output)

- All inference outputs are **estimates with evidence labels** — L0 GPS is fact; "inference" rows are hypotheses. Always spot-check before using for legal/official purposes.
- Reverse geocoding is only as good as the city-coordinate library shipped with it — add cities you visit or nearby-city misattribution will occur (documented real case: a famous lake 40 km from its nearest catalog city).
- Pure-audio ASR cannot detect video rewind/replays; timezones and DST shifts need explicit offsets in leg data.
- Non-destructive workflow recommended: never modify source records; keep raw evidence files.

## Requirements

- Python 3.8+ (openpyxl for XLSX), exiftool for photo GPS extraction
- Node.js + `clawhub` only if installing via ClawHub

## Links

- Source & docs: https://github.com/SunErdao/daily-location-timeline
- Method reference: `references/sop.md` · Travel-leg schema: `references/travel_legs_schema.md` · Photo-GPS method: `references/photo_gps_method.md`
