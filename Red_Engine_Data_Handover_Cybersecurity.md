# Red Engine Data Handover & Governance Document
**Prepared by:** Cybersecurity Team (Blue Team / Governance)  
**Target Team:** AI & Data Analysis (Red Engine Squad)  
**Project:** ImpersonaGuard — Week 2 Deliverable  
**Status:** Approved & Completed  

---

## 1. Executive Summary & Purpose
This document serves as the formal data handover manifest and specification for the **ImpersonaGuard Red Engine (Employee Risk Profiling Module)**. 

The Cybersecurity team has reviewed the data collection requirements and prepared this response to establish clear definitions, dataset availability, privacy boundaries, and governance controls. All features and logs provided under this agreement adhere to the principle of **Data Minimization**, strictly utilizing **Pseudonymous Employee Identifiers** to protect user privacy while enabling effective risk scoring.

---

## 2. Data Availability Matrix & Data Dictionary

Below is the exhaustive mapping of all requested data categories, their field-level specifications, native formats, and availability status.

### 2.1 Employee Context & Access
| Field Name | Type / Format | Availability Status | Description & Governance Notes |
| :--- | :--- | :--- | :--- |
| `employee_id_hash` | String (SHA-256) | **Available** | Stable pseudonymous join key across all sources. Direct identifiers (names/emails) are excluded. |
| `department` | String | **Available** | Employee department context (e.g., Finance, Engineering, HR). |
| `job_role_seniority` | String | **Available** | Role hierarchy level (e.g., Junior, Manager, Executive). |
| `role_criticality_tier` | Integer (1–4) | **Available** | Impact mapping tier (Tier 1: High Sensitivity/Executive, Tier 4: Low Sensitivity). |
| `system_privileges` | Array[String] | **Partially Available** | High-level privilege flags (`IS_ADMIN`, `ACCESS_FINANCE_DB`). Detailed ACLs excluded. |
| `data_classification_access` | Array[String] | **Available** | Highest data sensitivity tier authorized (`RESTRICTED`, `CONFIDENTIAL`, `PUBLIC`). |

### 2.2 Authentication & Identity Logs
| Field Name | Type / Format | Availability Status | Description & Governance Notes |
| :--- | :--- | :--- | :--- |
| `event_timestamp_utc` | ISO 8601 | **Available** | Event occurrence timestamp in UTC (`YYYY-MM-DDTHH:MM:SSZ`). |
| `auth_outcome` | Enum | **Available** | Outcome values: `SUCCESS`, `FAILED_BAD_PASSWORD`, `FAILED_MFA`, `ACCOUNT_LOCKED`. |
| `source_network_tag` | Enum | **Available** | Contextual tag: `CORPORATE_RANGE`, `VPN_RANGE`, `UNKNOWN_PUBLIC_IP`. Raw IPs masked. |
| `mfa_method_used` | String | **Available** | Authenticator type used (`TOTP`, `PUSH_NOTIFICATION`, `SMS`, `NONE`). |
| `is_service_account` | Boolean | **Available** | Flag to exclude automated non-human accounts from behavioral profiling. |

### 2.3 Remote Access & Network (VPN)
| Field Name | Type / Format | Availability Status | Description & Governance Notes |
| :--- | :--- | :--- | :--- |
| `vpn_session_id` | String | **Available** | Pseudonymous session tracking key. |
| `vpn_connect_timestamp` | ISO 8601 | **Available** | Session connection start time in UTC. |
| `vpn_duration_seconds` | Integer | **Available** | Total session duration in seconds. |
| `vpn_country_code` | String (ISO 3166-1) | **Partially Available** | Two-letter country code derived from Geo-IP. Precise GPS/City data excluded. |

### 2.4 Email & Simulation Security Context
| Field Name | Type / Format | Availability Status | Description & Governance Notes |
| :--- | :--- | :--- | :--- |
| `simulation_domain` | String | **Available** | Authorized internal lookalike domain used for authorized tests (e.g., `training-auth.local`). |
| `simulation_tracking_header` | String | **Available** | Headers attached to test emails (`X-ImpersonaGuard-Simulation: True`). |
| `email_action_event` | Enum | **Available** | User response: `REPORTED_PHISH`, `CLICKED_LINK`, `SUBMITTED_CREDS`, `IGNORED`. |
| `email_header_auth_verdict` | JSON | **Available** | Protocol check outcome: `{"spf": "PASS", "dkim": "PASS", "dmarc": "FAIL"}`. |
| `raw_email_body` | Text | **Unavailable** | **Excluded by Policy.** Email bodies and attachments are not collected or shared. |

