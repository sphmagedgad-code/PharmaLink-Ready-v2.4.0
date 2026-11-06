# PharmaLink OS 2.4.0 — Commercial Intelligence Engine

## What is new
**Product intelligence** (`src/engine/specs.js`, `intent.js`, `enrich.js`): exact strength, concentration (% and mg/ml), dosage form, pack size, box quantity, location, buying vs selling intent, question / hypothetical-price / out-of-stock detection. Only validated conversions are applied (mcg/mg/g). Ambiguous text (several strengths, "10 امبول" pack-or-quantity) is **flagged, never guessed**. The original message is kept as evidence on every record. Active ingredient is shown for ~27 known products (informational; different brands of one ingredient are never matched).

**Commercial rules built into data and logic** — a published price ≠ a confirmed price ≠ an executed transaction; advertised stock ≠ current availability; purchase interest ≠ a purchase. Price statuses: `announced`, `confirmed` (offer with explicit availability / request with explicit purchase wording, no question or approximation), `hypothetical` (kept but excluded from statistics), `transaction` (only from a completed deal).

**Permanent price history** (`priceHistory`, DB v5, append-only): every priced message and every completed deal is recorded with product spec, side (buy/sell), price, quantity, date, source message text, party and status. No update/delete API exists; ids are deterministic so syncs are idempotent; archiving old messages first syncs the history and aborts the delete if the sync fails.

**Prices screen (الأسعار)**: per product and per exact specification — latest confirmed market price, lowest current confirmed sell, highest current confirmed buy, latest recorded price per side with its status and the previous price, averages/median, % change, historical trend, unusual-move flags, ranked sellers and buyers with evidence, the last 15 records with status and original message. The **forecast** appears only with ≥ 6 observations on ≥ 3 days spanning ≥ 5 days and is labelled "تقدير إحصائي (ليس سعر سوق)" with direction, expected 7-day change, confidence and basis; otherwise it states why it is unavailable.

**Rankings** (every row shows its evidence): sellers = lowest *confirmed* price → freshest confirmation → documented deals; buyers = highest confirmed price → freshest → documented deals. Credibility comes only from deals the user recorded; message volume is displayed as "not used" and never ranked. Historical response speed and urgent-fulfilment ability are not measurable from the data and are stated as such.

**Opportunities screen**: A = Verified (exact spec, both prices confirmed, stock stated, fresh ≤ 72 h / 7 d, buyer price ≥ seller price, no conflicts, product identity verified, no poor credibility), B = Potential (what still needs confirmation is listed), C = Unqualified (expired offers > 7 d / requests > 14 d, withdrawn stock, not an offer). Incompatible products/specs and self-matches are never listed. Every card shows buyer & seller name/phone (WhatsApp/call), quantities, prices with status, price difference, confirmation time, credibility facts, missing information, the original messages, and the notice that it is **not a completed or guaranteed transaction**.

## Fixes inherited from 2.3.0 work kept
Complete scan (no 5,000-entity cap), no self-matches, no cross-strength matches.

## Database
Schema v3/v4 → **v5** adds only the empty `priceHistory` store (indexes by medicine, contact, time). Upgrade, interrupted-upgrade recovery, backup/restore and rollback are tested — see `QA-REPORT-2.4.0.md`, `docs/ROLLBACK-v5.md`, `docs/INTELLIGENCE-ENGINE.md` (data model).
Full backups now include `priceHistory`.

## Files
New: `src/engine/*` (config, specs, intent, ingredients, enrich, priceStats, ranking, qualify, market), `src/db/repositories/priceHistoryRepo.js`, `src/orchestrators/marketIntelligence.js`, `src/ui/prices/prices.js`, `src/ui/shared/format.js`, tests, docs.
Modified: `schema.js`, `constants.js` (v5, version 2.4.0, route), `nav.js`, `service-worker.js` (cache v2.4.0, precache), `opportunities.js` (rewritten), `importWhatsAppFile.js` (+price sync), `archiveWhatsApp.js`, `backupRestore.js`, `classifyWhatsAppMessage.js` (export only), `README.md`, `VERSION`.

## Known limitations
See QA report. Most important: **no real WhatsApp export was available — every accuracy figure is from developer-written test sets; real-market accuracy is unverified.**
