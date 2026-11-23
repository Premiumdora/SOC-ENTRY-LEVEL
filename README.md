# Day 1 — PowerShell Script Block Logging
### Investigating Event ID 4104 in Windows Event Viewer

PowerShell is one of the most common living-off-the-land tools — no malware to drop, no antivirus to trip. The control that keeps it visible to defenders is **Script Block Logging**: every script PowerShell runs gets written to Event ID 4104, text and all. Day 1 of this project is a hands-on investigation of that logging channel — locating a real event, reading it two different ways, and confirming the two readings agree.

---

## Table of Contents
- [Objectives](#objectives)
- [Environment](#environment)
- [Methodology](#methodology)
- [Evidence & Findings](#evidence--findings)
- [Security Assessment](#security-assessment)
- [Skills Demonstrated](#skills-demonstrated)
- [Status](#status)

---

## Objectives

- Explain why Script Block Logging matters for SOC-level visibility into PowerShell activity.
- Navigate Windows Event Viewer to the correct log channel for this event source.
- Locate a 4104 record and read it in both XML View (what a SIEM ingests) and Friendly View (what an analyst reads day to day).
- Confirm both views agree on every field rather than assuming they do.
- Report the finding in an evidence-based way: what the log proves, and what's still open.

## Environment

| Component | Detail |
|---|---|
| Operating system | Windows desktop |
| Primary tool | Windows Event Viewer (`eventvwr.msc`) |
| Secondary tools | Windows PowerShell, Group Policy Editor |
| Log channel | `Microsoft-Windows-PowerShell/Operational` |
| Event ID investigated | 4104 |
| Events in channel at capture | 631 |

## Methodology

1. **Confirm logging is enabled.** Script Block Logging is configured via Group Policy (`Computer Configuration → Administrative Templates → Windows Components → Windows PowerShell`). Its effect — events actually appearing in the Operational log — is the indirect confirmation used here.
2. **Open the correct log channel.** Event Viewer → Applications and Services Logs → Microsoft → Windows → PowerShell → Operational.
3. **Locate a 4104 record** and open its Details tab.
4. **Read the record twice** — once in XML View, once in Friendly View — and compare every field between the two.

## Evidence & Findings

![Event ID 4104 in XML View](fig1-4104-xml.png)
*Figure 1 — XML View of Event ID 4104: timestamp, process ID, channel, and security identifier, all machine-readable — the format a SIEM would ingest.*

| Field | Observed value |
|---|---|
| Event ID / channel | 4104 / `Microsoft-Windows-PowerShell/Operational` |
| Event record ID | 631 |
| Displayed local time | 23/09/2026, 02:20:34 |
| XML `TimeCreated` (UTC) | 2026-09-23T11:20:34Z |
| Level / task / opcode | Verbose (5) / Task 2 / Opcode 15 |
| Process ID / thread ID | 1864 / 8256 |

**Reading the evidence:** the displayed time (02:20:34) is **9 hours behind** the XML's UTC timestamp (11:20:34Z). That gap shouldn't exist for a host that should be on UTC+1 — it points to a clock-sync problem that would silently break log correlation the moment this host's events are compared against another machine's. Worth flagging on its own, independent of anything else in the record.

![Event ID 4104 in Friendly View](fig2-4104-friendly.png)
*Figure 2 — The same event in Friendly View: `ScriptBlockText = prompt`, PowerShell's own routine function for drawing its command line.*

| Field | Observed value |
|---|---|
| MessageNumber / MessageTotal | 1 / 1 |
| ScriptBlockText | `prompt` |
| ScriptBlockId | `9c7bda9b-c4ec-4899-a243-11f9c69622f1` |
| Path | Empty (ran interactively, not from a script file) |

**Reading the evidence:** `prompt` is logged routinely every time PowerShell draws its command line — it is not suspicious on its own. The value that matters here is that **both views agree on every field**: MessageNumber, MessageTotal, ScriptBlockText, and ScriptBlockId all match between XML and Friendly View, which confirms the logging pipeline is reliable and consistent regardless of which view an analyst chooses to work in.

## Security Assessment

| Investigation area | Finding | Status |
|---|---|---|
| Script Block Logging active | 631 events present in channel | ✅ Confirmed |
| Event ID 4104 located and read | XML and Friendly View both reviewed | ✅ Confirmed |
| XML vs. Friendly View consistency | All fields match | ✅ Confirmed |
| Script content | `prompt` — benign | ✅ Confirmed, no escalation |
| Host clock accuracy | 9-hour discrepancy from UTC | ⚠️ Flagged — needs correction |
| Planned test command (`Get-LocalUser`) | Not present in captured evidence | ⏳ Pending |

**Disposition:** benign/undetermined content on this record alone; no escalation warranted. The clock-drift finding is a legitimate, separate action item regardless of the benign script content.

## Skills Demonstrated

- Windows Event Viewer navigation to a specific log channel
- XML and Friendly View event interpretation
- UTC vs. local timestamp verification and clock-drift detection
- Distinguishing a routine PowerShell artifact from a planned test case
- Evidence-based reporting: separating what's confirmed from what's still open

## Status

- [x] Script Block Logging confirmed active (631 events in channel)
- [x] Event ID 4104 located and read in both XML and Friendly View
- [x] Clock-drift finding identified and documented
- [ ] Planned `Get-LocalUser` test command — not yet located in captured evidence
- [ ] Group Policy logging configuration — not directly screenshotted (inferred from active logging)

---

*Part of a daily hands-on SOC analyst project. Full narrative version: Medium — "Reading Windows the Way a SOC Analyst Does."*
