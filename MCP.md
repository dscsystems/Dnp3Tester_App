# MCP server

DNP3 Tester exposes a [Model Context Protocol](https://modelcontextprotocol.io) server so AI agents
can read live and historical data, events, logs and reports, and propose tag names.

## Endpoints

| Mode | How |
|---|---|
| In the GUI | Starts with the workspace (MCP page). Streamable HTTP at `http://127.0.0.1:47020/mcp`. |
| Headless HTTP | `dnp3tester serve --workspace site.dnp3proj` (or `--host`, `--sim`). Prints endpoint and token. |
| stdio | `dnp3tester mcp-stdio --workspace site.dnp3proj` for clients that only launch stdio servers. |

Every HTTP request needs `Authorization: Bearer <token>` (MCP page → Copy token). The server binds
to loopback by default and rejects browser requests from non-local origins.

Claude Code / Claude Desktop style configuration:

```json
{
  "mcpServers": {
    "dnp3tester": {
      "type": "http",
      "url": "http://127.0.0.1:47020/mcp",
      "headers": { "Authorization": "Bearer <token>" }
    }
  }
}
```

stdio:

```json
{
  "mcpServers": {
    "dnp3tester": { "command": "dnp3tester", "args": ["mcp-stdio", "--workspace", "C:/work/site.dnp3proj"] }
  }
}
```

## Tools

### Read (always available)

| Tool | Purpose |
|---|---|
| `list_sessions` | Sessions, state, addresses, IIN, point counts |
| `get_session_status` | State, stats, set IIN bits with meaning, response times, armed/read-only |
| `query_points` | Realtime point DB: session, types, index range, glob search, bad quality, changed since; paged |
| `get_point` | One point (by type/index or tag) with metadata and recent changes |
| `get_point_history` | Raw samples or aggregates (min/max/avg/first/last per interval) |
| `query_events`, `get_soe` | Events with event time, latency, class, unsolicited; SOE across sessions |
| `get_iin_history` | IIN bits raised/cleared over time |
| `get_statistics` | Counters, response-time percentiles, availability |
| `query_protocol_log` | Decoded frames, filterable; optional full decode and hex |
| `query_app_log`, `query_audit_log` | Application/session log; audit trail with hash-chain check |
| `get_alarms`, `get_device_info`, `get_checkout_status`, `compare_configuration` | Alarms, g0 attributes, commissioning progress, expected-vs-observed |
| `generate_report`, `list_reports` | Any report as PDF/XLSX/HTML/CSV/JSON; returns a `dnp3://reports/…` resource URI (CSV/JSON also inline) |

Times accept ISO-8601 or relative values (`-15m`, `-2h`, `-1d`).

### Tag naming (on by default, staged)

| Tool | Purpose |
|---|---|
| `get_tag_map` | Current names and metadata |
| `propose_tag_names` | Entries `{type, index, tag, description?, units?, stateLabels?, area?, bay?, scaleMultiplier?, scaleOffset?}` → a changeset for operator review (or applied if auto-accept is on); returns warnings |
| `import_point_list` | CSV text with a header row → changeset |
| `set_point_expectations` | Expected values/limits/flags → changeset |
| `get_changeset` | Pending / accepted / rejected |

### Operations (off by default)

| Tool | Needs |
|---|---|
| `integrity_poll`, `scan_classes`, `read_range`, `read_device_attributes`, `time_sync`, `freeze_at_time`, `write_attribute` | *Allow wire reads* |
| `set_unsolicited`, `connect_session` | *Allow session management* |
| `operate_control` (CROB/AO, SBO/DO/DONR) | *Allow controls* + tester **armed** + **operator approval in a dialog for every call**; returns status and feedback verification |

## Resources

Change notifications (`notifications/resources/updated`, throttled to 1 s) work with both protocol
generations:

* **2026-07-28 revision (stateless):** open a `subscriptions/listen` request with
  `notifications.resourceSubscriptions` set to the URIs you want. The server first sends
  `notifications/subscriptions/acknowledged` listing the URIs it granted (unsupported ones are
  left out), then streams `notifications/resources/updated` on that request's response stream until
  you cancel it. Every message carries the listen request id in
  `_meta["io.modelcontextprotocol/subscriptionId"]`. List-changed notifications are not offered —
  the tool, prompt and resource lists do not change at runtime.
* **2025-11-25 and earlier (initialize handshake):** `resources/subscribe` / `resources/unsubscribe`
  on the session.

Updates are sent for: `dnp3://workspace`, `dnp3://sessions/{session}`,
`dnp3://sessions/{session}/points`, `dnp3://sessions/{session}/points/{type}/{index}`,
`dnp3://sessions/{session}/events/recent` and `dnp3://logs/audit/recent`.

All resources:

| URI | Content |
|---|---|
| `dnp3://workspace` | Summary, safety state, permissions |
| `dnp3://sessions/{session}` | Session status |
| `dnp3://sessions/{session}/points` | Point database |
| `dnp3://sessions/{session}/points/{type}/{index}` | One point |
| `dnp3://sessions/{session}/events/recent` | Last 200 events |
| `dnp3://sessions/{session}/device` | Attributes and counts |
| `dnp3://tags/{session}` | Tag map |
| `dnp3://logs/{protocol|app|session|audit}/recent` | Recent log entries |
| `dnp3://reports/{id}` | Generated report (text or blob) |

## Prompts

`health_summary`, `commissioning_summary`, `diagnose_comm_issue`, `name_points_from_list`.

## Audit

Every tool call is recorded in the audit trail as `mcp:<client name>` with the tool, an argument
hash and the outcome, and shown live on the MCP page.
