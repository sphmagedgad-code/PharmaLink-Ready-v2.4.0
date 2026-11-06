# QA Report — PharmaLink OS 2.4.0

Everything below was executed in this build environment (Node 22, Playwright Chromium headless) **from a clean extraction of the final ZIP** (`unzip` into an empty directory, served over HTTP). **All datasets are synthetic / developer-written. No real WhatsApp data was available, so real-world accuracy is NOT established.**

## 1. Test results (all PASS)
| Test | Scope | Result |
|---|---|---|
| `phone.test.mjs`, `price-extraction.test.mjs`, `whatsapp-context-regression.mjs` | 2.3.0 regression (phones, prices, mixed buy/sell contexts) | 16 + 3 checks + context test PASS |
| `engine.test.mjs` | specs, intent, enrichment, price stats, forecast, ranking, qualification A/B/C, market assembly, commercial rules | 63 checks PASS |
| `e2e-browser.cjs` | 2.3.0 features on the new UI: WhatsApp/tel links, ALL-filter phone search (7 formats), 390/320 px layout, console errors | PASS |
| `intelligence-e2e.cjs` | full import → IndexedDB → engine → UI: 8 business scenarios, price history, prices screen, 390 px overflow, archive | PASS |
| `price-history-integrity.cjs` | append-only, completeness, executed-only-via-deal, atomic abort | PASS |
| `migration-safety.cjs` | copy of real v4 DB → v5, interrupted upgrade, backup/restore, original untouched | PASS |
| `rollback-e2e.cjs` | 2.4.0 full backup → 2.3.0 restore | PASS |
| `opportunity-scan-e2e.cjs` | 22,576 entities (finished) | PASS |

## 2. Price-history integrity (real IndexedDB)
* Overwrite attempt through the only write API (same ids, price 1, "tampered" evidence, year 2000): 0 rows added, every historical row **byte-identical** afterwards.
* Repository exports only `addObservations / countObservations / getAllObservations / getObservationsByMedicine` (no update/delete/put).
* A later price from the same seller is appended; old (3,200) and new (3,500) both kept in time order. Re-importing the same chat adds 0 rows.
* Every row (5 checked in this test; 60 in the migration data) carries id, product, spec key + spec object, side, price, currency, quantity, timestamp, source message text/id, party (phone/contact), status; message-derived rows are never `transaction`; a pending deal yields no executed price, a completed deal yields exactly one (`priceSource: deal`).
* Approximate/"around" prices are stored as `hypothetical` (evidence kept, excluded from statistics).
* An aborted write transaction leaves no partial row. Archiving deleted 19 entities and kept all 16 price rows.
* Limit: append-only is enforced by the repository API, not by the browser — someone writing raw IndexedDB in devtools could still alter rows.

## 3. Database migration safety
Previous-release database (2.3.0, DB v4) created by its own import pipeline: contacts 247, deals 1 (completed), medicines 174, parsedEntities 9,097, searchIndex 52,179, whatsappMessages 131.
* Upgrade run on a **copy** of the browser profile: after upgrade every old store has the same count and SHA-256 over its id-sorted JSON (247/1/174/9,097/52,179/131 → identical); `priceHistory` is a new empty store; **0 differences**.
* **Interrupted upgrade** (version-change transaction aborted half-way): database remains v4, all 6 stores identical, no half-created store; reloading completes the upgrade. (Browser-level atomicity; power-loss not simulated.)
* The **original** profile re-opened with 2.3.0 afterwards: still v4 and identical.
* Back-fill then created 60 price rows (59 from messages with evidence + 1 transaction from the completed deal); a second back-fill added 0.
* **Backup → restore** into an empty 2.4.0 DB: medicines 174, contacts 247, deals 1, messages 131, entities 9,097, price rows 60 — messages, entities and price history byte-identical; medicines/contacts/deals identical; restoring twice added nothing.
* **Rollback** (2.4.0 full backup → 2.3.0): all entities restored, same opportunity found. v3 → v5 direct upgrade was also tested earlier in this build (records identical, three obsolete indexes dropped).

