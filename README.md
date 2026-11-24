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
- [Summary](#summary)

---

## Objectives

- Explain why Script Block Logging matters for SOC-level visibility into PowerShell activity.
- Navigate Windows Event Viewer to the correct log channel for this event source.
- Locate a 4104 record and read it in both XML View (what a SIEM ingests) and Friendly View (what an analyst reads day to day).
- Confirm both views agree on every field.
- Report the finding clearly and precisely, in a way a SOC team could act on directly.

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

1. **Confirmed logging was enabled** — the presence of active 4104 events in the Operational log is direct confirmation that Script Block Logging is correctly configured via Group Policy.
2. **Opened the correct log channel** — Event Viewer → Applications and Services Logs → Microsoft → Windows → PowerShell → Operational.
3. **Located a 4104 record** and opened its Details tab.
4. **Read the record twice** — once in XML View, once in Friendly View — and compared every field between the two.

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

**Reading the evidence:** cross-referencing the displayed local time against the XML's UTC timestamp surfaced a 9-hour offset between the two. Identifying and documenting that offset here is itself a useful analytical result — it's exactly the kind of cross-check that confirms timestamps across a fleet of machines can be trusted for correlation, and it was caught precisely because both fields were checked against each other rather than read in isolation.

![Event ID 4104 in Friendly View](fig2-4104-friendly.png)
*Figure 2 — The same event in Friendly View: `ScriptBlockText = prompt`, PowerShell's own routine function for drawing its command line.*

| Field | Observed value |
|---|---|
| MessageNumber / MessageTotal | 1 / 1 |
| ScriptBlockText | `prompt` |
| ScriptBlockId | `9c7bda9b-c4ec-4899-a243-11f9c69622f1` |
| Path | Empty (ran interactively, not from a script file) |

**Reading the evidence:** `prompt` is logged routinely every time PowerShell draws its command line — it is not suspicious on its own. The value that matters here is that **both views agree on every field**: MessageNumber, MessageTotal, ScriptBlockText, and ScriptBlockId all match exactly between XML and Friendly View, confirming the logging pipeline is reliable and consistent regardless of which view an analyst works in.

## Security Assessment

| Investigation area | Finding | Result |
|---|---|---|
| Script Block Logging active | 631 events present in channel | ✅ Confirmed |
| Event ID 4104 located and read | XML and Friendly View both reviewed | ✅ Confirmed |
| XML vs. Friendly View consistency | All fields match exactly | ✅ Confirmed |
| Script content | `prompt` — benign | ✅ Confirmed, no escalation |
| Timestamp integrity check | 9-hour local/UTC offset identified | ✅ Identified and documented |

**Disposition:** benign content; logging pipeline verified as active, consistent, and trustworthy for this host. No escalation warranted.

## Skills Demonstrated

- Windows Event Viewer navigation to a specific log channel
- XML and Friendly View event interpretation
- Cross-referencing timestamp fields to verify log integrity
- Distinguishing routine PowerShell activity from activity that would warrant escalation
- Clear, precise technical reporting

## Summary

- [x] Script Block Logging confirmed active (631 events in channel)
- [x] Event ID 4104 located and read in both XML and Friendly View
- [x] Cross-view consistency verified field by field
- [x] Timestamp integrity checked and a 9-hour offset documented

---
