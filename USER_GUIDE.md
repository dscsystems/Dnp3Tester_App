# DNP3 Tester — User Guide

## 1. First start

The app opens the last workspace, or a **demo** workspace with one simulated outstation. Click
**Connect all** on the Dashboard: the simulator comes online, the integrity poll fills the point
database, and events start to flow. Everything below can be tried against it without hardware.

A **workspace** (`*.dnp3proj`) holds sessions, tag maps, checkout plans, alarm rules and settings.
Its historian (`*.dnp3db`, SQLite) sits beside it. Use **Save / Save as / Open / Recent** at the top
of the side panel.

## 2. Sessions

**Sessions → Add TCP / serial / TLS / UDP / simulator.** Key settings:

| Setting | Notes |
|---|---|
| Master / outstation address | Must mirror the device. Wrong addresses give *silence*, not errors — use **Tools → address scan**. |
| Response timeout | Per request. Raise it for slow radios. |
| Disable unsolicited on startup, integrity on startup | The standard startup sequence; also re-run after a device restart. |
| Link confirms, retries, timeout | For serial links. |
| Periodic scans | Class scans (`1,2,3`, `0,1,2,3`) or ranges (`g30v0 0-9`) with a period. |
| Automatic actions | Time sync on NEED_TIME, event poll on class bits, integrity on buffer overflow. |

TLS is always mutually authenticated (IEC 62351-3): give certificate, key and CA (PEM).

The **session bar** at the top of most pages selects the session and gives Connect/Disconnect,
Integrity, Class 1/2/3 and Time sync, plus the current IIN.

## 3. Points

The grid shows every reported (and every *expected*, from the point list) point: value (scaled, with
state label), decoded flags, quality, event timestamp and time quality, receive time, source
(static/event/unsolicited + group/variation + class) and change count. Changed rows flash.

Filter by type, text or glob (`CB*_OPEN`, `BI1?`), bad quality, or "changed in last 60 s". Select a
row for its detail: recent events, **Edit tag**, **Read now**, **Trend**, **Control…**, **Write deadband**.
**Snapshot** saves the whole database to the historian; compare snapshots with "now" on the Reports page.

## 4. Events & SOE

* **Live** — events in receive order, newest first.
* **SOE** — all sessions merged by the outstations' event timestamps.
* **History** — from the historian over any period (`since -8h`, `-7d`, ISO time).

Rows are coloured for invalid (red) and unsynchronized/out-of-order (amber) time. The status line
shows latency percentiles and flags EVENT_BUFFER_OVERFLOW.

## 5. Controls

1. **Arm** in the side panel (not possible while the workspace is read-only). Arming expires after
   the configured idle time.
2. Choose CROB or analog output, index, operation / value and mode:
   * **Select before operate** (default), **Direct operate**, **Direct operate, no ack**
   * **Select only** then **Operate only** — a manual two-step SBO for testing select timeouts and
     NO_SELECT behaviour.
3. **Send** — a confirmation shows what will be sent; critical outputs need the tag typed.

Results show each command status and round trip. If the output's tag defines **feedback**
(`BI12=1@5000`), the tester waits for it and reports *VERIFIED after N ms* or *NOT VERIFIED*.
**Sequences** chain commands with delays (trip, wait 2 s, close).

## 6. Device & operations

* **Device attributes** (group 0): vendor, model, versions, point counts. **Export observed Device Profile** writes an XML skeleton from what the device reports.
* **Configuration compare** against a Device Profile XML or the point list: missing, unexpected, class and variation mismatches, and counts vs. the device's own attributes.
* **Internal indications**: 16 LEDs and a 24-hour IIN history.
* **Operations**: time sync (LAN / delay-measured / recorded), write time or an offset (to test timestamping), delay measure, clear restart, unsolicited on/off, freeze / freeze-and-clear / freeze-no-ack / freeze at time, write attribute, assign class, deadbands, warm/cold restart.
* **Request builder**: any function code with hand-built object headers (group, variation, qualifier, range, data hex). Anything but READ and DELAY_MEASURE needs arming.

## 7. Protocol analyzer

Every byte in and out is captured at the connection stream and decoded: link header (control bits,
addresses, CRC), transport (FIR/FIN/seq), application (control, function, IIN bits, object headers
and values). Click a tree node to highlight its bytes in the hex view.

