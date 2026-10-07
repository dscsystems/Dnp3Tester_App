# DNP3 Tester

A DNP3 (IEEE 1815) **master** for testing and commissioning outstations — RTUs, IEDs, gateways.
Desktop GUI (.NET MAUI, Windows), headless CLI, and an **MCP server** that gives AI agents
live point data, history, events, logs and reports, and lets them propose tag names.

Built on [SharpDnp3](https://github.com/dscsystems/SharpDnp3).

## What it does

| Area | Highlights |
|---|---|
| Connections | TCP, TLS (mutual auth), UDP, serial, in-process simulator; multiple sessions; address and IP scanners |
| Point database | Every point type with decoded flags, time quality, source/group-variation, change highlighting, scaling and state labels |
| Events & SOE | Live event list, cross-session SOE by event time, latency, invalid/out-of-order time checks, historian queries |
| Operations | Integrity/class/range reads, time sync (LAN, delay-measured and recorded), write time, cold/warm restart, unsolicited enable/disable, freeze / freeze-clear / freeze at time, write device attributes, frozen analogs and command events, file authentication, Secure Authentication (symmetric subset), assign class, deadbands, delay measure, clear restart, device attributes (g0), file transfer (g70) |
| Controls | CROB and analog outputs, SBO / DO / DO-no-ack, manual SELECT-only and OPERATE-only, sequences, feedback verification, arming, typed confirmation for critical points |
| Protocol analyzer | Every frame tapped at the stream, decoded link/transport/application tree with hex highlighting, request/response timing, filters, pcapng export for Wireshark, offline pcap/hex decode, passive serial spy |
| Commissioning | Point list / Device Profile import, configuration compare, guided point-to-point checkout, verdicts, punch list, signature capture, hash-locked sign-off, PDF/XLSX report |
| Automation | YAML test plans, C# scripts (Roslyn), 28-check conformance-lite suite, loops/soak, JUnit, headless CLI |
| Reports | Commissioning, configuration compare, SOE, point history, communications, control log, snapshot, audit, tag changes, point list — PDF, XLSX, HTML, CSV, JSON |
| History & audit | SQLite historian per workspace, retention, snapshots; hash-chained audit trail of every operator and AI action |
| MCP | Streamable HTTP (in the GUI or headless) and stdio; read tools, resource subscriptions, staged tag naming, gated operations with human confirmation |

## Releases

| Download | Platforms |
|---|---|
| `Dnp3Tester-<ver>-win-<arch>-setup.exe` — installer (GUI + CLI, optional add-to-PATH) | Windows 10 1809+ x64, arm64 |
| `Dnp3Tester-<ver>-win-<arch>-portable.zip` — same, no install | Windows x64, arm64 |
| `dnp3tester-cli-<ver>-<rid>.zip` / `.tar.gz` — self-contained CLI | win-x64, win-arm64, linux-x64, linux-arm64, osx-x64, osx-arm64 |

## Safety

Controls are disarmed by default, workspaces can open read-only, arming auto-expires, every control
is confirmed and audited, and MCP controls are off unless enabled — and even then each one needs a
human to approve it in the tester. See [USER_GUIDE.md](USER_GUIDE.md#safety).

## License

DNP3 Tester is **source-available** under the [DNP3 Tester Source-Available License](LICENSE.md):
free for **education**, **non-commercial** use and **evaluation**. Any other use — including
commissioning, testing or troubleshooting done by or for a business — needs a commercial license
from DSC Systems (https://dscsys.com, mailto:admin@dscsys.com).

Bundled components keep their own licenses — see [THIRD-PARTY-NOTICES.md](THIRD-PARTY-NOTICES.md).
In particular, SharpDnp3 (`lib/SharpDnp3`) remains GPL-3.0-or-later; other third-party packages
are MIT, Apache-2.0 or MS-PL.
