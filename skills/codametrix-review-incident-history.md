---
name: Review CodaMetrix incident and maintenance history
description: Pull CodaMetrix's published incident and scheduled-maintenance history to assess reliability posture, maintenance cadence, and how the vendor communicates during an event — for vendor diligence, QBR prep, or a security/reliability questionnaire.
api: openapi/codametrix-status-openapi.yml
operations:
  - getIncidents
  - getScheduledMaintenances
  - getUpcomingScheduledMaintenances
  - getActiveScheduledMaintenances
generated: '2026-08-04'
method: generated
source: openapi/codametrix-status-openapi.yml + lifecycle/codametrix-lifecycle.yml
---

# Review CodaMetrix incident and maintenance history

Use this when evaluating CodaMetrix as a vendor, preparing a health-system QBR, or answering
"how often does CMX CARE go down and how do they tell us?"

## Before you start

- **No credentials.** `https://status.codametrix.com/api/v2` is public.
- **The window is fixed at 50 records per collection.** There are no pagination parameters. Anything
  older than the 50 most recent records is only reachable through
  `https://status.codametrix.com/history.atom` / `history.rss` or the HTML history pages.

## Steps

1. **Call `getIncidents`** — `GET /incidents.json`. Returns the 50 most recent incidents including
   `resolved` and `postmortem` ones.

2. **Bucket by `impact`** (`none`, `minor`, `major`, `critical`) and by year using `created_at`. Report
   counts per bucket, not a raw list. Note the range you actually have — if fewer than 50 records come
   back, the history is complete for the page's lifetime.

3. **Measure time-to-resolve** per incident as `resolved_at − started_at`. `monitoring_at` is when it
   moved to the monitoring state; a null `resolved_at` means still open.

4. **Read the communication quality, not just the numbers.** Walk `incident_updates` oldest-to-newest
   (the array arrives newest-first). Judge: did each state transition carry a substantive `body`? Was a
   named support contact given? Was an expected duration stated?

5. **Call `getScheduledMaintenances`** — `GET /scheduled-maintenances.json`. Bucket by `status`; compute
   the cadence from `scheduled_for` intervals and the typical window length from
   `scheduled_until − scheduled_for`. A page where all 50 records are `completed` is an actively used
   maintenance process, not a dormant page.

6. **Check what is pending** with `getUpcomingScheduledMaintenances` and `getActiveScheduledMaintenances`
   before making any claim about current state.

7. **Cross-check which surfaces were hit.** Roll up `incident_updates[].affected_components[]` by `code`
   to see whether events concentrate on the coding engine (**CMX Automate**), **Analytics Dashboards**,
   or **CMX-Amplify**.

## Rules

- **Do not compute an uptime percentage and present it as CodaMetrix's.** No SLA or uptime target is
  published; a derived figure is your estimate, and must be labeled as such.
- **Do not treat maintenance as downtime.** Maintenance impact is recorded separately and CodaMetrix's
  notices describe delayed case processing, not unavailability.
- **Quote, don't paraphrase, incident text** when it will be repeated to a customer — `body` is markdown
  written by CodaMetrix and is the vendor's own statement.
- **State the window.** Any reliability claim must say "over the 50 most recent published records,
  <date> to <date>" — the API cannot give you more than that.