Filters: direction, text, group, application-only, errors, unsolicited, session. **RTT** shows
request→response time per exchange. **Export pcapng** opens in Wireshark (frames are wrapped in
TCP port 20000, including serial captures). **Open capture…** decodes a pcap/pcapng or a hex dump
(`TX 05 64 …` / `RX …` lines) offline.

**Tools → Serial spy** monitors a serial line passively (one port on a half-duplex bus, or two
taps); frames appear here.

## 8. Trends

Add pens by id (`AI3`) or tag, or with **Trend** on the Points page. Live mode refreshes every second;
longer ranges use aggregates. Binaries and counters draw as steps; events are marked on the axis;
hover for values.

## 9. Tags & point list

Import CSV/XLSX (headers such as *Type, Index, Tag, Description, Units, Scale, Offset, On/Off text,
Class, Area, Bay, Critical* are detected), a Device Profile XML, or paste from Excel. Imports — and
names proposed by AI agents over MCP — become **changesets** that you review (with warnings for
duplicates and unknown points) and accept or reject. Every change is versioned (**History**).

Per point: tag, description, units, scaling, state labels, area/bay, expected value/limits, critical
flag and feedback links; add **alarms** (high/low/state/stale/bad quality).

## 10. Commissioning (point-to-point)

1. **New from point list** creates a plan with default steps per type (binaries ON/OFF, double-bits,
   analog 0/50/100 % injection steps, setpoints for analog outputs).
2. Select a point and press **Start** — the tester waits for the field change and checks value,
   ONLINE flag, that an event arrived, event time validity and (when determinable) class. Control
   points operate and verify feedback.
3. Mark **Pass / Fail / Skip** manually where needed; failures ask for a reason (punch list).
4. **Sign off…** captures tester and witness names and signatures, and locks the results with a
   SHA-256 hash that the report prints and re-verifies.
5. **Report (PDF/XLSX)**.

## 11. Tests & scripts

YAML plans (see the examples in `samples/`) or C# scripts. **Check** parses/compiles; **Run**
with iterations or a duration for soak tests. Results expand to steps and logs; export **JUnit** or
a **report**.

**Conformance-lite** runs 28 behaviour checks modelled on DNP3 subset-level expectations — IIN
error handling, reads, events and confirms, time, restarts, freeze, assign class, SBO/NO_SELECT,
select timeout, no-ack. Control and restart checks are opt-in. It is a commissioning aid, **not** a
substitute for DNP Users Group certification.

## 12. Reports

Pick a type and format (PDF, XLSX, HTML, CSV, JSON). **Preview** renders HTML in the page. Report
header fields (company, project, site, logo) are in Settings.

## 13. Logs & audit

One timeline of application logs, session events, the task monitor and the audit trail. The audit
trail is hash-chained: **Verify audit chain** detects edits. Log files rotate daily under
`%LOCALAPPDATA%\Dnp3Tester\logs`.

## 14. Tools

* **Simulator**: set binaries/analogs, generate an event storm, force a command status (LOCAL,
  NOT_SUPPORTED…), a stuck breaker (no feedback), all points COMM_LOST; run a standalone TCP simulator.
* **Scanners**: IP range for an open DNP3 port; outstation addresses on one endpoint.
* **Serial spy**, **Fault injection** (corrupt/drop received bytes to test link recovery).

## 15. MCP server

See [MCP.md](MCP.md). The MCP page shows the endpoint, token, permissions and live activity.

## Safety

* Controls, raw requests, restarts and file writes require **arming**; arming expires when idle.
* **Read-only** workspaces block every write to the outstation. New workspaces open read-only by default.
* Every control is confirmed in a dialog; **critical** outputs need the tag typed.
* MCP controls are off by default; when enabled, each still needs the tester armed **and** your approval in a dialog.
* Everything — controls, confirmations, arming, tag changes, MCP calls — is in the audit trail.

## Keyboard and CLI

The headless `dnp3tester` CLI (`dnp3tester --help`) runs plans, conformance, the MCP server,
the simulator, polls, reports, decoding and scans — useful in CI and on lab rigs.
