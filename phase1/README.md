# NetSage AI — Phase 1: Network Troubleshooting Case Dataset

## Purpose

Phase 1 creates a curated, Packet Tracer-compatible training dataset for **NetSage AI — AI Troubleshooting Helper with Human Review**. It deliberately documents one fault at a time in a small, known-working topology. The result is a reliable evidence-to-root-cause reference; it is not an AI system, dashboard, or automated checker.

`cases.csv` contains exactly **30** realistic troubleshooting cases: four VLAN, four IP addressing/default gateway, four DHCP, three DNS, five routing, three ACL, three NAT, and four wireless cases.

## Dataset file

`cases.csv` is a UTF-8 comma-separated file with a header row and quoted text fields, so it can be opened directly in Excel, LibreOffice, or a spreadsheet-aware data tool. Semicolons within a field separate commands or observations without breaking CSV rows.

| Column | Meaning |
|---|---|
| `case_id` | Stable unique identifier grouped by technology. |
| `case_title` | Short human-readable scenario name. |
| `symptom` | What the user or operator observes. |
| `topology_note` | Minimal Packet Tracer topology and addressing context. |
| `show_commands` | IOS, PC command-prompt, or Packet Tracer GUI checks to collect. |
| `show_output` | Concise expected evidence from those checks. |
| `expected_fault` | Single intended root cause. |
| `osi_layer` | Relevant OSI layer number. |
| `concept` | Technology tag for filtering or future model evaluation. |
| `severity` | Operational impact: Low, Medium, or High. |
| `expected_fix` | Corrective configuration/action and a validation cue. |

## Case coverage

| Category | Count | Case IDs |
|---|---:|---|
| VLAN | 4 | VLAN-01 through VLAN-04 |
| IP addressing and gateway | 4 | IP-01 through IP-04 |
| DHCP | 4 | DHCP-01 through DHCP-04 |
| DNS | 3 | DNS-01 through DNS-03 |
| Routing | 5 | ROUTE-01 through ROUTE-05 |
| ACL | 3 | ACL-01 through ACL-03 |
| NAT | 3 | NAT-01 through NAT-03 |
| Wireless | 4 | WLAN-01 through WLAN-04 |

## Packet Tracer recreation workflow

Use standard Packet Tracer devices: **PC-PT**, **Laptop-PT**, **Server-PT**, a **Cisco 2960** switch, a **1941 or 2911** router, and a **Wireless Router or Access Point** where applicable. Detailed, case-by-case build instructions are in [case_creation_guide.md](case_creation_guide.md).

For every case, follow this controlled workflow:

```text
Create working topology
→ Verify connectivity
→ Introduce one fault
→ Observe symptom
→ Run troubleshooting commands
→ Save evidence
→ Record correct root cause in cases.csv
```

1. Build the listed devices and links, and apply the stated initial working configuration.
2. Verify the healthy baseline using the guide’s verification command.
3. Make **only** the listed fault. Keep addressing and device names consistent with the scenario.
4. Reproduce the symptom and collect the specified command output or GUI observation.
5. Compare the evidence with `show_output`, identify the documented fault, apply the fix, and verify recovery.

Packet Tracer GUI evidence (for Server-PT services, wireless settings, and Laptop-PT wireless association) is intentionally named where an IOS `show` command is unavailable. Save each completed `.pkt` file separately if retaining lab artifacts; those binary topology files are intentionally not part of this CSV dataset.
