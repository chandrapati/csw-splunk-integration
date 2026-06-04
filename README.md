# Cisco Secure Workload → Splunk Integration Guide

A step-by-step integration guide for streaming Cisco Secure Workload (CSW) alerts and compliance events into Splunk using the **Cisco Security Cloud App for Splunk**.

[![Cisco Secure Workload](https://img.shields.io/badge/Cisco-Secure%20Workload-00205B?logo=cisco&logoColor=white)](https://www.cisco.com/go/secureworkload)
[![Splunk](https://img.shields.io/badge/Splunk-Enterprise%20%2F%20Cloud-65A637?logo=splunk&logoColor=white)](https://splunkbase.splunk.com/app/7404)
[![App Version](https://img.shields.io/badge/Cisco%20Security%20Cloud%20App-v3.6.5-007BC7)](https://splunkbase.splunk.com/app/7404)

> **⚠ Disclaimer:** This is a **community reference guide** prepared by Cisco Solutions Engineering — not an official Cisco product document. Always refer to the [official Cisco Secure Workload documentation](https://www.cisco.com/c/en/us/support/security/tetration/series.html) and the [Cisco Security Cloud App on Splunkbase](https://splunkbase.splunk.com/app/7404) for authoritative, up-to-date guidance.

---

## What This Covers

| Area | Detail |
|---|---|
| **Architecture** | Edge appliance → Syslog connector (TAN) → Splunk Heavy Forwarder → Cisco Security Cloud App |
| **Prerequisites** | Existing Edge + Ingest appliance; no new VMs required |
| **Alert types** | Compliance, Forensics (MITRE ATT&CK), Enforcement, Sensors, Traffic, Connectors |
| **Splunk app** | Cisco Security Cloud (Splunkbase App ID 7404, v3.6.5) |
| **Verified against** | CSW 4.0/4.1 on-prem and SaaS; Splunk Enterprise 9.x–10.x |

---

## Quick Start

### Prerequisites
- CSW Edge appliance deployed (VM on ESXi or KVM)
- Splunk Enterprise/Cloud ≥ 9.1, CIM 6.x installed
- Firewall rule open: Edge appliance IP → Splunk Heavy Forwarder, UDP/TCP on chosen port

### Steps (summary)

**On Cisco Secure Workload:**
1. `Manage → Workloads → Virtual Appliances` → select Edge appliance
2. Add **Syslog Connector** → set Protocol (UDP/TCP), Splunk HF IP, Port
3. `Alerts → Configuration` → add alert rules → set Publisher = Syslog Connector

**On Splunk:**
1. Install [Cisco Security Cloud](https://splunkbase.splunk.com/app/7404) from Splunkbase
2. Open app → Configure → **Cisco Secure Workload** → fill in Input Name, Protocol, Port, IP, Index
3. Verify: `index=cisco_csw | head 20`

See the [full step-by-step guide](CSW-Splunk-Integration-Guide.md) or [open the HTML version](CSW-Splunk-Integration-Guide.html) for detailed instructions with screenshots.

---

## Video Walkthrough

Cisco Senior SE Jorge Quintero demonstrates the full integration in **8 minutes**:

▶ **[Watch: Secure Workload and Splunk Integration](https://www.youtube.com/watch?v=CRnkH9imTZk)**

| Timestamp | Topic |
|---|---|
| 0:00 – 1:30 | Architecture overview |
| 1:30 – 3:00 | Syslog connector setup in CSW |
| 3:00 – 4:30 | Alert Config rules and publisher selection |
| 4:30 – 6:00 | Cisco Security Cloud App configuration in Splunk |
| 6:00 – 8:42 | CSW dashboard walkthrough and filtering |

---

## Architecture Diagram

![CSW and Splunk Integration Architecture](csw-splunk-architecture.png)

*Edge appliance VM hosts the Syslog connector container (alongside SNOW, ISE connectors). Control plane connects to CSW cluster; logs and alerts stream via syslog to Splunk.*

---

## Files in This Repo

| File | Description |
|---|---|
| [`README.md`](README.md) | This file — quick start and overview |
| [`CSW-Splunk-Integration-Guide.md`](CSW-Splunk-Integration-Guide.md) | Full step-by-step guide (Markdown source) |
| [`CSW-Splunk-Integration-Guide.html`](CSW-Splunk-Integration-Guide.html) | Styled HTML — open in browser for best experience (includes embedded YouTube player) |
| [`csw-splunk-architecture.png`](csw-splunk-architecture.png) | Architecture diagram from Cisco SE demo |
| [`build.sh`](build.sh) | Regenerate HTML/PDF from Markdown (requires pandoc + Chrome) |

---

## Alert Types — Quick Reference

| Alert Type | What It Detects | Requires Enforcement? |
|---|---|---|
| **Compliance → Enforcement Policy** | Policy violations / catch-all hits | ✅ Yes — workspace must be enforced |
| **Forensics** (MITRE ATT&CK) | Behavioral anomalies, lateral movement | ❌ No — works in Monitor mode |
| **Sensors** | Agent heartbeat loss / agent down | ❌ No |
| **Traffic** | Communication with known malicious IPs | ❌ No |
| **Connectors** | Connector/appliance health | ❌ No |
| **Enforcement** | Workload firewall off / policy state change | ❌ No |

> **Important:** Compliance alerts require the workspace to be in **Enforcement mode** and the catch-all set to **DENY**. For lateral movement detection before enforcement, use **Forensics** alerts instead.

---

## Related Cisco Secure Workload Resources

Other public repositories that walk the full CSW journey — from onboarding to compliance reporting:

| Repository | What It Covers | Best For |
|---|---|---|
| [**CSW-User-Education**](https://github.com/chandrapati/CSW-User-Education) | Intro guide, curated video library, and customer onboarding runbook | Anyone new to CSW — great first stop |
| [**CSW-Agent-Installation-Guide**](https://github.com/chandrapati/CSW-Agent-Installation-Guide) | Host agent install across Linux, Windows, cloud, containers, and agentless | Operations and deployment teams |
| [**CSW-Policy-Lifecycle**](https://github.com/chandrapati/CSW-Policy-Lifecycle) | Full policy lifecycle: ADM discovery → Monitor → Simulate → Enforce + day-2 ops | SE/SA and customer engineering |
| [**CSW-Compliance-Mapping**](https://github.com/chandrapati/CSW-Compliance-Mapping) | Customer-facing compliance reports for HIPAA, SOC 2, PCI DSS v4, NIST 800-53, ISO 27001, CISA ZTMM, FIPS 140 | CISO, GRC, and audit teams |
| [**CSW_POV_Template**](https://github.com/chandrapati/CSW_POV_Template) | Reusable POV engagement toolkit — clone for each new engagement | SEs running a CSW proof-of-value |
| [**csw_blast_radius_demo**](https://github.com/chandrapati/csw_blast_radius_demo) | Hands-on demo showing blast radius reduction via microsegmentation | Demo and lab environments |
| [**CSW-ServiceNow-Connector-Guide**](https://github.com/chandrapati/CSW-ServiceNow-Connector-Guide) | ServiceNow CMDB connector (Inventory Enrichment): setup, label usage, operations, and limits — validated against CSW 4.0 docs | SE/SA and platform/CMDB teams |

> **Suggested path for a new customer:**
> User-Education → Agent-Installation-Guide → Policy-Lifecycle → (Splunk integration — this repo) → Compliance-Mapping

---

## References

- [Cisco Security Cloud App on Splunkbase](https://splunkbase.splunk.com/app/7404)
- [CSW 4.0 Configure Alerts — official docs](https://www.cisco.com/c/en/us/td/docs/security/workload_security/secure_workload/user-guide/4_0/cisco-secure-workload-user-guide-on-prem-v40/configure-alerts.html)
- [Cisco Security Cloud App — CSW config guide](https://www.cisco.com/c/en/us/td/docs/security/cisco-secure-cloud-app/user-guide/cisco-security-cloud-user-guide/m_configure_cisco_products_in_cisco_security_cloud.html)
- [CSW 4.0 Connectors User Guide](https://www.cisco.com/c/en/us/td/docs/security/workload_security/secure_workload/user-guide/4_0/cisco-secure-workload-user-guide-on-prem-v40/configure-and-manage-connectors-for-secure-workload.html)

---

*Prepared by Cisco Solutions Engineering. Community contributions welcome — open a PR or issue.*
