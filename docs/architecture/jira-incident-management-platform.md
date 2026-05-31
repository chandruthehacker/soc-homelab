# SOC Jira Incident Management Platform Design

This document is the canonical Jira case-management reference for the SOC homelab. It turns the incident lifecycle into a structured, automation-friendly schema that works with Jira Service Management or a company-managed Jira Software project.

## 1. Operating Model

### Project setup

| Attribute | Recommended value |
|---|---|
| Project name | SOC Incident Management |
| Project key | SOC |
| Project type | Jira Service Management preferred, Jira Software acceptable |
| Parent issue type | Security Incident |
| Child issue types | Investigation Task, Containment Action, Recovery Action, IOC Record |

### Design principles

- Keep the field count disciplined; 42 total fields is the upper bound for the full schema.
- Use dropdowns or checkboxes wherever possible.
- Auto-populate anything Shuffle can derive from Splunk, VirusTotal, AbuseIPDB, or asset lookups.
- Use comments as the investigation log.
- Use the description field as the case file and IOC table, not as a dumping ground for custom text fields.
- Use linked issues for sub-work, not one field per action.
- Use attachments for evidence and exports.

### Native vs custom field strategy

| Native Jira fields | Custom fields |
|---|---|
| Summary, Description, Priority, Assignee, Reporter, Labels, Components | Incident ID, Alert Source, Alert Severity, Incident Category, IOC fields, MITRE fields, response fields, quality fields |

## 2. Field Schema

### Group 1. Incident Intake

| # | Field | Type | Native | Required | Default / source | Notes |
|---|---|---|---|---|---|---|
| 1 | Summary | Single-line text | Native | Required | Populated by Shuffle | Format: [SEVERITY] Short Description - Affected IP/Host - Hostname - Date |
| 2 | Description | Rich text | Native | Required | Shuffle template | Stores alert overview, asset context, IOC table, raw JSON, enrichment summary, investigation notes, and timeline |
| 3 | Incident ID | Single-line text | Custom | Required | INC-YYYYMMDD-XXXX | Human-readable incident identifier for reporting and cross-platform correlation |
| 4 | Alert Source | Single-select dropdown | Custom | Required | Shuffle | Values: Splunk SIEM, Velociraptor, Manual - Analyst, Manual - Threat Hunt, External Report, Email Report, Shuffle SOAR |
| 5 | Alert Severity | Single-select dropdown | Custom | Required | Shuffle from Splunk | Values: Critical, High, Medium, Low, Informational |
| 6 | Priority | Single-select dropdown | Native | Required | Initial mapping from severity | Values: P1 - Critical, P2 - High, P3 - Medium, P4 - Low, P5 - Informational |
| 7 | Incident Category | Single-select dropdown | Custom | Required | Analyst | Values: Malware, Command and Control, Credential Access, Unauthorized Access, Phishing, Insider Threat, Denial of Service, Reconnaissance, Lateral Movement, Data Exfiltration, Policy Violation, False Positive |
| 8 | Incident Sub-Category | Single-select dropdown | Custom | Optional at intake | Analyst | Contextual sub-category for the selected category; can be a flat list in a homelab |

### Group 2. Alert Metadata

| # | Field | Type | Native | Required | Default / source | Notes |
|---|---|---|---|---|---|---|
| 9 | Alert Timestamp | Date/time | Custom | Required | Splunk alert time | Used for MTTD and timeline reconstruction |
| 10 | Detection Rule Name | Single-line text | Custom | Required for automated alerts | Splunk saved search name | Should match the Splunk detection rule or use Manual Detection for manual incidents |
| 11 | Source Alert Link | URL | Custom | Optional | Constructed by Shuffle | Deep link back to Splunk search results |

### Group 3. Asset Context

