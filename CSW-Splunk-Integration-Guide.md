# Cisco Secure Workload — Splunk Integration Guide
**Audience:** Enterprise security operations teams
**Prepared by:** Cisco Solutions Engineering  
**Version:** 1.0

---

## Video Walkthrough — Cisco SE Demo (8 min)

> **Watch this first.** Cisco Senior Solutions Engineer Jorge Quintero demonstrates the complete CSW → Splunk integration end-to-end in this 8-minute walkthrough recorded December 2025.
>
> **[▶ Watch on YouTube: Secure Workload and Splunk Integration](https://www.youtube.com/watch?v=CRnkH9imTZk)**

The video covers exactly what you will configure in this guide:

| Timestamp | Topic |
|---|---|
| 0:00 – 1:30 | Architecture overview — Edge appliance, Syslog connector, Splunk flow |
| 1:30 – 3:00 | Navigating to Virtual Appliances and the Syslog connector in CSW UI |
| 3:00 – 4:30 | Configuring Alert Config rules (compliance, forensics, sensors) and selecting the publisher |
| 4:30 – 6:00 | Installing and configuring the Cisco Security Cloud App in Splunk (input name, port, protocol, index) |
| 6:00 – 8:42 | Exploring the CSW dashboard — traffic, compliance, enforcement, forensics panels; time-range filtering |

---

## Overview

This guide walks through the end-to-end configuration required to stream Cisco Secure Workload (CSW) alerts and compliance events into Splunk using the **Cisco Security Cloud App for Splunk** (formerly "Cisco Security Cloud Control"). The integration uses a **Syslog Connector** running on your existing CSW Edge appliance.

> **Prerequisite confirmed:** Edge appliance and Ingest appliance are already deployed and healthy in the your environment. No new virtual machines are required for this integration.

### What You Get in Splunk

| Alert Category | What It Covers |
|---|---|
| **Traffic Alerts** | East-west flows hitting policy, catch-all hits |
| **Compliance Alerts** | Enforcement policy violations, annotated flow deviations |
| **Enforcement Alerts** | Policy enforcement state changes per workload |
| **Sensor / Connector Alerts** | Agent down, heartbeat loss, ingest appliance health |
| **Forensics Alerts** | Runtime behavior anomalies detected on workloads |
| **Uncategorized Alerts** | Events without a specific tag assigned |

---

## Architecture

![Cisco Secure Workload and Splunk Integration Architecture](csw-splunk-architecture.png)

*Figure 1 — The Edge appliance VM (ESXi/KVM) runs the Syslog connector container alongside SNOW and ISE connectors. The control plane connects back to the CSW cluster; logs and alerts stream via syslog (UDP/TCP) directly to Splunk.*

**Key components:**
- **TAN (Tetration Alert Notifier):** Docker service running on the Edge appliance that handles all alert-notifier connectors (Syslog, Email, Slack, PagerDuty, Kinesis)
- **Syslog Connector:** One instance per Edge appliance; listens on a configured port and streams CSW alerts via UDP or TCP
- **Cisco Security Cloud App:** Splunk add-on that ingests the syslog stream and renders purpose-built dashboards

> **Limits to be aware of:**
> - Maximum **1 Syslog connector per Edge appliance**
> - Maximum **1 Syslog connector per tenant (root scope)**
> - CSW supports up to 150 Syslog connectors cluster-wide

---

## Pre-Work Checklist

Before starting configuration, confirm the following with the your organization network / security teams:

- [ ] CSW Edge appliance is reachable from the Splunk Heavy Forwarder (or indexer) on the chosen syslog port
- [ ] Firewall rule opened: Edge appliance IP → Splunk Heavy Forwarder IP, on the chosen port (UDP/TCP, recommended **UDP 514** or a non-conflicting port above 1024)
- [ ] Splunk version is **9.1 or later** (Splunk Enterprise or Splunk Cloud)
- [ ] Splunk CIM (Common Information Model) version **6.x** is installed
- [ ] You have **admin rights** on both the CSW cluster and the Splunk instance
- [ ] Identify the Splunk **index** where CSW events will land (e.g., `cisco_csw`, or the default `main`). Create the index in Splunk before starting if using a custom name.

---

## Part 1 — Configure the Syslog Connector in CSW

### Step 1.1 — Navigate to Virtual Appliances

1. Log into the CSW cluster UI as an admin.
2. In the left navigation pane, go to **Manage → Workloads → Virtual Appliances**.
3. Locate your **Edge appliance** (already deployed in the your environment).
4. Click on the Edge appliance name to open its detail view.

### Step 1.2 — Add the Syslog Connector

1. Within the Edge appliance page, click **+ Add Connector** or navigate to the **Connectors** tab.
2. Select **Syslog** from the connector type list.
3. Configure the connector with the following parameters:

| Parameter | Value | Notes |
|---|---|---|
| **Protocol** | UDP *(recommended)* or TCP | Match this exactly on the Splunk input side |
| **Server Address** | IP address of the Splunk Heavy Forwarder | Must be reachable from the Edge appliance |
| **Port** | `514` (default) or a unique port above 1024 | Must not be in use by any other Splunk input |

4. Click **Save / Deploy**.

> **Test button:** Use the **Test** button on the connector page to send a test syslog alert to verify the Splunk server is receiving traffic before configuring alert rules.

### Step 1.3 — Configure Alert Types to Stream (Alerts Config)

This step determines *which* CSW alerts are forwarded to Splunk. Configure each alert type you want to export:

1. In the left navigation pane, go to **Defend → Alerts → Alert Config** (or **Platform → Alerts → Alerts Config** depending on your CSW version).
2. Click **+ Add Alert Rule**.
3. For each rule, configure:
   - **Alert Type** — see the table below
   - **Scope / Application** — the application workspace or root scope to monitor
   - **Condition** — the trigger condition (e.g., enforcement annotated flows, catch-all hits)
   - **Severity** — LOW / MEDIUM / HIGH / CRITICAL / IMMEDIATE ACTION
   - **Publisher** — select the **Syslog Connector** you just created (this routes the alert to Splunk)

#### Recommended Alert Types for your organization

| Alert Type | Purpose | Recommended Severity |
|---|---|---|
| **Compliance → Enforcement Policy** | Flows hitting catch-all or rejected by policy | HIGH |
| **Compliance → Allowed Policy** | Flows explicitly allowed — baseline verification | MEDIUM |
| **Forensics** | Runtime behavior anomalies on workloads | CRITICAL |
| **Sensors** | Agent down, agent stopped reporting | HIGH |
| **Connectors** | Connector or appliance health issues | HIGH |
| **Traffic** | Unusual east-west traffic patterns | MEDIUM |

> **Tip:** Start with **Compliance** and **Sensor** alert types to establish a baseline. Add Forensics alerts once agents are in enforcement mode — forensics events can be voluminous in simulation mode.

#### CSW Severity → Syslog Priority Mapping (default)

| CSW Severity | Syslog Priority |
|---|---|
| LOW | LOG_DEBUG |
| MEDIUM | LOG_WARNING |
| HIGH | LOG_ERR |
| CRITICAL | LOG_CRIT |
| IMMEDIATE ACTION | LOG_EMERG |

You can adjust this mapping under the Syslog Connector → **Severity Mapping** settings.

---

## Part 2 — Configure the Cisco Security Cloud App on Splunk

### Step 2.1 — Install the App from Splunkbase

> **Skip this step if the app is already installed.**

1. In Splunk, go to **Apps → Find More Apps**.
2. Search for **"Cisco Security Cloud"**.
3. Install **Cisco Security Cloud** (App ID: 7404), published by Cisco Systems, Inc.
   - Latest version: **3.6.5** (May 1, 2026)
   - Compatible with Splunk Enterprise / Splunk Cloud 9.1–10.4
4. Restart Splunk if prompted.

Direct link: [Cisco Security Cloud on Splunkbase](https://splunkbase.splunk.com/app/7404)

> **Important (Distributed Splunk):** In a distributed Splunk architecture, modular inputs **must** be configured and run on the **Heavy Forwarder (HF)**, not on Search Heads or Indexers.

### Step 2.2 — Add the Cisco Secure Workload Input

1. Open the **Cisco Security Cloud** app in Splunk.
2. Navigate to **Configuration → Setup** (or the **My Apps** section).
3. Click **Add New** or look for **Cisco Secure Workload** in the product list.
4. Click **Configure** (or **Edit Configuration** if a config already exists).

### Step 2.3 — Fill In the Configuration Fields

| Field | Value | Notes |
|---|---|---|
| **Input Name** | `org-csw-prod` *(or similar)* | A unique, meaningful name for this data source |
| **Input Type** | Syslog | Fixed for CSW |
| **Protocol** | UDP or TCP | **Must match** the protocol configured on the CSW Syslog Connector (Step 1.2) |
| **Port** | `514` or the port chosen in Step 1.2 | Must be unique; not used by another Splunk input |
| **IP Address** | IP of the CSW Edge appliance Syslog Connector | The source IP sending syslog to Splunk |
| **Source Type** | `cisco:csw` *(default)* | Leave as default unless customized |
| **Index** | `cisco_csw` *(or your chosen index)* | Must match the index created in pre-work |
| **Polling Interval** | `60` seconds (default) | How often the app queries CSW; 60s is recommended |

5. Click **Save** to apply the configuration.

### Step 2.4 — Verify the Syslog Input is Receiving Data

1. In Splunk, navigate to **Settings → Data Inputs → TCP** or **UDP** (depending on protocol).
2. Confirm a new input exists on the configured port, bound to the CSW connector IP.
3. Run a quick search to verify events are landing:

```spl
index=cisco_csw sourcetype=cisco:csw | head 20
```

If events appear, the pipeline is working. If not, see the Troubleshooting section below.

---

## Part 3 — Explore the CSW Dashboard in Splunk

Once data is flowing, navigate to:

**Apps → Cisco Security Cloud → App Analytics → Cisco Secure Workload Dashboard**

The dashboard includes the following panels:

| Panel | What It Shows |
|---|---|
| **Traffic Alerts** | East-west flows matching or violating policy |
| **Compliance Alerts** | Flows hitting catch-all, enforcement policy deviations |
| **Enforcement Alerts** | Policy state changes per workload scope |
| **Sensor & Connector Alerts** | Agent heartbeat loss, connector health, ingest appliance status |
| **Forensics Alerts** | Runtime behavior anomalies (process execution, file access) |
| **Uncategorized Alerts** | Events not tagged to a specific category |
| **Severity Summary** | Count of HIGH / CRITICAL alerts with timeline distribution |

### Filtering by Time Range

Use the Splunk time range picker in the top-right corner of the dashboard. Common views:
- **Last 24 hours** — daily SecOps review
- **Last 7 days** — weekly compliance posture
- **Custom range** — incident investigation

---

## Part 4 — Recommended Alert Rules for SecOps team

Configure the following alert rules in CSW (**Alerts Config**) and route them to the Syslog connector. These align to common enterprise microsegmentation requirements for microsegmentation compliance:

### Rule 1 — Catch-All Enforcement Hits (HIGH priority)
- **Type:** Compliance → Enforcement Policy
- **Condition:** Enforcement annotated flows → catch-all allow or deny
- **Scope:** Root scope (all workloads)
- **Severity:** HIGH
- **Value:** Alert when any flow hits the catch-all policy — indicates a gap in policy coverage

### Rule 2 — Agent Down (HIGH priority)
- **Type:** Sensors
- **Condition:** Agent stopped sending heartbeats
- **Scope:** Root scope
- **Severity:** HIGH
- **Value:** Immediate awareness when enforcement coverage drops on a workload

### Rule 3 — Forensics — Process Anomaly (CRITICAL)
- **Type:** Forensics
- **Scope:** Crown-jewel application workspaces (e.g., trading systems, data vaults)
- **Severity:** CRITICAL
- **Value:** Runtime behavior alerts — unexpected process spawning, unusual file access

### Rule 4a — Lateral Movement Detection via Forensics (CRITICAL) ✅ Works in Monitor mode

- **Type:** Forensics (no Alert Config modal required — auto-enabled once Forensics is turned on in agent config)
- **Detection mechanism:** MITRE ATT&CK lateral movement tactic rules (e.g., T1021 — Remote Services, T1076 — RDP, anomalous process execution, unexpected network connections)
- **Scope:** Any scope — works in Monitor, Simulate, or Enforce mode
- **Severity:** CRITICAL
- **Prerequisite:** Enable Forensics feature in **Manage → Software Agent Config → Forensics toggle ON**
- **Value:** Behavioral detection of post-compromise lateral movement; no explicit deny policies needed; fires before enforcement is enabled

> **This is the recommended approach during initial deployment** — it works immediately in Monitor mode and does not require enforcement to be active.

### Rule 4b — Intra-Scope Policy Violation Detection (HIGH) ⚠️ Requires enforcement to be active

- **Type:** Compliance → Enforcement Policy
- **Condition:** Flows hitting the **catch-all DENY** within a specific application scope (indicates traffic with no matching allow policy)
- **Scope:** Individual application workspace scope where enforcement is enabled
- **Severity:** HIGH
- **Prerequisites (all three required):**
  1. Workspace must be in **Enforcement mode** (not Monitor or Simulate)
  2. Explicit **DENY** policies must exist between same-scope workloads in the workspace
  3. Catch-all policy in the workspace must be set to **DENY**
- **Value:** Detects attempts to move laterally between workloads within the same tier after enforcement is active; catch-all hits surface traffic that has no explicit allow rule

> **Note on terminology:** CSW does not use "Intra-EPG" in alerts config — that is ACI terminology. CSW uses **scopes**. If ACI EPGs are mapped to CSW scopes via the ACI Connector, same-scope workloads correspond to same-EPG workloads, but the CSW alert is configured at the scope level, not EPG level.

---

## Troubleshooting

### No Events Appearing in Splunk

| Check | Action |
|---|---|
| Syslog connector test in CSW | Use **Test** on the connector page; confirm it returns success |
| Firewall between Edge appliance and Splunk | Run `tcpdump -i <iface> udp port 514` on the Splunk HF to verify packets arrive |
| Splunk input port conflict | Ensure no other input is bound to the same port (`Settings → Data Inputs`) |
| Protocol mismatch | Confirm UDP/TCP matches exactly on both CSW connector and Splunk input |
| Splunk index missing | Confirm the index exists: `Settings → Indexes` |
| Heavy Forwarder role | In distributed Splunk, ensure the modular input runs on HF, not Search Head |

### Connector Shows "Cannot connect to server, check config"

This alert appears in CSW when the connector cannot reach the Splunk syslog listener. Check:
1. Splunk Heavy Forwarder is up and listening on the configured port
2. No ACL/firewall blocking UDP/TCP between Edge appliance and Splunk HF
3. Edge appliance has outbound connectivity to the Splunk HF IP

### Alert Volume is Too High

If the alert volume overwhelms the Splunk index:
1. Narrow the **Scope** in Alert Config rules (use a specific application workspace instead of root scope)
2. Raise the **Severity threshold** — only stream HIGH and CRITICAL initially
3. Disable **Compliance → Allowed Policy** rules during high-volume periods (these can be very noisy in simulation mode)

### Connector Health Alert from CSW

CSW generates a **"Missing heartbeats"** alert (Severity: HIGH) if the Edge appliance or Syslog connector is not sending heartbeats. Check:
1. Edge appliance VM is powered on and running
2. TAN (Tetration Alert Notifier) Docker service is healthy — navigate to **Virtual Appliances → Edge → Connectors** and confirm status is **Running**

---

## Summary of Changes Made in Each System

### On Cisco Secure Workload
- [ ] Syslog Connector created on Edge appliance (Protocol, IP, Port configured)
- [ ] Alert Config rules defined: Compliance, Forensics, Sensors, Traffic (as applicable)
- [ ] Publisher set to Syslog Connector for each rule
- [ ] Test alert sent and confirmed successful

### On Splunk
- [ ] Cisco Security Cloud App installed (v3.6.5+)
- [ ] Cisco Secure Workload input configured (Name, Protocol, Port, IP, Index)
- [ ] Target index created (e.g., `cisco_csw`)
- [ ] Verified events arriving: `index=cisco_csw | head 20`
- [ ] CSW Dashboard accessible under App Analytics

---

## Reference Links

| Resource | URL |
|---|---|
| Cisco Security Cloud App for Splunk (Splunkbase) | https://splunkbase.splunk.com/app/7404 |
| Cisco Security Cloud App — CSW Config Guide | https://www.cisco.com/c/en/us/td/docs/security/cisco-secure-cloud-app/user-guide/cisco-security-cloud-user-guide/m_configure_cisco_products_in_cisco_security_cloud.html |
| CSW 4.0 Connectors User Guide | https://www.cisco.com/c/en/us/td/docs/security/workload_security/secure_workload/user-guide/4_0/cisco-secure-workload-user-guide-on-prem-v40/configure-and-manage-connectors-for-secure-workload.html |
| Demo video (Cisco SE walkthrough) | https://www.youtube.com/watch?v=CRnkH9imTZk |

---

*Document prepared by Cisco Solutions Engineering for the your organization microsegmentation engagement. Customer environment details are anonymized per Cisco policy.*
