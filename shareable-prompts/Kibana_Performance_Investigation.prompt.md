---
name: "Kibana Performance Investigation"
description: "Investigate a WiseCloud client performance incident using Kibana GlobalSearch. Use for SQL lock timeout errors, force-close events, slow save operations, and system-wide blocking events."
argument-hint: "Provide the CW license code (e.g. MO4SYR), the physical SQL server hostname if known (e.g. us2wp-ssql-438b), and the date/time window to investigate (UTC or ET)"
agent: "agent"
---

Conduct a Kibana GlobalSearch server-side investigation for a WiseCloud performance incident.

## Requirements

- Open the GlobalSearch entry page at https://wisetechglobal.sharepoint.com/sites/ServerManagement/SitePages/Elastic-GlobalSearch.aspx to confirm the correct Kibana instance for the client's datacenter before proceeding.
- Read the full technical reference at `.github/guides/kibana-performance-investigation.md` before running any queries. All field names, data view names, and query templates are there.
- Determine the correct Kibana instance from the client's datacenter: AMER (us2wp-*) → kibana.amer-prod-1.wtg.zone; APAC (au2wp-*) → kibana.apac-prod-1.wtg.zone; EMEA (de1wp-*) → kibana.emea-prod-1.wtg.zone.
- Navigate to the `wisecloudsupport` space.

## Investigation Steps

Run all ES|QL queries via the Kibana REST API (`page.evaluate` + `fetch`), not the browser editor. See the repo memory file for the exact fetch pattern and header requirements.

Step 1 — Confirm the physical SQL server FQDN for the client's license code by checking the `srdh.server` field in the performance log. The CW DNS alias (`CWCODE.db.wisegrid.net`) does not match the indexed hostname.

Step 2 — Count lock timeout errors by hour across the incident date window:
  - Data view: `logs-cargowise.sessionhost.performance-wtg`
  - Filter: `srdh.code == "CWCODE"` and `srdh.message LIKE "*Lock request time out*"`
  - Group by `DATE_TRUNC(1 hour, @timestamp)`

Step 3 — Retrieve all lock timeout errors in chronological order to identify affected table, PK, session host, and operation type. Look for: same PK repeated (same user retrying), multiple PKs on multiple hosts (systemic block), recurring bursts (scheduled process).

Step 4 — Identify which form(s) produced the errors (group by `srdh.object`).

Step 5 — Find very slow completed operations (`srdh.duration >= 30000 AND srdh.message == "completed"`). These are candidates for the lead blocker process — a long-running Load or Save on an overlapping time window.

Step 6 — Narrow to 15-minute buckets to pinpoint the exact start and end time of the blocking window.

Step 7 — Check the SQL Server error log (`logs-microsoft_sqlserver.log-wtg`) for the physical server during the same window. Look for Error 17830 (connection resets), deadlock events, or maintenance operations.

## Evidence to Surface

At minimum, confirm and report:
- Total count of lock timeout errors
- Blocked table name (from `srdh.message` text — look for `Tablename:`)
- Time range (first and last error timestamp, ET)
- Number of distinct session hosts affected
- Escalating or burst pattern (by 15-minute bucket)
- Build version (from `CWProductVersion` field in performance log if available, or from the incident record)

## Output

- Summarise the confirmed server-side findings in chat with specific numbers and timestamps.
- These findings are used to update the client-facing response file for the incident with confirmed evidence rather than behavioral inference.
- Do not draft a new client-facing response file in this prompt — update the existing one if instructed.
- If the lead blocker cannot be identified from available Kibana data (requires Extended Events traces), state that explicitly and confirm the escalation path to WiseCloud infrastructure.

Do not poll, retry loops, or sleep in any terminal commands. Return Kibana query results via `run_playwright_code` only.
