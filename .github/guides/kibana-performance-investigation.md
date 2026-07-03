# Kibana GlobalSearch — Performance / SQL Lock Investigation Workflow

## Entry Point

Start here: https://wisetechglobal.sharepoint.com/sites/ServerManagement/SitePages/Elastic-GlobalSearch.aspx

This SharePoint page lists all available GlobalSearch instances. eye.wtg.ws was decommissioned February 28, 2026 — do not use it.

## PROD Kibana Instances by Region

| Region / Datacenter | URL | Use for |
|---|---|---|
| AMER / US2 | kibana.amer-prod-1.wtg.zone | WiseCloud Americas (us2wp-* hosts) |
| APAC / AU2 | kibana.apac-prod-1.wtg.zone | WiseCloud AU / APAC |
| EMEA / DE1 | kibana.emea-prod-1.wtg.zone | WiseCloud Europe |

- Navigate to the correct regional instance for the client's datacenter.
- Space for WiseCloud support investigations: `wisecloudsupport`

## ES|QL: Use REST API, NOT the Browser Editor

The Kibana Discover ES|QL browser editor corrupts long queries (autocomplete inserts completions mid-type; URL-based navigation double-encodes hostnames). Always use the REST API via `page.evaluate` + `fetch`:

```javascript
const result = await fetch('/s/wisecloudsupport/api/console/proxy?path=%2F_query&method=POST', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json', 'kbn-xsrf': 'true' },
  body: JSON.stringify({ query: 'FROM logs-cargowise.sessionhost.performance-wtg | WHERE ...' })
});
const data = await result.json();
// Results are in data.values (column-oriented array)
```

The `kbn-xsrf: true` header is required. Call this inside `run_playwright_code` with `page.evaluate(async () => { ... })`.

## Key Data Views for SQL Lock / Performance Incidents

| Data View | Best For |
|---|---|
| `logs-cargowise.sessionhost.performance-wtg` | **PRIMARY** — SQL Error 1222 (lock request timeout) ONLY appears here; also slow operations and error patterns |
| `logs-microsoft_sqlserver.log-wtg` | SQL Server error log — deadlocks, Error 17830, hardware errors. Does NOT contain Error 1222. |
| `logs-cargowise.sessionhost.userstats-wtg` | Session counts per customer — verify whether sessions dropped during the incident |
| `logs-cargowise.servicetask-wtg` | Background service task activity |

Not available in the wisecloudsupport space (return Unknown index):
- `logs-microsoft_sqlserver.audit-wtg`
- `metrics-microsoft_sqlserver.transaction_log-wtg`
- `metrics-sqlcpumonitoring-wtg` (Architecture space only, APAC — no support access)

## Performance Log Fields (`logs-cargowise.sessionhost.performance-wtg`)

| Field | Description | Example |
|---|---|---|
| `srdh.code` | CW license code (uppercase) | `"MO4SYR"` |
| `srdh.server` | Database host FQDN | `"us2wp-ssql-438b.wisecloud.zone"` |
| `srdh.duration` | Operation duration in ms | `19186` |
| `srdh.object` | Form / module name | `"ForwardingShipment"` |
| `srdh.op` | Operation type | `"Load"`, `"Save"` |
| `srdh.message` | `"completed"` for success; full error text for failures | `"** Error Saving Record **..."` |
| `srdh.host` | Session host FQDN | `"US2WP-SRDH-956"` |
| `srdh.Customer` | Customer code (lowercase) | `"mo4"` for MO4SYR |

## SQL Error 1222 — Lock Request Timeout

- This is an APPLICATION-LAYER error. It does NOT appear in the SQL Server error log (`logs-microsoft_sqlserver.log-wtg`).
- It ONLY appears in `logs-cargowise.sessionhost.performance-wtg`, inside `srdh.message`.
- ES|QL filter: `srdh.message LIKE "*Lock request time out*"`
- Typical `srdh.duration`: 15,000–19,000 ms (SQL waits the full lock timeout period, then returns the error).
- The `srdh.message` text includes: ServerName, DatabaseName, Tablename, PK, RowState, Factory name, Business object class, and Inner Message.

## Key ES|QL Queries

### Count lock timeouts by hour
```esql
FROM logs-cargowise.sessionhost.performance-wtg
| WHERE srdh.code == "CWCODE" AND @timestamp >= "2026-07-02T00:00:00.000Z" AND @timestamp <= "2026-07-03T00:00:00.000Z"
| WHERE srdh.message LIKE "*Lock request time out*"
| EVAL hour = DATE_TRUNC(1 hour, @timestamp)
| STATS count = COUNT(*) BY hour | SORT hour ASC
```

