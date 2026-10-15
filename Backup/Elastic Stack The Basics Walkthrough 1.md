---
title: "Elastic Stack: The Basics Walkthrough"
description: Understand how SOC analysts use the Elastic Stack (ELK) for log investigations. Elastic basics and introduction for beginners.
pubDate: 2026-10-14
categories:
  - SOC
  - Walkthrough
  - TryHackMe
tags:
  - TryHackMe
  - Walkthrough
  - Rooms
image: /image/Elastic_Stack_The _Basics.png
---
---
## Summary



---

## 1. Architectural Components of the Elastic Stack

The Elastic Stack combines modular open-source components into a unified pipeline:

![](../../assets/Pasted%20image%2020261014214926.png)

### A. Elasticsearch
A distributed JSON-based document store and full-text analytics engine acting as the database tier. It exposes REST APIs for searching, aggregating, and correlating vast volumes of telemetry.

### B. Logstash
A server-side data processing and normalization pipeline. Configuration files are divided into three functional blocks:
1. **Input:** Specifies the ingestion source (network ports, Beats agents, log files).
2. **Filter:** Normalizes and enriches raw records (e.g., Grok parsing, IP geolocation, field extraction).
3. **Output:** Defines destination targets (Elasticsearch cluster, external queues, storage).

![](../../assets/Pasted%20image%2020261014215041.png)

### C. Beats
Single-purpose, low-footprint data shippers installed across endpoints:
* **Winlogbeat:** Collects Windows Event logs, PowerShell, and Sysmon telemetry.
* **Packetbeat:** Sniffs network sockets and analyzes application protocols.
* **Filebeat:** Tunnels flat-file logs (syslog, web traffic, audit logs).

![Beats Overview](../../assets/Pasted%20image%2020261014215104.png)

### D. Kibana
The administrative and analytical web console. It enables SOC analysts to explore raw documents, build detection visualizations, manage index patterns, and organize investigations into dashboards.

![](../../assets/Pasted%20image%2020261014215137.png)

---

## 2. Kibana Discover Interface & Workflow

The **Discover** tab serves as the primary hunting and log triage interface:

![](../../assets/Pasted%20image%2020261014221111.png)

* **Index Patterns:** Tell Kibana which Elasticsearch index datasets to target (e.g., `vpn_connections`).
![](../../assets/Pasted%20image%2020261014221140.png)
* **Fields Pane:** Highlights parsed field keys and provides quick statistics (Top 5 values) for fast filtering without typing complex queries.
![](../../assets/Pasted%20image%2020261014221224.png)
* **Time Filter & Timeline:** Facilitates granular time window definitions and visualizes traffic frequency to spot volume spikes.
![](../../assets/Pasted%20image%2020261014221310.png)
![](../../assets/Pasted%20image%2020261014221346.png)

---

## 3. Kibana Query Language (KQL) Mechanics

KQL is a query language optimized for exploring Elasticsearch indices:

![](../../assets/Pasted%20image%2020261014225857.png)

* **Free Text & Wildcard Queries:** Matches strings across document boundaries. Using wildcards (`United*`) ensures partial string expansion:
![](../../assets/Pasted%20image%2020261014225918.png)
![](../../assets/Pasted%20image%2020261014225937.png)
![](../../assets/Pasted%20image%2020261014225957.png)

* **Boolean Operators (`AND`, `OR`, `NOT`):** Refines search logic to combine or exclude criteria:
  * `"United States" AND "Virginia"`
  ![](../../assets/Pasted%20image%2020261014230020.png)
  * `"United States" OR "England"`
  ![](../../assets/Pasted%20image%2020261014230039.png)
  * `"United States" AND NOT ("Florida")`
  ![](../../assets/Pasted%20image%2020261014230055.png)

* **Field-Based Queries:** Target specific extracted attributes (`Field: Value`):

```
  kql
  Source_ip : 238.163.231.224 AND UserName : Suleman
```

