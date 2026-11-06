# PharmaLink OS 2.3.0 — Release Notes

## New
- **Call + WhatsApp icons in Opportunities** for the buyer and every seller. WhatsApp uses `https://wa.me/<international digits>`, calling uses `tel:+<international>`. Egyptian numbers are normalised (`010…`, `+20 10…`, `0020…`, `20…`, `10…`, Arabic digits). A number that cannot be validated shows **no** icon (never a dead link). Landlines get call only.
- **Phone search under the ALL filter**, in the existing main search field: full or partial numbers (≥ 4 digits) in any Egyptian format, alongside the unchanged name/product search. Other filters are unchanged.
- Phone icons also appear on search-result cards (contact, parsed entity, message).

## Fixed
- **Complete opportunity scan.** v2.2.1 only looked at the newest 5,000 parsed entities, so older buy requests silently vanished once the database grew. Every medicine's entities are now read once through the existing index, so no buy request or offer is excluded because of volume.
- **Self-matches.** A contact could be shown as a seller for his own request (251 of 1,704 opportunities in the synthetic test set). Buyers are never matched to themselves (same contact or same phone).
- **Cross-strength false match.** A seller who offered a medicine in another strength (e.g. Clexane 20 mg) could re-enter a 40 mg request through his saved price, and that price was shown as a real quote. Fixed in `matchSupplier.js`.
- **Prices written as "3200 ج"** are now recognised; words such as "جرام" / "جديد" are not read as prices.

## Performance (see QA-REPORT-2.3.0.md; synthetic data)
- Import: 80.9 s → 62.6 s (1.29x, 3 alternating runs, 131 messages / 9,097 entities).
- Opportunities lookup: 2,396 ms → 791 ms (3.03x) while scanning *more* data (complete scan).
- Classifier alone: 1.32x (1,800 messages).

## Data / upgrade
- IndexedDB schema **v3 → v4**, automatic and non-destructive. Three unused indexes are dropped (`parsedEntities.by_groupId`, `parsedEntities.by_type`, `searchIndex.by_entityType`). No record is modified. Verified by SHA-256 over every store (see QA report).
- Service-worker cache bumped to `v2.3.0-phone-actions-import-speed`; `phone.js` and `phoneSearch.js` are pre-cached.
- No new dependencies, no paid services.

## Known limitations
See QA-REPORT-2.3.0.md §Limitations. Key points: all accuracy/performance numbers are synthetic (no real export was supplied); touch/Android behaviour was tested in desktop Chromium emulation only; the complete scan scales with database size and was not measured at 500k+ messages.
