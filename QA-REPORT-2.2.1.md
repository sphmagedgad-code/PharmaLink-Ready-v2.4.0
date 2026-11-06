# PharmaLink Ready 2 — v2.2.1 QA Report

## Automated regression benchmark
Input: user-provided WhatsApp TXT export, 829,408 bytes, 34,493 text lines, 1,795 reconstructed WhatsApp messages.

| Metric | v2.2.0 | v2.2.1 |
|---|---:|---:|
| Classified messages | 776 | 776 |
| Extracted medicine mentions | 12,868 | 12,868 |
| Classifier elapsed time | 8,925 ms | 2,983 ms |
| Classification output equality | Baseline | Identical |

Observed classifier speedup: approximately 2.99x on this test run. Exact classified output comparison passed, including status, buy/sell type, medicine names, per-medicine side, phone numbers, price, quantity, and urgency.

## Checks
- `node --check src/orchestrators/classifyWhatsAppMessage.js` — passed.
- `node --check src/ui/whatsapp/whatsapp.js` — passed.
- v2.2.0 vs v2.2.1 classifier output comparison — passed (identical).

## Not yet verified
- Full import wall-clock time including IndexedDB writes, entity persistence, and search-index rows.
- 6–10 file batch completion within 10 minutes.
- Real-device app switching / background throttling on Realme C12 and Windows 7 laptop.
- Recovery after browser process termination or reload.

Do not treat the 2.99x classifier improvement as proof of the 10-minute end-to-end target. A full browser benchmark is required before production acceptance.
