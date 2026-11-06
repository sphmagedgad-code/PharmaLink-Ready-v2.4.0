# PharmaLink Ready 2 — v2.2.0 High-Volume TXT Import Engine

## Objective
Reduce the real daily import workload (6–10 WhatsApp TXT exports) without weakening medicine-name, Arabic/English alias, strength, or phone lookup.

## Changes
1. **Bounded duplicate detection**
   - Added compound IndexedDB index `by_groupId_receivedAt`.
   - Daily duplicate checks now use a range cursor over the latest 30 days for each source group instead of loading every historical message for that group into memory.
   - The database version advances from 2 to 3 solely to add this index. Existing stores and records are preserved; no reset or destructive migration is performed.
2. **30-day operational import window**
   - New operational ads older than 30 days in an imported export are skipped and counted in `outsideRetention`.
   - Existing medicines, contacts, deals, and already stored historical records are not deleted.
3. **Search-index write reduction**
   - Raw WhatsApp message token cap reduced from 40 to 12.
   - Parsed-entity token cap reduced from 20 to 12, prioritizing medicine text, canonical name, strength, phone, sender, and curated Arabic/English aliases.
   - Bulk entity chunk size raised to 1,000; search-index writes remain bounded at 4,000 rows per transaction for low-memory Android stability.
4. **Progress and reporting**
   - Existing real stage progress remains active through file read, classification, linking, and persistence.
   - The final report now includes the count of messages skipped because they are older than 30 days.

## Data safety
- No store is deleted or cleared.
- Medicines, contacts, deals, and old stored messages remain intact.
- Search-index rows are reduced only for new imports; existing search-index rows are not automatically rebuilt or deleted.
- Existing imported ads older than 30 days remain stored until the user archives them; the 30-day cutoff applies to newly imported content and duplicate scanning.
- Price estimates are not represented as confirmed offers.

## Version
- App: `2.2.0`
- IndexedDB: `3`
- Service worker cache: `v2.2.0-fast-index-import`