## 4. Opportunity classification (developer-labeled scenarios, real import pipeline)
| Scenario | Expected | Got |
|---|---|---|
| Nexavar 200 mg, buyer 3,400 vs sellers 3,100/3,200, all confirmed, fresh | A; sellers ranked 3,100 then 3,200 | A, correct order |
| Clexane 40 mg vs 20 mg | no opportunity | none |
| Lynparza 56 vs 28 tablets | no opportunity | none |
| Offer 10 days old | C (expired) | C |
| Buyer wants 5 boxes, seller has 2 | B (quantity shortfall) | B |
| Same seller 700 then 800 within 24 h | B, conflict flagged, both prices kept | B |
| Buyer gives no strength | B (strength missing) | B |
| Seller price "approx" | B (price not confirmed) | B |
Opportunity-level precision 100 % / recall 100 % / category-correct 6/6 **on these 8 hand-written scenarios only**. Engine unit tests add: price equal on both sides without stated availability → B (never A); only completed deals give executed prices; two brands of the same ingredient never match; anonymous senders never matched; different pack size = conflict.
Specification extraction on 20 hand-labeled lines: strength 11/11, concentration 7/7, form 16/16, pack 8/8, quantity 3/3, 0 false positives; intent flags 13/13. Same caveat: author-written examples, not real chats.

## 5. Large-scale scan (22,576 parsed entities in the database)
Finished. Import 137 s (single run). Market view 1.5 s. Valid opportunities: **1** (the only eligible buyer/seller pair — 25 days old, so correctly Unqualified/C "expired"); **0 duplicates; 0 self-matches in results (1 self-pair correctly skipped); 0 false matches** (no unrelated pair listed); eligible pairs found 1 of 1. 22,484 active offers had no buyer, so no further opportunities exist in this fixture. Missed matches are measurable only for this constructed fixture. The old 2.3.0-style limit (5,000 newest entities) is gone.
Earlier 9,097-entity synthetic dataset: 2.4.0 returns 4,498 opportunities (A 0, B 1,863, C 2,635 — all synthetic messages are 20 days old, hence expired/unqualified); not a market figure.

## 6. Performance (synthetic, deterministic export, 131 messages / 9,097 entities, 3 alternating runs)
| | 2.3.0 | 2.4.0 |
|---|---:|---:|
| Import (median) | 45.2 s | 45.9 s (1.01x; includes price-history sync) |
| Screen data load (median) | 483 ms | 1,119 ms (2.3x **slower**, but now parses specs/intent, qualifies A/B/C and ranks) |

## 7. Remaining limitations (honest list)
1. **No real-data validation.** Precision/recall on real WhatsApp text is unknown; a labeled export (anonymized) is required to measure it and to tune thresholds in `src/engine/config.js`.
2. "Confirmed" price is a wording-based rule, not confirmation by the counterparty; "Verified" therefore means *both messages explicitly state intent, spec, price and stock and are fresh* — it is not a guarantee. Nothing is presented as a completed transaction.
3. Catalog posts: a price is taken only when its line has exactly one product and one price; otherwise left empty (precision over recall).
4. Credibility exists only for contacts with deals recorded in the app; with no deals every party shows "no evidence". Historical response speed and urgent-fulfilment ability are not measurable and are not scored.
5. Active ingredient: partial informational map (~27 products), not used for matching; brand/generic interchangeability is deliberately not assumed.
6. Quantity: only box units (علبة/كرتونة…) are read; counts of ampoules/vials/pens/syringes are flagged ambiguous. Unknown quantity is listed for confirmation but does not by itself block "Verified". Location uses governorate keywords only.
7. Forecast: simple linear regression, no seasonality/market drivers, never validated against real prices; shown only with enough data and always as an estimate.
8. Performance at 500k+ messages was not measured; the screen load grows with the number of buyer/seller pairs per product.
9. Messages older than the existing 30-day import retention are ignored at import (price history keeps what was already recorded).
10. WhatsApp/tel links are verified as correct URLs in Chromium; launching WhatsApp or the dialer on a physical phone was not tested. Firefox/Safari not tested.
11. Append-only is API-level (see §2). A power loss during upgrade was not simulated.

## 8. Production-readiness assessment
**Suitable for a supervised pilot with real data and a backup** — all verified functionality passes and no known critical defect is open. **Not yet proven for unsupervised commercial use**: accuracy on real pharmaceutical chats, real-device behaviour and large-database performance are unverified. Recommended next step: run a pilot on an anonymized real export, label ~200 messages, measure precision/recall, then tune.
