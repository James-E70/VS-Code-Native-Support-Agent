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

## ES|QL: Use the Internal Search API via `page.evaluate`

The Kibana Discover ES|QL browser editor corrupts long queries. The `api/console/proxy` endpoint is **disabled** in Kibana 9.x. Always use the internal search API via `page.evaluate` + `fetch`:

```javascript
const result = await page.evaluate(async () => {
  const response = await fetch('/s/wisecloudsupport/internal/search/esql', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json', 'kbn-xsrf': 'true', 'x-elastic-internal-origin': 'Kibana' },
    body: JSON.stringify({
      params: { query: 'FROM logs-cargowise.sessionhost.performance-wtg | WHERE ...' }
    })
  });
  const data = await response.json();
  return data.rawResponse || data;
});
// Results are in result.values (column-oriented array); columns in result.columns
```

**Critical:** The body must use `{ params: { query: '...' } }` — NOT `{ query: '...' }` at the top level. The three required headers are `Content-Type`, `kbn-xsrf`, and `x-elastic-internal-origin`. Call this inside `run_playwright_code` with `page.evaluate(async () => { ... })`.

## All Confirmed Data Views — wisecloudsupport Space (APAC, verified July 2026)

### Available ✅

| Data View | Key Fields | Best For |
|---|---|---|
| `logs-cargowise.sessionhost.performance-wtg` | `srdh.code`, `srdh.duration`, `srdh.message`, `srdh.object`, `srdh.op`, `srdh.host`, `srdh.server` | **PRIMARY** — SQL Error 1222 (lock request timeout) ONLY appears here; slow DB operations; error patterns |
| `logs-cargowise.rdgateway.userstats-wtg` | `srdg.Customer`, `srdg.Username`, `srdg.FullUserName`, `srdg.ClientAddress`, `srdg.ConnectedResource`, `srdg.ConnectionDuration`, `srdg.NumberOfKilobytesReceived`, `srdg.NumberOfKilobytesSent`, `srdg.IdleTime` | **RDP connection health and client-side throughput** — diagnose per-user lag, throttled RDP streams, client PC bottlenecks |
| `logs-cargowise.sessionhost.userstats-wtg` | `srdh.Customer`, `srdh.Username`, `srdh.UnifiedSessionId`, `srdh.usercount`, `message` | Session counts per host — verify whether sessions dropped during an incident |
| `logs-cargowisecloud.blazor-wtg` | `CWC.EnterpriseCode`, `CWC.Username`, `CWC.LoginName`, `CWC.FormName`, `CWC.ElapsedMilliseconds`, `CWC.ExecutionTimeMs`, `CWC.ExceptionMessage`, `CWC.ExceptionDetails`, `CWC.CloseReason`, `CWC.FailureMessage`, `CWC.IsSuccess`, `CWC.DPI`, `CWC.CircuitId` | **CargoWise Next (Blazor) app events** — errors, exceptions, session close reasons, per-form performance, DPI issues |
| `logs-cargowisecloud.general-wtg` | `CWC.*` (broad set of infrastructure/auth fields) | General CargoWise Cloud infrastructure and authentication events |
| `logs-wtg.application.logs-wtg` | `ExceptionMessage`, `ServiceTaskCode`, `action`, `al.attributes.*` | Application-level exceptions and service task errors |
| `logs-cargowise.processcontroller.servicetask-wtg` | `CWLicenseCode`, `ActionCode`, `BranchCode`, `CWDatabaseName`, `CWDatabaseHost`, `CWProductVersion`, `AlLoggerName` | Background service task execution — duration, action codes, per-license activity |
| `logs-cargowise.processcontroller.servicetaskqueue-wtg` | Standard ECS fields + queue metadata | Service task queue state and scheduling activity |
| `logs-microsoft_dnsserver.analytical-wtg` | `dns.question.name`, `dns.question.type`, `dns.response_code`, `source.ip`, `destination.ip`, `microsoft_dnsserver.analytical.elapsed_time` | DNS resolution across WiseCloud — useful for connectivity failures, hostname resolution errors |
| `logs-inventory.cw1iis.sites-wtg` | `cw1iis.payload.Customer`, `cw1iis.payload.State`, `cw1iis.payload.DB`, `cw1iis.payload.DBSRV`, `cw1iis.payload.CNAME`, `cw1iis.payload.HealthCheckConfig.IsUpgrading`, `cw1iis.payload.HealthCheckConfig.UpgradeStartTime`, `cw1iis.payload.Stage` | IIS site inventory — confirm if a CW1 site is currently upgrading, in maintenance, or has a DB server assignment |

