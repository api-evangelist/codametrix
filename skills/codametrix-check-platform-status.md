---
name: Check CodaMetrix platform status
description: Determine whether the CodaMetrix CMX CARE platform is healthy right now, which components are affected, and whether an incident or maintenance window is in progress — in one unauthenticated call.
api: openapi/codametrix-status-openapi.yml
operations:
  - getSummary
  - getStatus
  - getComponents
generated: '2026-08-04'
method: generated
source: openapi/codametrix-status-openapi.yml + conventions/codametrix-conventions.yml
---

# Check CodaMetrix platform status

Use this when you need the current operational state of CodaMetrix's autonomous coding platform —
before escalating a coding-throughput complaint, when a health system reports cases not processing,
or on a monitoring loop.

## Before you start

- **No credentials.** Every endpoint is public. Do not send an `Authorization` header or API key.
- **Base URL:** `https://status.codametrix.com/api/v2`
- **Respect the cache.** Responses carry `cache-control: max-age=10` and a weak `ETag`. Poll no faster
  than every 10 seconds; send `If-None-Match` with the previous ETag and treat `304` as "unchanged".

## Steps

1. **Call `getSummary`** — `GET /summary.json`. One call returns everything: page identity, the blended
   `status` rollup, the full `components` array, unresolved `incidents`, and active/upcoming
   `scheduled_maintenances`. Do not make four separate calls.

2. **Read `status.indicator` first.** It is one of `none`, `minor`, `major`, `critical`. `none` with
   description `All Systems Operational` means healthy — stop here unless the caller asked for detail.

3. **If the indicator is not `none`, walk `components`.** Skip any record where `group` is `true` — those
   are group headers (`CMX-Automate`, `CMX-Amplify`), not monitored surfaces. Report the leaf components
   whose `status` is not `operational`. The three real surfaces are **CMX Automate** (the coding engine),
   **Analytics Dashboards**, and **CMX-Amplify**.

4. **Check `incidents`.** In `summary.json` this array holds only *unresolved* incidents. Each carries
   `name`, `impact` (`none`/`minor`/`major`/`critical`), `status`
   (`investigating`/`identified`/`monitoring`), a `shortlink` to the public incident page, and an
   `incident_updates` timeline. Report the **most recent** update — `incident_updates[0].body` — as the
   current statement, not the oldest.

5. **Check `scheduled_maintenances`.** In `summary.json` this holds active and upcoming windows only.
   `scheduled_for` and `scheduled_until` bound the window. Maintenance is not an outage: CodaMetrix's own
   notices describe minor case-processing delays, not unavailability.

6. **If you only need the headline**, call `getStatus` (`GET /status.json`) instead — it returns just
   `page` and `status` and is the cheapest poll.

## Rules

- **An empty array is the healthy state, not a failure.** `incidents: []` means nothing is wrong.
- **Never infer an outage from a 404.** A `404` on this API means you called a path that does not exist —
  the body is empty and there is no error envelope. Re-check the path against the eight valid endpoints.
- **Do not join components by name.** `affected_components[].name` is group-qualified
  (`"CMX-Automate - CMX Automate"`) while `components[].name` is not. Join on the component id
  (`affected_components[].code` → `components[].id`).
- **Do not claim an SLA.** CodaMetrix publishes no uptime target or public SLA; report observed status only.