### 2.5 Security Alerts, Incidents & Labels
| Field Name | Type / Format | Availability Status | Description & Governance Notes |
| :--- | :--- | :--- | :--- |
| `alert_id` | String | **Available** | Unique tracking key for security alerts. |
| `analyst_disposition` | Enum | **Available** | Verified ground truth: `CONFIRMED_BENIGN`, `FALSE_POSITIVE`, `CONFIRMED_INCIDENT`. |
| `incident_severity` | Enum | **Available** | Impact rating: `LOW`, `MEDIUM`, `HIGH`, `CRITICAL`. |

---

## 3. Explicit Responses to Handover Questions (Section 5 Compliance)

1. **Source System Coverage & Exclusions:**
   * Included: Core Auth Logs, VPN Access Logs, SEG (Email Gateway) Event Logs, SOC Incident Logs.
   * Excluded: Raw network packet captures (PCAP), raw file/email contents, and personal mobile device logs due to strict corporate privacy policies.

2. **Historical Coverage Period:**
   * Logs cover a rolling **90-day historical window**. Historical schema changes or gaps are documented in the log metadata manifests.

3. **Row Granularity:**
   * Auth/VPN/Email files represent **single discrete events**. Incident files represent **aggregated case summaries**.

4. **Authoritative Employee Join Key:**
   * `employee_id_hash` is the universal, stable join key across all sources generated via HMAC-SHA256 with a strictly guarded salt key.

5. **Handling Service & Contractor Accounts:**
   * Service accounts are flagged with `is_service_account: true` and must be segmented or filtered out during employee behavioral model training.

6. **Timestamps & Timezones:**
   * All timestamps are standardized to **ISO 8601 UTC**. Ingestion delays are logged under `ingestion_timestamp_utc` where latency exceeds 60 seconds.

7. **Simulation vs. Real Incident Separation:**
   * All authorized spear-phishing campaigns carry the `X-ImpersonaGuard-Simulation` header and are logged in a dedicated table. They are explicitly isolated from real-world threat incident labels.

8. **Privacy Boundaries & Exclusions:**
   * Raw PII (names, personal phone numbers, physical home addresses, national IDs) and message contents (body, attachments) are strictly excluded from all exports.

9. **HR-Derived Signals (`hr_flag`):**
   * HR flags (e.g., performance warnings, resignation notices) are **excluded by default** to prevent ethical bias, legal liability, and model proxy leakage.

---

## 4. Operational Governance & Risk Scoring Formula

To ensure alignment between the Cybersecurity domain rules and the AI model outputs:

$$\text{Risk Index} = \text{Behavioral Likelihood Score (0--100)} \times \text{Role Criticality Impact Factor (1.0--2.5)}$$

* **Likelihood Score:** Calculated by the AI Engine based on anomaly frequency, failed MFA rates, and simulation performance.
* **Impact Factor:** Derived directly from `role_criticality_tier` supplied by Cybersecurity.
* **Human-in-the-Loop Safeguard:** Risk scores generated by Red Engine are strictly advisory for security operations and cannot trigger automated employment actions.

---

## 5. Handover Manifest & Verification Checklist

- [x] All available source datasets mapped and exported.
- [x] Complete Data Dictionary provided with data types and missing value rules.
- [x] Stable pseudonymous employee key (`employee_id_hash`) verified across all tables.
- [x] Ground-truth incident label taxonomy defined (`CONFIRMED_INCIDENT` vs `FALSE_POSITIVE`).
- [x] Authorized simulation headers (`X-ImpersonaGuard-Simulation`) configured for SEG bypass.
- [x] Privacy controls, PII masking, and data minimization signed off.
- [x] Section 5 Handover Questions fully answered.

---
**Approved by:** Cybersecurity Team Lead (Mohamed)  
**Date:** Week 2 — Operational Handover