### Not Available in wisecloudsupport Space ❌

- `logs-glow.performance-wtg` — Unknown index (not in this space)
- `logs-microsoft_sqlserver.log-wtg` — SQL Server error log (deadlocks, Error 17830); **not listed in the visible index list but worth trying** — returns Unknown index in APAC wisecloudsupport as of July 2026
- `logs-microsoft_sqlserver.audit-wtg`
- `metrics-microsoft_sqlserver.transaction_log-wtg`
- `metrics-sqlcpumonitoring-wtg` (Architecture space only — no support access)

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

> **Note:** `srdh.Customer` does NOT exist in the APAC wisecloudsupport index. Use `srdh.code` with a wildcard to discover the license code: `WHERE srdh.code LIKE "EX5*"`.

## RD Gateway Log Fields (`logs-cargowise.rdgateway.userstats-wtg`)

| Field | Description | Example |
|---|---|---|
| `srdg.Customer` | Customer code (lowercase, 2–3 chars) | `"ex5"` |
| `srdg.Username` | WiseCloud login name | `"ex5.ian.mclucas"` |
| `srdg.FullUserName` | Display name | `"Ian McLucas"` |
| `srdg.ClientAddress` | Client IP as seen by RD Gateway | `"10.2.64.13"` |
| `srdg.ConnectedResource` | Session host FQDN the user landed on | `"AU2WP-SRDH-466.wisegrid.net"` |
| `srdg.ConnectionDuration` | Running session duration counter | `"00000000022839.000000:000"` |
| `srdg.IdleTime` | Idle time counter | `"00000000000146.000000:000"` |
| `srdg.NumberOfKilobytesReceived` | Cumulative KB the gateway received from the client (keyboard/mouse input) | `3897` |
| `srdg.NumberOfKilobytesSent` | Cumulative KB the gateway sent to the client (screen updates) | `4722` |
| `srdg.usercount` | User count on this gateway at snapshot time | — |

> **Throughput interpretation:** Snapshots are recorded every ~5 minutes. A healthy active WiseCloud RDP session typically accumulates 200–500 KB **received** per 5-minute interval. If a user's cumulative KB received grows by <10 KB per interval while other users on the same session host are accumulating 200+ KB, the bottleneck is on the client side (old PC, degraded WiseCloud client, poor network). Confirmed in CS02400183 (July 2026): Ian McLucas (EX5SYD) received 3.8 MB total vs 142 MB for another user on the same session host.

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

## ES|QL Queries — RD Gateway Throughput (Client-Side Lag)

### Discover license/customer code (if unknown)
```esql
FROM logs-cargowise.sessionhost.performance-wtg
| WHERE srdh.code LIKE "EX5*"
| STATS count = COUNT(*) BY srdh.code | SORT count DESC | LIMIT 10
```
> Note: `srdg.Customer` in the RD Gateway log is lowercase (e.g. `"ex5"`).

### All RD Gateway sessions for a customer today
```esql
FROM logs-cargowise.rdgateway.userstats-wtg
| WHERE srdg.Customer == "ex5" AND @timestamp >= "2026-07-13T00:00:00.000Z"
| KEEP @timestamp, srdg.Username, srdg.FullUserName, srdg.ClientAddress, srdg.ConnectedResource, srdg.ConnectionDuration, srdg.IdleTime, srdg.NumberOfKilobytesReceived, srdg.NumberOfKilobytesSent
| SORT @timestamp DESC | LIMIT 30
```

### Throughput comparison — all users in a time window (spot the outlier)
```esql
FROM logs-cargowise.rdgateway.userstats-wtg
| WHERE srdg.Customer == "ex5" AND @timestamp >= "2026-07-13T05:00:00.000Z" AND @timestamp <= "2026-07-13T08:00:00.000Z"
| STATS max_kb_recv = MAX(srdg.NumberOfKilobytesReceived), max_kb_sent = MAX(srdg.NumberOfKilobytesSent), sessions = COUNT(*) BY srdg.Username
| SORT max_kb_recv DESC | LIMIT 10
```