| # | Field | Type | Native | Required | Default / source | Notes |
|---|---|---|---|---|---|---|
| 12 | Affected Hostname | Single-line text | Custom | Required | Shuffle from Splunk | NetBIOS name or FQDN |
| 13 | Affected IP Address | Single-line text | Custom | Required | Shuffle from Splunk | IPv4 or IPv6 |
| 14 | Affected Username | Single-line text | Custom | Optional at intake | Shuffle when available | DOMAIN\\username or username@domain |
| 15 | Asset Criticality | Single-select dropdown | Custom | Required during triage | Asset lookup or analyst | Values: Critical, High, Medium, Low |

### Group 4. Threat Intelligence

| # | Field | Type | Native | Required | Default / source | Notes |
|---|---|---|---|---|---|---|
| 16 | Primary IOC Type | Single-select dropdown | Custom | Required during investigation | Analyst or Shuffle suggestion | Values: IPv4 Address, IPv6 Address, Domain, URL, File Hash - MD5, File Hash - SHA1, File Hash - SHA256, File Name, File Path, Email Address, Email Subject, Registry Key, Process Name, Named Pipe, Service Name, User Agent, JA3/JA3S Hash, None / Behavioral |
| 17 | Primary IOC Value | Single-line text | Custom | Required during investigation unless no IOC exists | Shuffle | Actual indicator value |
| 18 | VirusTotal Score | Text | Custom | Optional | Shuffle + VirusTotal | Format: X/Y, Not Checked, N/A, or Clean |
| 19 | AbuseIPDB Confidence Score | Number | Custom | Optional | Shuffle + AbuseIPDB | Range -1 to 100; use -1 for not checked |

### Group 5. MITRE ATT&CK

| # | Field | Type | Native | Required | Default / source | Notes |
|---|---|---|---|---|---|---|
| 20 | MITRE ATT&CK Tactic | Multi-select | Custom | Required during investigation | Analyst or rule tag mapping | Values: Reconnaissance, Resource Development, Initial Access, Execution, Persistence, Privilege Escalation, Defense Evasion, Credential Access, Discovery, Lateral Movement, Collection, Command and Control, Exfiltration, Impact |
| 21 | MITRE ATT&CK Technique ID | Single-line text | Custom | Required during investigation | Analyst or Shuffle | Format: TXXXX or TXXXX.XXX |

### Group 6. Investigation and Response

| # | Field | Type | Native | Required | Default / source | Notes |
|---|---|---|---|---|---|---|
| 22 | Investigation Findings | Rich text | Custom | Required before containment | Analyst | Formal assessment, scope, evidence, and recommended actions |
| 23 | Alert Disposition | Single-select dropdown | Custom | Required before exiting investigation | Analyst | Values: True Positive, Benign True Positive, False Positive, False Positive - Needs Tuning, Inconclusive |
| 24 | Containment Status | Single-select dropdown | Custom | Required during containment | Analyst or Shuffle | Values: Not Contained, Partially Contained, Fully Contained, Containment Failed, Not Applicable |
| 25 | Containment Actions | Multi-select checkboxes | Custom | Required when containment is applicable | Analyst or Shuffle | Host Isolated (Network), Account Disabled, Account Password Reset, Firewall Rule Added, DNS Sinkhole, Process Terminated, File Quarantined, Email Quarantined, No Action Taken |
| 26 | Eradication Actions | Multi-select checkboxes | Custom | Required during eradication | Analyst | Malware Removed, Persistence Mechanism Removed, Registry Entries Cleaned, Scheduled Tasks Removed, Compromised Credentials Rotated, System Rebuilt / Reimaged, Patches Applied, Vulnerability Remediated, Backdoor Account Removed, No Eradication Needed |
| 27 | Recovery Actions | Multi-select checkboxes | Custom | Required during recovery | Analyst | System Restored to Network, Backup Restored, Service Restarted, User Access Restored, Monitoring Enhanced, Detection Rule Created / Updated, No Recovery Needed |

### Group 7. Lessons Learned and Metrics

