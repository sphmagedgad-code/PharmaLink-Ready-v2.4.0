# PharmaLink Ready 2 — v2.2.1 Import Engine Optimization

## Changes
- Optimized medicine-side context classification: BUY/SELL keyword offsets are collected once per message and resolved by binary search for each medicine occurrence. This removes repeated full keyword rescans for every extracted medicine occurrence.
- Precomputed normalized known-medicine identities once at module load rather than normalizing the full dictionary for each extracted entity.
- Kept extraction decisions and output fields unchanged; the regression comparison on the supplied WhatsApp TXT returned byte-identical classified output.
- WhatsApp screen now keeps the active analysis job/progress in module state and reconnects the newly rendered screen after internal navigation, instead of leaving the progress UI bound to a detached screen.
- Bumped app and service-worker cache versions to force updated JS delivery.

## Important limits
- The 10-minute target for 6–10 files is not claimed as verified. The benchmark below measures parsing/classification only; IndexedDB writes and search-index construction still require a full browser run on the target laptop/phone with the complete batch.
- Returning to the WhatsApp screen within the same live app session reconnects to the active job. A browser/app process kill, tab discard, device restart, or page reload is not yet a durable checkpoint/resume guarantee.