### All users on a specific session host today (rule out host-side issue)
```esql
FROM logs-cargowise.rdgateway.userstats-wtg
| WHERE srdg.ConnectedResource == "AU2WP-SRDH-466.wisegrid.net" AND @timestamp >= "2026-07-13T00:00:00.000Z"
| STATS max_kb_recv = MAX(srdg.NumberOfKilobytesReceived), max_kb_sent = MAX(srdg.NumberOfKilobytesSent), sessions = COUNT(*) BY srdg.Username, srdg.Customer
| SORT max_kb_recv DESC | LIMIT 15
```
> If other users on the same host have normal throughput (100+ MB) and the affected user has <5 MB, the session host is not the cause — the bottleneck is on the client side.

### CargoWise Next (Blazor) exceptions for a customer
```esql
FROM logs-cargowisecloud.blazor-wtg
| WHERE CWC.EnterpriseCode == "EX5" AND @timestamp >= "2026-07-13T00:00:00.000Z"
| WHERE CWC.ExceptionMessage IS NOT NULL
| KEEP @timestamp, CWC.Username, CWC.FormName, CWC.ExceptionMessage, CWC.ExceptionType, CWC.FailureMessage
| SORT @timestamp DESC | LIMIT 20
```

### IIS site state — check if a CW1 site is currently upgrading
```esql
FROM logs-inventory.cw1iis.sites-wtg
| WHERE cw1iis.payload.Customer == "EX5SYD"
| KEEP @timestamp, cw1iis.payload.State, cw1iis.payload.HealthCheckConfig.IsUpgrading, cw1iis.payload.HealthCheckConfig.UpgradeStartTime, cw1iis.payload.DBSRV, cw1iis.payload.Stage
| SORT @timestamp DESC | LIMIT 5
```

## Hostname Gotchas

- CW DNS alias format: `{CWCODE}.db.wisegrid.net` — appears in error messages but is NOT the server name Kibana indexes. Using it in `host.name` filters returns no results.
- Physical SQL server format: `us2wp-ssql-NNN.wisecloud.zone` — get from the incident, Windows service details, or the `srdh.server` field in the performance log.
- Session hosts (APAC): `au2wp-srdh-NNN.wisegrid.net` (in RD Gateway log) or `AU2WP-SRDH-NNN` (in performance log `srdh.host`).
- Session hosts (AMER): `us2wp-srdh-NNN.wisecloud.zone`
- Customer code in performance log (`srdh.code`): uppercase license code, e.g. `"EX5SYD"`, `"MO4SYR"`
- Customer code in RD Gateway and sessionhost userstats logs (`srdg.Customer`, `srdh.Customer`): lowercase short code, typically first 2–3 chars, e.g. `"ex5"`, `"mo4"`

## Interpreting Lock Timeout Patterns

- **Same PK 3+ times within 90 seconds**: one user repeatedly retrying a blocked save. Confirms blocking was sustained, not momentary.
- **Multiple PKs on multiple session hosts simultaneously**: systemic table-level or page-level lock, not row-level. Consistent with a background process holding an exclusive lock.
- **Recurring bursts over several hours**: suggests a scheduled process acquiring and releasing the lock repeatedly (e.g., a report running on a schedule).
- **Escalating error count through the morning**: consistent with a long-running maintenance operation (index rebuild, statistics update) whose impact grows as concurrent user activity increases.
- **Blocked table = StorageDocs**: the eDocs document storage table in OdysseyXXX. A lock on StorageDocs blocks any record save that involves attaching or modifying an eDocs entry.

## Reference Example: CS02400183 (July 2026) — Client-Side RDP Lag

- Client: EX5SYD / Export Freight Systems Pty Ltd (EXPFRESYD)
- Symptom: 30–60 second keypress lag for one user (ex5.ian.mclucas), other users unaffected
- Session host: AU2WP-SRDH-466.wisegrid.net
- Kibana findings:
  - Zero SQL lock timeouts, zero errors in performance log
  - RD Gateway throughput: Ian = 3.8 MB received; other users on same host = 40–142 MB received
  - 18.7× throughput deficit vs the unaffected `ex5.operations` user in the same time window
  - Session host confirmed healthy (other users working normally at full throughput)
- Root cause: Old PC hardware unable to process RDP stream; stale WiseCloud client
- Fix: Reinstall CargoWise Cloud Client on Ian's PC

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
