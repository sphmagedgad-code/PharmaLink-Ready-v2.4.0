# QA Report — PharmaLink OS 2.3.0

All results below were produced by commands actually run in this build environment (Node 22, Playwright Chromium, headless). **Dataset: synthetic** (`tests/bench/generate-synthetic-export.mjs`, deterministic) unless stated. No real WhatsApp export was available, so **real-data accuracy (precision/recall) is NOT established.**

## Test results
| Test | Result |
|---|---|
| `tests/phone.test.mjs` (Egyptian/international parsing, wa.me/tel links, invalid numbers, search scoring) | PASS, 16 checks |
| `tests/price-extraction.test.mjs` | PASS, 3 checks |
| `tests/whatsapp-context-regression.mjs` (mixed contexts, extraction, strengths, prices, phones) | PASS |
| `tests/e2e-browser.cjs` (real Chromium: import, icons on every opportunity phone, ALL-filter phone search in 7 formats, Clexane 40 vs 20 must not match, 390 px/320 px layout, console errors) | PASS, no console/page errors |
| `tests/migration-v3-v4.cjs` (v3 DB created by the original app's own import, upgraded in place by v2.3.0) | PASS, run twice (before and after the final code) |
| `tests/opportunity-scan-e2e.cjs` (22,576 entities in DB) | PASS: original misses a 25-day-old matched buyer, 2.3.0 finds it; self-match absent |

## Migration v3 → v4
Populated v3 database (247 contacts, 174 medicines, 9,097 parsed entities, 52,211 search-index rows, 131 messages, 1 user-created deal). After upgrade every store has identical record count and identical SHA-256 over its id-sorted JSON. Only three obsolete indexes were removed. Search, by-medicine/contact/phone lookups and `getDistinctPhones` work on the migrated DB.

## Opportunity count investigation (9,097 entities from 131 messages)
- The count is not duplication: 4,766 distinct buyer/medicine/strength requests; 1,704 had a compatible seller within the old 5,000-entity window. The synthetic generator uses 70 senders posting random 30–90-item catalogues, so heavy overlap is expected.
- Real defects found and fixed: 251 self-matches; cross-strength saved-quote matches (33 opportunities carried a wrong price; 2 opportunities existed only because of it).
- Original vs 2.3.0 on the identical database: all 1,704 original opportunities retained (0 lost), +1,378 older buyers that the 5,000 cap had hidden (all outside the old window), 0 duplicate buyer keys, 0 self-matches.
- The earlier "1,704 vs 1,705" difference came from comparing runs on different generated files (the generator used `Date.now()`; now fixed) and a manually added deal, not from a matching change.

## Performance (synthetic, 131 messages / 9,097 entities, 3 alternating runs, machine otherwise idle)
| | Original 2.2.1 | 2.3.0 | Speed-up |
|---|---:|---:|---:|
| Import (runs) | 80.9 s median (78.2–85.6) | 62.6 s median (60.0–63.6) | 1.29x |
| Opportunities lookup | 2,396 ms | 791 ms | 3.03x (scans more: 3,082 vs 1,709 results) |
| Classifier only, 1,800 msgs | 3,278 ms | 2,475 ms | 1.32x (output identical except 108 price-only differences from the bare-ج rule) |
Large fixture (22,576 entities): import 291 s → 239 s (single run).

## Limitations / not verified
- No real-data precision/recall; duplicate and false-positive *rates* on real data unknown.
- WhatsApp/tel links verified as correct hrefs in Chromium; actually launching WhatsApp / the dialer on a physical Android device was not tested.
- Complete scan cost grows with database size (each medicine's entities are read once); not measured at 500k+ messages.
- Messages older than the existing 30-day import retention are still ignored (unchanged behaviour).
- Phone search covers phones known to the contact list / parsed entities; a phone only inside free message text is not indexed.
- Landline numbers: call icon only.
- Not pushed to GitHub (no network/repo access); the work was committed only locally in the build sandbox (the `.git` folder is not shipped; the original upload remains your baseline).
- Back up data before upgrading real installs (the migration was tested on pipeline-generated data, not a user's own database).