### All lock timeouts (chronological) — identifies affected table, PK, session host
```esql
FROM logs-cargowise.sessionhost.performance-wtg
| WHERE srdh.code == "CWCODE" AND @timestamp >= "..." AND @timestamp <= "..."
| WHERE srdh.message LIKE "*Lock request time out*"
| KEEP @timestamp, srdh.duration, srdh.object, srdh.op, srdh.host, srdh.message
| SORT @timestamp ASC | LIMIT 50
```

### Which form(s) are producing errors?
```esql
FROM logs-cargowise.sessionhost.performance-wtg
| WHERE srdh.code == "CWCODE" AND @timestamp >= "..." AND @timestamp <= "..."
| WHERE srdh.message LIKE "*Lock request time out*"
| STATS count = COUNT(*) BY srdh.object | SORT count DESC | LIMIT 10
```

### Very slow completed operations (candidates for lead blocker)
```esql
FROM logs-cargowise.sessionhost.performance-wtg
| WHERE srdh.code == "CWCODE" AND @timestamp >= "..." AND @timestamp <= "..."
| WHERE srdh.duration >= 30000 AND srdh.message == "completed"
| KEEP @timestamp, srdh.duration, srdh.object, srdh.op, srdh.host
| SORT srdh.duration DESC | LIMIT 15
```

### 15-minute bucket distribution (pinpoint start / end of blocking window)
```esql
FROM logs-cargowise.sessionhost.performance-wtg
| WHERE srdh.code == "CWCODE" AND @timestamp >= "..." AND @timestamp <= "..."
| WHERE srdh.message LIKE "*Lock request time out*"
| EVAL bucket = DATE_TRUNC(15 minutes, @timestamp)
| STATS count = COUNT(*) BY bucket | SORT bucket ASC
```

### SQL Server error log — by event code
```esql
FROM logs-microsoft_sqlserver.log-wtg
| WHERE host.name == "us2wp-ssql-NNN.wisecloud.zone"
| WHERE @timestamp >= "..." AND @timestamp <= "..."
| STATS count = COUNT(*) BY event.code | SORT count DESC | LIMIT 30
```

## Hostname Gotchas

- CW DNS alias format: `{CWCODE}.db.wisegrid.net` — appears in error messages but is NOT the server name Kibana indexes. Using it in `host.name` filters returns no results.
- Physical SQL server format: `us2wp-ssql-NNN.wisecloud.zone` — get from the incident, Windows service details, or the `srdh.server` field in the performance log.
- Session hosts: `us2wp-srdh-NNN.wisecloud.zone`
- Customer code in userstats: lowercase, typically first 2-3 chars of license code (e.g., `"mo4"` for MO4SYR)

## Interpreting Lock Timeout Patterns

- **Same PK 3+ times within 90 seconds**: one user repeatedly retrying a blocked save. Confirms blocking was sustained, not momentary.
- **Multiple PKs on multiple session hosts simultaneously**: systemic table-level or page-level lock, not row-level. Consistent with a background process holding an exclusive lock.
- **Recurring bursts over several hours**: suggests a scheduled process acquiring and releasing the lock repeatedly (e.g., a report running on a schedule).
- **Escalating error count through the morning**: consistent with a long-running maintenance operation (index rebuild, statistics update) whose impact grows as concurrent user activity increases.
- **Blocked table = StorageDocs**: the eDocs document storage table in OdysseyXXX. A lock on StorageDocs blocks any record save that involves attaching or modifying an eDocs entry.

## Reference Example: CS02390824 (July 2026)

- Client: MO4SYR / Mohawk Global Logistics Inc.
- Physical SQL server: `us2wp-ssql-438b\INSTANCE1` → `us2wp-ssql-438b.wisecloud.zone`
- CW DNS alias: `MO4SYR.db.wisegrid.net`
- Main database: `OdysseyMO4SYR`; custom report DB: `OdysseyMO4SYR_UserRepository`
- Customer code: `"mo4"` (userstats), `"MO4SYR"` (performance log `srdh.code`)
- Build at time of incident: 26.2.25.818
- Confirmed blocking: 58 lock timeout errors on `StorageDocs`, 8:15 AM–11:29 AM ET, July 2, 2026
- All errors: ForwardingShipment / Save, eDocsFactory for DB:1, Enterprise.DocumentScanning.Business.StorageFile
- Prior incident: CS01639170 (May 2024) — MGL Milestone Status Report (`fnMGL_Report_MilestoneStatus` in `OdysseyMO4SYR_UserRepository`) caused a 12-minute system-wide block; resolved by WiseCloud rebuilding indexes