| # | Field | Type | Native | Required | Default / source | Notes |
|---|---|---|---|---|---|---|
| 28 | Root Cause | Single-select dropdown | Custom | Required before closing true positives | Analyst | Values: Phishing - User Clicked Link, Phishing - User Opened Attachment, Phishing - Credentials Entered, Vulnerable Software / Exploitation, Weak / Compromised Credentials, Misconfiguration, Insider Threat - Malicious, Insider Threat - Negligence, Supply Chain Compromise, Physical Access, Unknown, Not Applicable |
| 29 | Lessons Learned | Rich text | Custom | Required for P1/P2 incidents | Analyst | Post-incident review and action items |
| 30 | Detection Quality Rating | Single-select dropdown | Custom | Required before closing | Analyst | Values: Accurate - No Changes Needed, Accurate - Severity Adjustment Needed, Accurate - Enrichment Needed, False Positive - Needs Exclusion, False Positive - Needs Redesign, Missed Context - Additional Rule Needed, Duplicate Detection |
| 31 | Time to Detect (MTTD) | Number | Custom | Optional | Calculated | Minutes from malicious activity to alert creation |
| 32 | Time to Respond (MTTR) | Number | Custom | Optional | Calculated | Minutes from creation to first containment action |
| 33 | Time to Close (MTTC) | Number | Custom | Optional | Calculated | Minutes from creation to closure |

### Group 8. Compliance and Reporting

| # | Field | Type | Native | Required | Default / source | Notes |
|---|---|---|---|---|---|---|
| 34 | Data Breach Involved | Single-select dropdown | Custom | Required before closing true positives | Analyst | Values: Yes - Confirmed, Yes - Suspected, No, Under Investigation |
| 35 | Escalated | Single-select dropdown | Custom | Optional | Analyst | Values: No, Yes - Tier 2, Yes - Tier 3 / DFIR, Yes - Management, Yes - External (Legal/Law Enforcement) |

### Group 9. Threat Hunting and DFIR

| # | Field | Type | Native | Required | Default / source | Notes |
|---|---|---|---|---|---|---|
| 36 | Velociraptor Collection ID | Single-line text | Custom | Optional | Shuffle or analyst | Links the incident to a specific forensic collection |
| 37 | Threat Hunt Hypothesis | Multi-line text | Custom | Required when the alert source is Manual - Threat Hunt | Analyst | Structured hunt hypothesis and search plan |

### Group 10. Detection Engineering Feedback

| # | Field | Type | Native | Required | Default / source | Notes |
|---|---|---|---|---|---|---|
| 38 | Detection Engineering Action Required | Single-select dropdown | Custom | Required before closing | Analyst | Values: No Action Required, Create Exclusion, Tune Threshold, Rewrite Rule, Create New Rule, Add Enrichment, Deprecate Rule |

### Group 11. Native Jira Support Fields

| # | Field | Type | Native | Required | Default / source | Notes |
|---|---|---|---|---|---|---|
| 39 | Labels | Labels | Native | Optional | Analyst or automation | Suggested patterns: campaign:*, exercise:*, related:*, tool:*, data-source:*, regulation:* |
| 40 | Assignee | User picker | Native | Required during triage | Analyst claims ticket | Drives accountability and workload dashboards |
| 41 | Reporter | User picker | Native | Required | Shuffle service account or reporting analyst | Source of the incident report |
| 42 | Components | Component picker | Native | Optional | Analyst | Recommended values: Endpoint Security, Network Security, Email Security, Identity & Access, Detection Engineering, Threat Hunting, DFIR |

## 3. Description Template

Use the Jira Description field as the case file. Shuffle should pre-fill the template below and append enrichment data as it arrives.

