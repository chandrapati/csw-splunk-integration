# CSW alerting → SIEM & notifiers — customer handoff checklist

**Purpose:** Give this page to your customer's **SecOps / SIEM / platform** team when wiring Cisco Secure Workload (CSW) alerts into Splunk and other notifiers.

**Integration summary:** CSW alert rules select a **Publisher**. Each publisher is a **notifier connector** running on the **Edge appliance** via the **TAN (Tetration Alert Notifier)** service. The **same alert** can be sent to multiple destinations — a SIEM (Splunk/Syslog), on-call paging (PagerDuty), ChatOps (Slack/Webex/Discord), and Email.

---

## Notifier destinations

| Notifier | Best for | What the team must provide | Transport |
|---|---|---|---|
| **Syslog → Splunk / any SIEM** | System of record | SIEM collector IP + port, protocol (UDP/TCP), target index | Syslog |
| **Email** | Human notification | SMTP server+port, (optional) SMTP creds, from + recipients | SMTP |
| **Slack** | SecOps channel | Incoming Webhook URL | HTTPS |
| **PagerDuty** | On-call paging | Events API integration (service) key | HTTPS |
| **Kinesis** | AWS streaming | IAM access/secret key (least-privilege), region, stream name | Kinesis |
| **Webex** | SecOps space | Incoming Webhook URL | HTTPS |
| **Discord** | SecOps channel | Webhook URL | HTTPS |

---

## What we need from the SecOps / SIEM team

### 1. Splunk (primary)
| Item | Detail |
|------|--------|
| **Cisco Security Cloud App** | Installed (Splunkbase App ID 7404), CIM 6.x present |
| **Heavy Forwarder** | Modular input runs on the HF (distributed Splunk) |
| **Index** | Target index (e.g. `cisco_csw`) created |
| **Port** | Chosen syslog port (UDP/TCP), unique, reachable from Edge appliance |

### 2. Connectivity (per notifier)
| Direction | Source | Destination | Port |
|-----------|--------|-------------|------|
| Outbound | CSW Edge appliance | Splunk HF / SIEM collector | Syslog UDP/TCP (e.g. 514) |
| Outbound | CSW Edge appliance | SMTP relay | 25 / 587 / 465 |
| Outbound | CSW Edge appliance | Slack / PagerDuty / Webex / Discord | 443 (HTTPS webhook) |
| Outbound | CSW Edge appliance | AWS Kinesis endpoint | 443 |

### 3. Facts to confirm
- [ ] Edge + Ingest appliance deployed and healthy (TAN service running)
- [ ] Splunk ≥ 9.1, CIM 6.x, Cisco Security Cloud App installed
- [ ] Firewall open Edge → each notifier destination
- [ ] Secrets provided securely (webhook URLs, PagerDuty key, SMTP password, AWS keys) — never in tickets/source
- [ ] Alert routing plan (which alert types → which notifier/severity)

---

## Recommended routing

| Alert type | Severity | Splunk | PagerDuty | Slack/Webex | Email |
|---|---|:---:|:---:|:---:|:---:|
| Forensics (MITRE ATT&CK) | CRITICAL | ✅ | ✅ | ✅ | — |
| Compliance → catch-all DENY | HIGH | ✅ | ✅ | ✅ | — |
| Sensors (agent down) | HIGH | ✅ | ✅ | ✅ | ✅ |
| Connectors / appliance health | HIGH | ✅ | — | ✅ | ✅ |
| Traffic (malicious IP) | MEDIUM | ✅ | — | ✅ | — |

---

## Limits

- Typically **one connector of each notifier type per Edge appliance** and **per tenant (root scope)**.
- All notifiers run on the **TAN** Docker service — monitor its health under **Virtual Appliances → Edge → Connectors**.

---

## Validation (joint test)

| # | Test | Owner | Pass |
|---|------|-------|------|
| 1 | Syslog **Test** button returns success | CSW admin | ☐ |
| 2 | Events land in Splunk: `index=cisco_csw \| head 20` | SIEM team | ☐ |
| 3 | PagerDuty test alert pages on-call | SecOps | ☐ |
| 4 | Slack/Webex test alert posts to channel | SecOps | ☐ |
| 5 | Email test alert delivered | SecOps | ☐ |
| 6 | TAN service healthy on Edge appliance | CSW admin | ☐ |

---

## Security & compliance talking points

- **Secrets hygiene** — webhook URLs, PagerDuty keys, SMTP passwords, and AWS keys are secrets; store in a secrets manager, use least-privilege (e.g. Kinesis `PutRecord` only).
- **Same alert, many destinations** — SIEM for the record, PagerDuty for paging, ChatOps for awareness, Email for digests.
- **Least exposure** — restrict Edge appliance egress to only the notifier destinations required.

---

## References

- Full guide: [CSW-Splunk-Integration-Guide.md](../CSW-Splunk-Integration-Guide.md) (see **Part 5** for all notifiers)
- CSW Connectors for Alert Notifications: https://www.cisco.com/c/en/us/td/docs/security/workload_security/secure_workload/user-guide/4_0/cisco-secure-workload-user-guide-on-prem-v40/configure-and-manage-connectors-for-secure-workload.html

---

*Generic template — no customer-specific names. Customize endpoints, indexes, and contacts locally before sending.*
