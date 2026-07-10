# Cisco Secure Workload → Splunk & Alert Notifiers Integration Guide

![Visitors](https://visitor-badge.laobi.icu/badge?page_id=chandrapati.csw-splunk-integration&left_text=visitors)
[![GitHub stars](https://img.shields.io/github/stars/chandrapati/csw-splunk-integration?style=social)](https://github.com/chandrapati/csw-splunk-integration/stargazers)
[![GitHub forks](https://img.shields.io/github/forks/chandrapati/csw-splunk-integration?style=social)](https://github.com/chandrapati/csw-splunk-integration/network/members)
[![Last commit](https://img.shields.io/github/last-commit/chandrapati/csw-splunk-integration)](https://github.com/chandrapati/csw-splunk-integration/commits/main)

A step-by-step integration guide for streaming Cisco Secure Workload (CSW) alerts and compliance events into **Splunk** using the **Cisco Security Cloud App**, **plus a complete reference for every CSW alert notifier** — **Syslog** (any SIEM), **Email**, **Slack**, **PagerDuty**, **Kinesis**, **Webex**, and **Discord**.

[![Cisco Secure Workload](https://img.shields.io/badge/Cisco-Secure%20Workload-00205B?logo=cisco&logoColor=white)](https://www.cisco.com/go/secureworkload)
[![Splunk](https://img.shields.io/badge/Splunk-Enterprise%20%2F%20Cloud-65A637?logo=splunk&logoColor=white)](https://splunkbase.splunk.com/app/7404)
[![Notifiers](https://img.shields.io/badge/Notifiers-Syslog%20%C2%B7%20Email%20%C2%B7%20Slack%20%C2%B7%20PagerDuty%20%C2%B7%20Kinesis%20%C2%B7%20Webex-007BC7)](https://www.cisco.com/c/en/us/td/docs/security/workload_security/secure_workload/user-guide/4_0/cisco-secure-workload-user-guide-on-prem-v40/configure-and-manage-connectors-for-secure-workload.html)

> **⚠ Disclaimer:** This is a **community reference guide** prepared by Cisco Solutions Engineering — not an official Cisco product document. Always refer to the [official Cisco Secure Workload documentation](https://www.cisco.com/c/en/us/support/security/tetration/series.html) and the [Cisco Security Cloud App on Splunkbase](https://splunkbase.splunk.com/app/7404) for authoritative, up-to-date guidance.

---

## What This Covers

| Area | Detail |
|---|---|
| **Primary path** | Edge appliance → Syslog connector (TAN) → Splunk Heavy Forwarder → Cisco Security Cloud App |
| **All notifiers** | Syslog (any SIEM), Email, Slack, PagerDuty, Kinesis, Webex, Discord — same alerts, many destinations |
| **Prerequisites** | Existing Edge + Ingest appliance; no new VMs required |
| **Alert types** | Compliance, Forensics (MITRE ATT&CK), Enforcement, Sensors, Traffic, Connectors |
| **Splunk app** | Cisco Security Cloud (Splunkbase App ID 7404) |
| **Verified against** | CSW 4.0/4.1 on-prem and SaaS; Splunk Enterprise 9.x–10.x |

---

## Quick Start (Splunk / Syslog)

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

## All Alert Notifiers — Quick Reference

Every notifier runs on the **Edge appliance** via the **TAN (Tetration Alert Notifier)** service. Configure the notifier, then point alert rules at it as the **Publisher** — the same alerts can go to multiple destinations. Full details in the guide's **Part 5**.

| Notifier | Best for | Key fields | Transport |
|---|---|---|---|
| **Syslog** | Any SIEM (Splunk, QRadar, Sentinel, Elastic, Chronicle) | Protocol (UDP/TCP), IP, port | Syslog |
| **Email** | Human notification / ticket mailboxes | SMTP server+port, (optional) user/pass, from, recipients | SMTP |
| **Slack** | ChatOps channel | Incoming Webhook URL | HTTPS webhook |
| **PagerDuty** | On-call escalation | Integration (service) key | HTTPS Events API |
| **Kinesis** | AWS streaming / data lake | Access key, secret, region, stream | Kinesis Data Stream |
| **Webex** | ChatOps space | Incoming Webhook URL | HTTPS webhook |
| **Discord** | ChatOps channel | Webhook URL | HTTPS webhook |

> **Routing pattern:** SIEM (Splunk/Syslog) for the record · PagerDuty to page on-call · Slack/Webex for the SecOps channel · Email for health/digest.

---

## Video Walkthrough

> **Legend:** 🎬 video · 📘 guide · 📄 doc

Cisco Senior SE Jorge Quintero demonstrates the full Splunk integration in **8 minutes**:

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

*Edge appliance VM hosts the notifier connectors (Syslog/Email/Slack/PagerDuty/Kinesis/Webex/Discord) via the TAN service, alongside SNOW and ISE connectors. Control plane connects to the CSW cluster; alerts stream to Splunk (syslog) and/or any other notifier destination.*

---

## Files in This Repo

| File | Description |
|---|---|
| [`README.md`](README.md) | This file — quick start and overview |
| [`CSW-Splunk-Integration-Guide.md`](CSW-Splunk-Integration-Guide.md) | Full step-by-step guide (Markdown source) — includes **Part 5: all alert notifiers** |
| [`CSW-Splunk-Integration-Guide.html`](CSW-Splunk-Integration-Guide.html) | Styled HTML — open in browser (includes embedded YouTube player) |
| [`csw-splunk-architecture.png`](csw-splunk-architecture.png) | Architecture diagram from Cisco SE demo |
| [`build.sh`](build.sh) | Regenerate HTML/PDF from Markdown (requires pandoc + Chrome) |
| [`docs/CUSTOMER-HANDOFF.md`](docs/CUSTOMER-HANDOFF.md) | Checklist to hand to the customer's SecOps / SIEM team |
| [`docs/00-official-references.md`](docs/00-official-references.md) | Authoritative Cisco + Splunk + notifier doc links |

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

## Step-by-Step Guides

> **Legend:** 🎬 video · 📘 guide · 📄 doc

Hands-on integration and deployment guides — follow these top to bottom to build out a deployment:

| Guide | Description | Best for |
|-------|-------------|---------|
| [📘 Agent Installation](https://github.com/chandrapati/CSW-Agent-Installation-Guide) | Deploy CSW agents on Linux / Windows / cloud | Day-1 sensor deployment |
| [📘 Kubernetes](https://github.com/chandrapati/csw-kubernetes-integration) | K8s connector + DaemonSet: pod/service labels, flow visibility, iptables enforcement, CVE scanning | Container segmentation |
| [📘 OpenShift](https://github.com/chandrapati/csw-openshift-integration) | OpenShift connector + DaemonSet (privileged SCC): project/pod/service labels, flows, iptables enforcement | Red Hat OpenShift segmentation |
| [📘 Policy Lifecycle](https://github.com/chandrapati/CSW-Policy-Lifecycle) | Policy discovery → enforcement workflow | Policy management |
| [📘 vCenter](https://github.com/chandrapati/csw-vcenter-integration) | VMware vCenter VM identity + tag/category label import | Virtualization-driven policy |
| [📘 ACI](https://github.com/chandrapati/csw-aci-integration) | Cisco ACI endpoint/label ingestion, VRF→scope mapping, ESG enforcement | ACI fabric segmentation |
| [📘 DNS](https://github.com/chandrapati/csw-dns-integration) | AXFR zone-transfer hostname labels (`dns_name`) for scopes/policy | Hostname-driven policy |
| [📘 ISE / pxGrid](https://github.com/chandrapati/csw-ise-integration) | ISE/pxGrid: user-identity–aware microsegmentation | Identity & Zero Trust |
| [📘 AnyConnect NVM](https://github.com/chandrapati/csw-anyconnect-nvm) | Endpoint process flows + user identity via NVM | Endpoint telemetry |
| [📘 ServiceNow CMDB](https://github.com/chandrapati/csw-servicenow-integration) | ServiceNow CMDB label enrichment for workload scopes | CMDB-driven policy |
| [📘 Infoblox](https://github.com/chandrapati/csw-infoblox-integration) | Infoblox IPAM/DNS extensible-attribute label enrichment | IPAM/DNS-driven policy |
| [📘 F5 BIG-IP](https://github.com/chandrapati/csw-f5-integration) | F5 virtual-server labels, policy enforcement, IPFIX flow visibility | Load balancer segmentation |
| [📘 NetScaler ADC](https://github.com/chandrapati/csw-netscaler-integration) | NetScaler LB virtual-server labels, ACL enforcement + AppFlow/IPFIX flow visibility | Load balancer segmentation |
| [📘 AWS Connector](https://github.com/chandrapati/csw-aws-connector) | EC2 tag ingestion + VPC flow logs + Security Group enforcement | AWS workloads |
| [📘 Azure Connector](https://github.com/chandrapati/csw-azure-connector) | Azure VM tag ingestion + VNet flow logs + NSG enforcement | Azure workloads |
| [📘 GCP Connector](https://github.com/chandrapati/csw-gcp-connector) | GCE label ingestion + VPC flow logs + firewall enforcement | GCP workloads |
| [📘 NetFlow](https://github.com/chandrapati/csw-netflow-integration) | NetFlow v9/IPFIX agentless flow ingestion from switches | Network fabric visibility |
| [📘 ERSPAN](https://github.com/chandrapati/csw-erspan-integration) | Agentless packet mirroring for legacy / OT / IoT devices | Deep agentless visibility |
| [📘 Secure Firewall](https://github.com/chandrapati/CSW-Secure-Firewall-Integration-Guide) | NSEL flow ingestion + FMC policy enforcement | Firewall visibility & enforcement |
| [📘 Secure Connector](https://github.com/chandrapati/csw-secure-connector) | SaaS reverse-tunnel proxy to private orchestrator APIs (no inbound holes) | SaaS → on-prem reachability |
| [📘 Splunk & Notifiers](https://github.com/chandrapati/csw-splunk-integration) | CSW alerts → Splunk SIEM + all alert notifiers (Email/Slack/PagerDuty/Kinesis/Webex/Discord) | SecOps / SIEM teams |

## Resources

> **Legend:** 🎬 video · 📘 guide · 📄 doc

Learning paths, reference material, and day-2 tooling:

| Resource | Description | Best for |
|----------|-------------|---------|
| [📘 User Education](https://github.com/chandrapati/CSW-User-Education) | Onboarding guides, concept explainers, and curated video library | New CSW users |
| [📘 Compliance Mapping](https://github.com/chandrapati/CSW-Compliance-Mapping) | Map CSW controls to NIST, PCI-DSS, HIPAA, CIS | Compliance & audit |
| [📘 Tenant Insights](https://github.com/chandrapati/CSW-Tenant-Insights) | Tenant-level reporting and analytics | Visibility metrics |
| [📘 Operations Toolkit](https://github.com/chandrapati/CSW-Operations-Toolkit) | Day-2 ops scripts: health checks, reporting, policy analysis | Ongoing operations |
| [📄 Supported OS & Compatibility Matrix](https://www.cisco.com/c/m/en_us/products/security/secure-workload-compatibility-matrix.html) | Cisco's authoritative list of supported agent operating systems, external systems, and connector requirements | Platform planning & prerequisites |

> **Suggested customer journey:**
> User Education → Agent Installation → Policy Lifecycle → ISE/pxGrid → ServiceNow CMDB → Infoblox → F5 BIG-IP → NetScaler ADC → Splunk & Notifiers → Compliance Mapping → Operations Toolkit