```text
h2. Alert Overview
||Field||Value||
|Alert Source|Splunk - Rule: "..."|
|Alert Timestamp|2025-01-15T14:23:07Z|
|Splunk Search ID|1705312987.42|
|Splunk Alert Link|[View in Splunk|https://splunk.lab.local:8000/app/search/...]|

h2. Affected Asset
||Field||Value||
|Source IP|10.0.1.45|
|Hostname|WIN10-VICTIM|
|Operating System|Windows 10 Pro 22H2|
|Asset Criticality|High - User Workstation|
|Last Logged-on User|jsmith|

h2. Indicators of Compromise
||Type||Value||Enrichment||
|IPv4|198.51.100.23|AbuseIPDB: 95% confidence malicious|
|Domain|evil.example.com|VirusTotal: 14/90 vendors flagged malicious|
|SHA256|e3b0c442...|VirusTotal: Cobalt Strike, 58/72 detections|
|Named Pipe|\\.\\pipe\\msagent_89|Known Cobalt Strike default pipe name|

h2. Raw Alert Data
{code:json}
{
  "source": "XmlWinEventLog:Microsoft-Windows-Sysmon/Operational",
  "EventCode": "17",
  "PipeName": "\\msagent_89",
  "Image": "C:\\Windows\\Temp\\beacon.exe",
  "User": "VICTIM\\jsmith",
  "UtcTime": "2025-01-15 14:23:07.123"
}
{code}

h2. Enrichment Summary
* VirusTotal: ...
* AbuseIPDB: ...
* Velociraptor: ...

h2. Investigation Notes
_Analyst to populate during triage and investigation._

h2. Timeline
||Timestamp||Event||Source||
|2025-01-15T14:23:07Z|Named pipe created by beacon.exe|Sysmon EventID 17|
|2025-01-15T14:23:08Z|Network connection to 198.51.100.23:443|Sysmon EventID 3|
|2025-01-15T14:23:15Z|Splunk alert fired|Splunk|
|2025-01-15T14:23:18Z|Jira ticket created by Shuffle|Shuffle SOAR|
```

## 4. Workflow Model

### Status flow

```text
NEW ALERT -> TRIAGE -> INVESTIGATION -> CONTAINMENT -> ERADICATION -> RECOVERY -> VALIDATION -> CLOSED
                                         \-> REOPENED
```

### Status rules

| Status | Purpose | Entry criteria | Exit criteria |
|---|---|---|---|
| New Alert | Unreviewed alert | Ticket created by Shuffle or manually | Analyst claims the ticket |
| Triage | Initial assessment | Analyst assigns themselves | Priority confirmed, category set, asset criticality set, initial MITRE tactic set |
| Investigation | Deep analysis | Ticket warrants investigation | Findings written, disposition set, technique ID set, containment need decided |
| Containment | Stop the bleeding | True positive and containment needed | Containment status is fully contained |
| Eradication | Remove the threat | Containment complete | Eradication actions recorded |
| Recovery | Restore normal operations | Eradication complete | Recovery actions recorded |
| Validation | Post-incident review | Recovery complete or false positive | Closure-required fields completed |
| Closed | Final state | Validation complete | Reopen only if new evidence appears |
| On Hold | Waiting on external input | Analyst cannot proceed | External input received |
| Reopened | Closed incident reopened | New evidence or recurrence | Returns to investigation |

### Entry and exit requirements by stage

| Stage | Required fields or actions |
|---|---|
| New Alert | Summary, Description, Alert Source, Alert Severity, Priority, Alert Timestamp, Detection Rule Name, Affected Hostname, Affected IP Address |
| Triage | Priority confirmed, Incident Category, Asset Criticality, initial MITRE ATT&CK Tactic |
| Investigation | Investigation Findings, Alert Disposition, MITRE ATT&CK Technique ID, Primary IOC Type, Primary IOC Value |
| Containment | Containment Status, Containment Actions |
| Eradication | Eradication Actions |
| Recovery | Recovery Actions |
| Validation | Root Cause, Detection Quality Rating, Detection Engineering Action Required, Data Breach Involved, Lessons Learned for P1/P2 |

## 5. Shuffle Automation Mapping

### Auto-populate logic

