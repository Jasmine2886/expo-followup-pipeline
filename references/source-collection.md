# Source collection — scraping detail for the five sources

## Source 1: Calendar covering the trip (highest priority)

Why it ranks first: meetings marked "confirmed" in the pre-trip plan often never happen, and unplanned ones do. Short events the user typed into their phone on site ("Acme demo", "dinner w/ distributor") *are* the real meeting list.

How:
```
<CAL_CLI> agenda --start <trip start date> --end <return date + 1>
```
- Pull it, sort by start time, print `time | title | event_id`
- Mark **manually created short-title events** as "actually happened"; mark **pre-planned ones** as "planned"
- Where the two diverge — planned but no actual (no-show), or actual but no plan (walk-up meeting) — that divergence is exactly what belongs in the follow-up table

## Source 2: AI voice transcripts

- Lives at `VOICE_APP_URL`, target session `VOICE_SESSION_TITLE`
- Typical shape: one session titled something like "notes to organize", containing multiple transcript segments, possibly out of order, with transcription errors
- Handling: don't ask the user to clean it up — ingest the raw transcript. Slice it by company/person name into the Step 2 cards
- Tell the user explicitly: knowing "which days/occasions this covers", "whether a next step was agreed", and "what was left unclear at the time" makes the output dramatically more accurate
- Login mechanics: SKILL.md §2. **More than 3 failed login rounds → fall back to having the user paste the raw text**

## Source 3: Event app messages (Swapcard-style)

- Entry point `EVENT_APP_URL`; login via one-time email code
- Grab all conversation permalinks in one pass (JS snippet in SKILL.md §3)
- Body parsing: content begins at the first `Month D, YYYY` line and ends at `Scroll to bottom`
- Timestamps render in the browser's local timezone → convert to `TZ_BASE`
- Cross-check trick: a message's time plus its content (e.g. "I can't find your booth, 18:49 home ≈ 14:49 local, right after our scheduled 14:00") can confirm whether your timezone base is correct
- Output:
  - Has a matching calendar event → append raw chat + summary into that event's notes
  - No matching event → it's a new lead, list it separately (these are frequently the high-value ones everyone forgets)

## Source 4: Email

- Search the event name, the organizer name, and key contact surnames
- Focus on three types: confirmations / cancellations / inbound follow-ups from the other side
- Calendar events corresponding to a cancellation should be deleted (mind the confirm flag) so the calendar isn't left holding phantom meetings

## Source 5: Ledger / CRM (`LEDGER_DOC`)

- Check whether the *on-site notes* and *post-meeting action points* fields are empty → empty means on-site never filled them in, which is precisely this run's target
- Reuse the existing P1/P2/P3 tiers and contact tags (prospect / in negotiation / channel partner / existing customer) — **don't invent a parallel taxonomy**
- When writing back, use a targeted string replace on the cell text, then read back to verify

## Merge rules

Multiple people at one company (e.g. a regional director and a global partnerships lead at the same firm) → **merge into one card**, noting each thread's progress separately, so nobody double-contacts them.
Two companies claiming the same booth number (e.g. both listing B50) → put it in "needs verification" and ask the user. Don't guess.
