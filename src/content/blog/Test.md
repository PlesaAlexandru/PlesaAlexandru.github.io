---
title: "LetsDefend: Event ID 102 - Suspicious PowerShell Execution"
description: "SOC L1 investigation walkthrough covering alert triage, Base64 decoding, network analysis, and endpoint remediation."
pubDate: 2026-10-09
categories: ["SOC", "Investigations"]
tags: ["letsdefend", "powershell", "sysmon", "triage", "mitre"]
---

## 1. Alert Summary

During routine monitoring in the SIEM console, an alert triggered regarding an anomalous PowerShell command executed on an accounting endpoint.

| Field | Value |
| :--- | :--- |
| **Alert Name** | Suspicious Encoded PowerShell Command |
| **Event Time** | Oct 09, 2026 - 11:24:15 UTC |
| **Hostname** | `WKSTN-FIN-04` |
| **Source User** | `contoso\m.popescu` |
| **MITRE ATT&CK** | **T1059.001** (PowerShell), **T1071.001** (Web Protocols) |
| **Initial Severity** | High |

---

## 2. Initial Triage & Process Tree

Examining the endpoint telemetry via **Sysmon Event ID 1** (Process Creation) revealed a suspicious parent-child process relationship:

```text
explorer.exe (PID: 2840)
  └── EXCEL.EXE (PID: 4912)
        └── powershell.exe (PID: 6108)
```