| Field | Shuffle action | Source |
|---|---|---|
| Summary | Format template | Splunk payload |
| Description | Populate the full case template | Splunk payload plus enrichment responses |
| Alert Source | Map webhook source | Webhook metadata |
| Alert Severity | Map from severity | Splunk payload |
| Priority | Initial mapping from severity | Shuffle logic |
| Alert Timestamp | Extract trigger time | Splunk payload |
| Detection Rule Name | Extract search name | Splunk payload |
| Source Alert Link | Build URL from SID | Splunk payload |
| Affected Hostname | Extract field | Splunk payload |
| Affected IP Address | Extract field | Splunk payload |
| Affected Username | Extract field when present | Splunk payload |
| Primary IOC Type | Infer from populated IOC field | Shuffle logic |
| Primary IOC Value | Extract indicator value | Splunk payload |
| VirusTotal Score | Call VirusTotal API | VirusTotal API |
| AbuseIPDB Confidence Score | Call AbuseIPDB API | AbuseIPDB API |
| Incident ID | Generate daily sequence | Shuffle logic |

### Recommended workflow

1. Trigger on a Splunk webhook.
2. Parse the alert payload.
3. Enrich in parallel with VirusTotal, AbuseIPDB, and asset context lookup.
4. Build Summary and Description.
5. Map severity to initial Jira Priority.
6. Generate Incident ID.
7. Create the Jira issue.
8. Notify Slack or email for P1/P2.
9. Optionally trigger Velociraptor triage collection.

### Jira automation rules

| Rule | Trigger | Action |
|---|---|---|
| SLA breach notification | Scheduled every 15 minutes | Notify on open P1 incidents approaching breach |
| Auto-calculate MTTC | Issue transitioned to Closed | Set Time to Close from creation timestamp |
| Create detection engineering task | Detection Engineering Action Required changes | Create linked follow-up issue |
| Notify on P1/P2 creation | Issue created | Send alert to Slack or email |
| Auto-transition to Triage | Assignee changed from unassigned | Move issue from New Alert to Triage |

## 6. Dashboard Design

### SOC operations dashboard

| Row | Gadget | Query intent |
|---|---|---|
| 1 | Open incidents, critical/high open, MTTD, MTTR | Current load and response speed |
| 2 | Incidents by status and priority | Bottlenecks and queue health |
| 3 | Priority distribution and created-vs-resolved trend | Severity mix and incident volume trend |
| 4 | Alert disposition, top alert sources, detection quality ratings | True positive vs false positive quality |
| 5 | Incident categories, MITRE tactic distribution, top IOC types | Threat landscape and coverage |
| 6 | Top affected hostnames, analyst workload | Repeat victims and staffing balance |
| 7 | Overdue incidents, oldest open incidents | SLA and aging |

### Homelab dashboard minimum

- Open incidents by priority.
- Alert disposition pie chart.
- Incidents by status.
- MITRE ATT&CK tactic distribution.
- Recent incidents.
- Detection quality ratings.

## 7. Schema Tiers

| Tier | Scope | Notes |
|---|---|---|
| Minimum homelab schema | 20 fields | Enough to demonstrate a functional SOC workflow |
| Professional portfolio schema | 32 fields | Adds incident numbering, timestamps, alert link, response metrics, and learning loops |
| Full enterprise schema | 42 fields | Includes compliance, hunting, and DFIR integrations |

## 8. Example Ticket Patterns

Use the following incident profiles when creating screenshots, case studies, and dashboard samples.

| Example | Core pattern |
|---|---|
| Cobalt Strike execution | Malware, C2, containment, eradication, recovery, detection tuning |
| C2 beaconing | Command and control linked to the primary compromise |
| LSASS dump | Credential access with elevated escalation and breach consideration |
| Timestomping | Defense evasion on top of an existing compromise |
| Encoded PowerShell | Can be a benign true positive if the activity is authorized |
| SSH brute force | Credential access with external source reputation enrichment |
| Phishing report | User-reported alert that may or may not become a true incident |

## 9. Practical Recommendations

- Use the full workflow even in the homelab; the lifecycle is what makes the portfolio credible.
- Keep the custom field set disciplined and resist adding redundant text fields.
- Capture screenshots of Splunk, Jira, VirusTotal, and Velociraptor evidence for case studies.
- Treat Detection Quality Rating and Detection Engineering Action Required as the core feedback loop.
- Use Jira comments for the live investigation log and description for the structured case file.
