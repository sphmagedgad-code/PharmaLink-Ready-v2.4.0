# PharmaLink Ready 2 — v2.2.0 QA Report

## Automated checks
- JavaScript syntax check across all `.js` and `.mjs` files: PASS.
- WhatsApp context/extraction regression suite: PASS.
- ZIP integrity: PASS.
- Schema migration is additive: adds only `by_groupId_receivedAt`; no object store deletion or data reset.

## Real TXT parser benchmark
Input: `دردشة في واتساب مع عالم الادويه .txt`
- Input size: 1,347,612 bytes as uploaded (normalized text bytes differ after bidi/digit normalization).
- Lines: 34,493.
- Reconstructed messages: 1,829.
- Actionable classified messages: 776.
- Message-level BUY: 404.
- Message-level SELL: 372.
- Urgent messages: 157.
- Medicine mentions: 12,868.
- Parser + reconstruction + classification on this test host: 9.38 seconds.
- Messages outside 30 days in this sample: 0.

These are engine output counts, not a human-labelled precision/recall score.

## Import-path optimizations
- Duplicate lookup now scans only the last 30 days by the compound source-group/date index.
- Raw-message indexing cap: 40 -> 12 tokens.
- Parsed-entity indexing cap: 20 -> 12 tokens, retaining curated aliases and prioritizing phone/strength/name.
- Bulk import batch: 500 -> 1,000 records; search-index write transaction remains capped at 4,000 rows.

## Production acceptance boundary
The full browser IndexedDB import (including the one-time v2-to-v3 index creation, dedupe, parsing, linking, saving, and index writes) has not been measured on the user's Realme C12 or Windows 7 laptop. Therefore no claim is made that the 45–50 minute real-world duration is already reduced below five minutes. The 9.38-second result is the parser-only benchmark, not the complete import. A first app open after upgrade may spend additional time creating the new compound index over existing messages; subsequent daily duplicate checks use that index.

## Release status
Performance-focused production candidate. Static and extraction regressions pass. Final acceptance requires timing a complete import on the target device and checking extracted medicine/phone/price records.
