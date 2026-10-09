# ⚙️ Release Operation Manual — Talaria

 **Document Status:** [CPSDCM-XXXX](https://jira.rakuten-it.com/jira/browse/CPSDCM-XXXX) — `Draft`
 **Operation ID:** `OPS-XXXX`
 **Last Updated:** YYYY-MM-DD
 **Initiator:** `{{OPERATOR_NAME}}`

---

## 📋 Table of Contents

1. [Summary](#1-summary)
2. [Classification & Risk Assessment](#2-classification--risk-assessment)
3. [Deployment Strategy](#3-deployment-strategy)
4. [Variable Parameters](#4-variable-parameters)
5. [Communication & Announcements](#5-communication--announcements)
6. [Prerequisite Check](#6-prerequisite-check)
7. [Login & Access Procedures](#7-login--access-procedures)
8. [Pre-Execution Checklist](#8-pre-execution-checklist)
9. [Buddy Role](#9-buddy-role)
10. [Abortion Criteria](#10-abortion-criteria)
11. [Operations Procedure](#11-operations-procedure)
12. [Rollback Procedure](#12-rollback-procedure)
13. [Post-Operation Monitoring](#13-post-operation-monitoring)
14. [Test Evidence](#14-test-evidence)
15. [Known Risks & Improvements](#15-known-risks--improvements)

---

## 1. Summary

| Field                | Value                                                                 |
|----------------------|-----------------------------------------------------------------------|
| **Operation ticket** | [CPSDCM-XXXX](https://jira.rakuten-it.com/jira/browse/CPSDCM-XXXX)   |
| **Summary**          | Release of a newer version of the Talaria API to `{{AZS}}`, with a chef org sync, database update, and rollout of the new application version. |
| **Details**          | An upgraded version of the Talaria API will be released to `{{AZS}}`. The process may or may not include a database update, as not every release requires one. |
| **Release Note — What's changed?** | `{{RELEASE_NOTES_LINK}}` |
| **QA Test Tickets**  | `{{QA_TEST_TICKETS_LINK}}` — testing is done against QA only; there are no separate BDD test tickets. |

 ℹ️ This manual covers both the **QA** and **BDD** release environments. Set `{{ENVIRONMENT}}` (Section 4) to `QA` or `BDD` before starting — every environment-specific value in this document is keyed off that switch.

---

## 2. Classification & Risk Assessment

 ⚠️ Default risk parameters below were reviewed and validated by a technical expert before approval.

 General classification specifications: [Risk Assessment Guidelines](https://confluence.rakuten-it.com/confluence/x/PQOoeQE)

Automation level specifications: [Automation Level Guidelines](https://confluence.rakuten-it.com/confluence/x/702oeQE)

| Parameter                            | Value       | Notes                                                          |
|--------------------------------------|-------------|-----------------------------------------------------------------|
| **Risk Level**                       | Medium      | Assessed by the initiator and validated by a tech expert — non-production environment (`{{ENVIRONMENT}}`), no production traffic, but execution is fully manual |
| **Automation Level**                 | `0`         | 0 = Fully manual, 1 = Partially automated, 2 = Fully automated  |
| **GMS Impact Possible**              | `No`        | Does this operation risk impacting the  Gross Merchandise Sales? |
| **Data Loss or Corruption Possible** | `No`        | Impact status: Minimal impact. Downtime: None.                  |

 **Example:**

 | Risk Level | Automation Level | GMS Impact | Data Loss |
 |------------|------------------|------------|-----------|
 | Medium     | 0                | No         | No        |

 **User Impact:** `{{USER_IMPACT}}`
 **Risks of Release (worst case):** Release Failure
 **External Dependency:** N/a

---

## 3. Deployment Strategy

- [ ] **Canary** — Route X% of traffic to new version before full rollout
- [ ] **Blue/Green** — Maintain two identical environments; switch traffic
- [ ] **Night Time** — Execute during low-traffic window (specify time & TZ)
- [x] **Rolling Update** — Incrementally replace instances
- [ ] **Feature Flag** — Gate new behaviour behind a runtime toggle
- [ ] **Other** — Describe: ___________

 **Justification / Notes:** Talaria is released to one server at a time within an availability zone.
 A smoke test is run after each data center before moving to the next.

---

## 4. Variable Parameters

> ⚠️ **CM Requirement:** All dynamic values **must** be defined here and separated from the procedure body.
> Variables must be validated during review **and** immediately before execution.
> Local file editing (e.g., `vi`) is **prohibited** — only values committed to Git are authoritative.

| Variable Name                              | Description                                                                                | Example Value              | Validated (✅/❌) |
|---------------------------------------------|-----------------------------------------------------------------------------------------------|-----------------------------|:----------------:|
| `{{OPERATION_TICKET_JIRA_LINK}}`              | Jira link to the operation ticket                                                              | `CPSDCM-XXXX`               | ❌               |
| `{{RELEASE_NOTES_LINK}}`                      | Link to the Confluence release notes for this release                                           | `https://confluence.rakuten-it.com/confluence/spaces/UCP/pages/6210496246/Talaria+Release+Notes+v2.4.1` | ❌ |
| `{{QA_TEST_TICKETS_LINK}}`                    | Link to the QA test Jira ticket(s) covering this release                                        | `https://jira.rakuten-it.com/jira/browse/BDG-4824` | ❌    |
| `{{ENVIRONMENT}}`                             | **Master switch** for this run — selects which environment-specific row/value applies throughout Sections 4, 11 and 12 | `QA` or `BDD`               | ❌               |
| `{{CHEF_ORGS_ALREADY_SYNCED}}`                | Whether chef org sync for `{{ENVIRONMENT}}` was already run in this same operation as part of another component's release (Keraunos/Trident) | `Yes` or `No` | ❌ |
| `{{DATE_TIME}}`                              | Date and time of the operation. Include start and end time.                                    | `2026-04-14 02:00–04:00 UTC`| ❌               |
| `{{RISK_LEVEL}}`                              | Risk level of the operation, taking into account service and operational risk                  | `Medium`                    | ❌               |
| `{{USER_IMPACT}}`                             | Impact on existing users if the operation is performed                                          | `None (non-production environment)` | ❌      |
| `{{OPERATOR_NAME}}` / `{{BUDDY_NAME}}`          | Operator of operation and Buddy (checker)                                                       | `gunalanshali01 / xulun01`  | ❌               |
| `{{OPERATOR_SHORT_ACCOUNT}}`                  | Short account username of operator                                                              | `gunalanshali01`            | ❌               |
| `{{AZS}}`                                     | The data center set where Talaria will be built, released and tested — resolves per `{{ENVIRONMENT}}` | `JPE1Z and JPE2Z` (QA) / `HND1 and HND2` (BDD) | ❌  |
| `{{NEW_TALARIA_VERSION}}`                     | Talaria repo version to rollout                                                                  | `v2.4.1`                    | ❌               |
| `{{CURRENT_TALARIA_VERSION}}`                 | Talaria repo version to rollback to in case of any issue (current version)                       | `v2.3.9`                    | ❌               |
| `{{SWITCH_APPLICATION_VERSION_JENKINS_URL}}`  | Jenkins URL used for switching the BMaaS application version — same job for QA and BDD           | `https://jpe2z-bdg-jenkins01.bmaas.jpe2z.dcnw.rakuten/view/BMaaS_Release/job/core%20resource%20group/job/bmaas/job/Switch_BMaaS_Application_Version/build?delay=0sec` | ❌ |
| `{{SYNC_CHEF_ORGS_JPE1Z_JENKINS_URL}}`        | Jenkins URL used to sync chef orgs on JPE1Z (QA only)                                            | `https://jpe1z-genesis-jenkins01.bmaas.jpe1z.dcnw.rakuten/job/sync_chef_orgs/build?delay=0sec` | ❌ |
| `{{SYNC_CHEF_ORGS_JPE2Z_JENKINS_URL}}`        | Jenkins URL used to sync chef orgs on JPE2Z (QA only)                                            | `https://jpe2z-genesis-jenkins01.bmaas.jpe2z.dcnw.rakuten/job/sync_chef_orgs/build?delay=0sec` | ❌ |
| `{{SYNC_CHEF_ORGS_BDD_JENKINS_URL}}`          | Jenkins URL used to sync chef orgs for BDD — a single run covers both HND1 and HND2     | `https://hnd1-genesis-jenkins01.bmaas.jpe1a.dcnw.rakuten/job/sync_chef_orgs/build?delay=0sec` | ❌ |
| `{{UPDATE_TALARIA_DATABASE_QA_JENKINS_URL}}`  | Jenkins URL used to update the Talaria database (QA)                                             | `https://jpe2z-bdg-jenkins01.bmaas.jpe2z.dcnw.rakuten/view/BMaaS_Release/job/core%20resource%20group/job/bmaas/job/QA_Talaria_Database_Update/build?delay=0sec` | ❌ |
| `{{UPDATE_TALARIA_DATABASE_BDD_JENKINS_URL}}` | Jenkins URL used to update the Talaria database (BDD)                                            | `https://jpe2z-bdg-jenkins01.bmaas.jpe2z.dcnw.rakuten/view/BMaaS_Release/job/core%20resource%20group/job/bmaas/job/Talaria_Database_Update/build?delay=0sec` | ❌ |
| `{{RELEASE_TALARIA_QA_JENKINS_URL}}`          | Jenkins URL used to release Talaria (QA)                                                          | `https://jpe2z-bdg-jenkins01.bmaas.jpe2z.dcnw.rakuten/view/BMaaS_Release/job/core%20resource%20group/job/bmaas/job/QA_Talaria_Release/build?delay=0sec` | ❌ |
| `{{RELEASE_TALARIA_BDD_JENKINS_URL}}`         | Jenkins URL used to release Talaria (BDD)                                                         | `https://jpe2z-bdg-jenkins01.bmaas.jpe2z.dcnw.rakuten/view/BMaaS_Release/job/core%20resource%20group/job/bmaas/job/Talaria_Release/build?delay=0sec` | ❌ |
| `{{TALARIA_LINK_JPE1Z}}`                      | Talaria application link used for the smoke test on JPE1Z (QA)                                    | `https://jpe1z-maas01.bmaas.jpe1z.dcnw.rakuten/talaria/` | ❌ |
| `{{TALARIA_LINK_JPE2Z}}`                      | Talaria application link used for the smoke test on JPE2Z (QA)                                    | `https://jpe2z-talaria.bmaas.jpe2z.dcnw.rakuten/talaria/` | ❌ |
| `{{TALARIA_SWAGGER_LINK_JPE1Z}}`              | Talaria Swagger UI link used for the smoke test on JPE1Z (QA)                                     | `https://jpe1z-maas01.bmaas.jpe1z.dcnw.rakuten/talaria/docs/` | ❌ |
| `{{TALARIA_SWAGGER_LINK_JPE2Z}}`              | Talaria Swagger UI link used for the smoke test on JPE2Z (QA)                                     | `https://jpe2z-talaria2.bmaas.jpe2z.dcnw.rakuten/talaria/docs/` | ❌ |
| `{{TALARIA_LINK_HND1}}`                       | Talaria application link used for the smoke test on HND1 (BDD)                                    | `https://j1m-prov.vip.hnd1.bdd.local/talaria/` | ❌ |
| `{{TALARIA_LINK_HND2}}`                       | Talaria application link used for the smoke test on HND2 (BDD)                                    | `https://j2m-prov.vip.hnd2.bdd.local/talaria/` | ❌ |
| `{{TALARIA_SWAGGER_LINK_HND1}}`               | Talaria Swagger UI link used for the smoke test on HND1 (BDD)                                     | `https://j1m-prov.vip.hnd1.bdd.local/talaria/docs/` | ❌ |
| `{{TALARIA_SWAGGER_LINK_HND2}}`               | Talaria Swagger UI link used for the smoke test on HND2 (BDD)                                     | `https://j2m-prov.vip.hnd2.bdd.local/talaria/docs/` | ❌ |

 ✅ **Pre-execution sign-off:** All variables above confirmed correct by `{{OPERATOR_NAME}}` / `{{BUDDY_NAME}}` on `YYYY-MM-DD HH:MM UTC`.

>If rollback is needed, call L4 and L3 to determine which rollback steps are required.
>If the rollback procedure does not result in full recovery, call L3 and L2 to decide next actions.
>If there is no impact, validate in the test environment before executing rollback actions.

---

## 5. Communication & Announcements

An announcement must be sent **before** the operation starts. Contacts/recipients of questions:

- `lun.xu@rakuten.com`
- `sascha.duschen@rakuten.com`

Announcements MUST be done as follows:
- Before starting the operation the details of the operations schedule, scope, impacts, risks and contact details MUST be announced.
- When the operation starts.
    - The Operator MUST reply to the operation details announcement that they are starting the operation.
- When the operation is paused as it will be resumed later, reply to the starting operation announcement.
- When the operation is resumed after being paused, reply to the operation paused announcement.
- After the operation ends.
    - The Checker/Buddy MUST reply to the operations latest announcement and confirm the operation is completed (i.e. This confirms that they have completed the predefined health checks and the operation has ended).

ALL CPSD members, including global teams must announce in:
- Viber channel: [[R] CPSD Changes & Ope. - All regions](https://invite.viber.com/?g2=AQBh0He79UC2pk%2FXuMYy6M8OpTQIqPT5G2DqRcMqq%2FsXIfKwA%2FMQPT3YkZL89PeE) (MANDATORY)
- Other channels defined by service owner.

**Announcement template:**

```
Hi all,
The CLSD Core Resource Group is performing a Talaria API release to {{AZS}}.

Please refer to the below for details.
================================================
■ Schedule
{{DATE_TIME}}

■ Outline
Release a new version of the Talaria API to {{AZS}}.
{{OPERATION_TICKET_JIRA_LINK}}

■ Purpose
To incorporate new features, bug fixes, enhancements, and data patches (if applicable).

■ Environment
{{AZS}}

■ Impact
{{USER_IMPACT}}

■ Risk
{{RISK_LEVEL}}

■ Target Component
{{AZS}}

■ Contact
lun.xu@rakuten.com
sascha.duschen@rakuten.com
================================================

If you have any questions and/or concerns, please let us know and we will assist you.
Thank you.
```
---

## 6. Prerequisite Check

 ✅ Verify that all required tools are available on the machine that will execute this operation **before** the maintenance window starts.

```bash
chef-client --version || echo "NOT INSTALLED: chef-client"
git --version         || echo "NOT INSTALLED: git"
curl --version         || echo "NOT INSTALLED: curl"
```

All tools must be present and at the expected version before proceeding to Section 7.

---

## 7. Login & Access Procedures

 ℹ️ SSH login is permitted to execute this procedure. All access must be traceable.

### 7.1 Required Access & Permissions

| System / Tool               | Access Method | Permission Level  | Notes                                                                |
|------------------------------|----------------|--------------------|-----------------------------------------------------------------------|
| Jenkins                     | Web UI         | Build/Execute      | Required to trigger and monitor deployment pipelines.                |
| Talaria                     | Swagger UI     | Execute/Run API    | Required to execute the smoke test.                                  |
| Chef Server/Nodes           | SSH / CLI      | Sudo / Root        | Required to execute chef-client and apply configuration changes.     |
| Target Servers `JPE1Z`/`JPE2Z` (QA) or `HND1`/`HND2` (BDD) | SSH | Sudo Privileges | Required for log analysis, troubleshooting, and manual service checks.|
| Version Control (Git)       | Git            | Read/Write         | Required to verify deployment tags or configuration manifests (if applicable). |

---

## 8. Pre-Execution Checklist

 ✅ All items must be checked before starting Step 1 of the procedure.
  Abortion is possible at any point before the Release sub-step in Section 11.

| #  | Check                                                                | Owner      | Status |
|----|-----------------------------------------------------------------------|------------|:------:|
| 1  | Jira ticket `CPSDCM-XXXX` is in `Approved` status                     | Initiator  | ❌     |
| 2  | All variable parameters validated and committed to Git                | Initiator  | ❌     |
| 3  | Rollback version (`{{CURRENT_TALARIA_VERSION}}`) is available and tested | Tech Lead  | ❌     |
| 4  | On-call engineer is available and notified                            | Team Lead  | ❌     |
| 5  | Maintenance window confirmed: `{{DATE_TIME}}`                           | Initiator  | ❌     |
| 6  | Announcement sent to all required channels (Section 5)                | Initiator  | ❌     |
| 7  | Buddy is confirmed and available (Section 9)                          | Initiator  | ❌     |
| 8  | Operator validated command pre-execution                              | Operator   | ❌     |
| 9 | Buddy validated command pre-execution                                 | Buddy      | ❌     |

---

## 9. Buddy Role

 ℹ️ The buddy is a second operator who monitors the execution in real time and is authorized to abort the operation at any point if something goes wrong. The buddy does not execute commands — they watch, validate, and act as a safety net.

 Operator / Buddy for this operation: `{{OPERATOR_NAME}}` / `{{BUDDY_NAME}}` (e.g. `gunalanshali01 / xulun01`).

 The names of the operator and buddy, along with their respective responsibilities for this operation, are recorded in the CPSDCM approval ticket below.

| Ticket          | Scope                                                                   |
|-----------------|----------------------------------------------------------------------------|
| `CPSDCM-XXXX`   | Official CM approval — operator and buddy assignment for this operation.  |

---

## 10. Abortion Criteria

> These are the global rollback/abortion criteria. These criteria apply throughout the entire Operations Procedure (Section 11).

If even one of the below is met, the operation must be aborted and rolled back:

- [ ] Application Down
- [ ] Smoke tests failed
- [ ] Multiple 500 Errors observed

> **On abortion:**
> - If rollback is needed, in most cases, the operator and buddy should judge whether a rollback/abortion is warranted.
> - If rollback is needed and the operator and buddy are not able to judge whether a rollback/abortion is warranted or if there is the risk of impact to other systems, contact L4 and L3 to determine which rollback steps are required.
> - If the rollback procedure does not result in full recovery, contact L3 and L2 to decide next actions.

---

## 11. Operations Procedure

 ⚠️ **CM Requirement:** Execution must be **segmented** — do not perform mass operations.
 Each step must be completed and validated before proceeding to the next.
 **Abortion is possible after any numbered step** unless stated otherwise.

 **If `{{ENVIRONMENT}}` = `QA`:** data centers for this operation are **JPE1Z, JPE2Z**.
 **If `{{ENVIRONMENT}}` = `BDD`:** data centers for this operation are **HND1, HND2**.

---

### Step 1 — Sync Chef Orgs

 **Scope:** Sync chef orgs for the target environment (`{{ENVIRONMENT}}`).
 **Abortion:** ✅ Safe to abort after this step.
 **Automation Level:** Scripted (Jenkins)

 **If `{{CHEF_ORGS_ALREADY_SYNCED}}` = `Yes`:** chef orgs for `{{ENVIRONMENT}}` are already in sync — skip this step and proceed to [Step 2](#step-2--switch-and-release-talaria).
 **If `{{CHEF_ORGS_ALREADY_SYNCED}}` = `No`:** continue below and run this step.

Build the `sync_chef_orgs` Jenkins pipeline with these common parameters:

```
Branch: master
SERVICE: bmaas
TARGET_GROUP: org
SKIP_UPDATE_CHEF_ENVIRONMENT: {UNCHECKED}
NOTIFY_SLACK: {UNCHECKED}
```

**If `{{ENVIRONMENT}}` = `QA`:** run the pipeline for each of the following three targets:

| DcCode | TARGET                | SOURCE_BRANCH | SLACK_CHANNEL | Jenkins URL                                  |
|--------|------------------------|----------------|------------------|------------------------------------------------|
| JPE1Z  | `roc_jpe1z_bmaas`      | `AT`           | `{EMPTY}`        | `{{SYNC_CHEF_ORGS_JPE1Z_JENKINS_URL}}`         |
| JPE1Z  | `rakuten_jp_onecloud`  | `AT`           | `{EMPTY}`        | `{{SYNC_CHEF_ORGS_JPE1Z_JENKINS_URL}}`         |
| JPE2Z  | `roc_jpe2z_bmaas`      | `AT`           | `{EMPTY}`        | `{{SYNC_CHEF_ORGS_JPE2Z_JENKINS_URL}}`         |

**If `{{ENVIRONMENT}}` = `BDD`:** run the pipeline once — it covers both HND1 and HND2:

| DcCode | TARGET                 | SOURCE_BRANCH | SLACK_CHANNEL        | Jenkins URL                           |
|--------|-------------------------|----------------|------------------------|-----------------------------------------|
| HND1   | `rakuten_jp_bdd_infra`  | `master`       | `#iac-genesis-prod`   | `{{SYNC_CHEF_ORGS_BDD_JENKINS_URL}}`    |

**Expected Result:** Each pipeline completes with `SUCCESS`.

**Validation:** Confirm `SUCCESS` in Jenkins console output for all jobs run for the selected `{{ENVIRONMENT}}`.

| ✅ Condition to proceed to Step 2                                      | ⛔ Action if condition not met            |
|---------------------------------------------------------------------|--------------------------------------------|
| All `sync_chef_orgs` jobs for the selected `{{ENVIRONMENT}}` report `SUCCESS` | Investigate failed job; do not proceed |

---

### Step 2 — Switch and Release Talaria

**Scope:** Talaria API deployment across both data centers for the target environment (`{{ENVIRONMENT}}`).
**Abortion:** ⚠️ Abortion after this step requires rollback of Talaria — see [Section 12](#12-rollback-procedure).
**Automation Level:** Scripted (Jenkins) + manual smoke test

**Switch the Talaria version** — Jenkins URL: `{{SWITCH_APPLICATION_VERSION_JENKINS_URL}}` (same job regardless of `{{ENVIRONMENT}}`)

**If `{{ENVIRONMENT}}` = `QA`:**
```
COMPONENT: talaria
VERSION_TYPE: latest
VERSION: {{NEW_TALARIA_VERSION}}
REVISION: {EMPTY}
```

**If `{{ENVIRONMENT}}` = `BDD`:**
```
COMPONENT: talaria
VERSION_TYPE: stable
VERSION: {{NEW_TALARIA_VERSION}}
REVISION: {EMPTY}
```

**Update the Talaria Database**

**If `{{ENVIRONMENT}}` = `QA`:** Jenkins URL: `{{UPDATE_TALARIA_DATABASE_QA_JENKINS_URL}}`
```
UPDATE_DATABASE: QA
DRY_RUN: {UNCHECKED}
```

**If `{{ENVIRONMENT}}` = `BDD`:** Jenkins URL: `{{UPDATE_TALARIA_DATABASE_BDD_JENKINS_URL}}`
```
UPDATE_DATABASE: JP1
DRY_RUN: {UNCHECKED}
```

**Release Talaria**

**If `{{ENVIRONMENT}}` = `QA`:** Jenkins URL: `{{RELEASE_TALARIA_QA_JENKINS_URL}}` — run once per data center:
- `TARGET_SERVERS: JPE1Z` / `DRY_RUN: {UNCHECKED}`
- `TARGET_SERVERS: JPE2Z` / `DRY_RUN: {UNCHECKED}`

**If `{{ENVIRONMENT}}` = `BDD`:** Jenkins URL: `{{RELEASE_TALARIA_BDD_JENKINS_URL}}` — run once per data center:
- `TARGET_SERVERS: HND1` / `DRY_RUN: {UNCHECKED}`
- `TARGET_SERVERS: HND2` / `DRY_RUN: {UNCHECKED}`

**Smoke Test for Talaria** — verify all endpoints return HTTP `200` on each data center:
```
GET /api/1.0/servers
GET /api/1.0/servers/{id}
GET /api/1.0/servers/{id}/event
POST /api/1.0/servers/tags
DELETE /api/1.0/servers/tags
POST /api/1.0/servers/install
POST /api/1.0/servers/destroy
POST /api/1.0/servers/health-check
```

**If `{{ENVIRONMENT}}` = `QA`:** test against:
- JPE1Z: `{{TALARIA_LINK_JPE1Z}}` / Swagger: `{{TALARIA_SWAGGER_LINK_JPE1Z}}`
- JPE2Z: `{{TALARIA_LINK_JPE2Z}}` / Swagger: `{{TALARIA_SWAGGER_LINK_JPE2Z}}`

**If `{{ENVIRONMENT}}` = `BDD`:** test against:
- HND1: `{{TALARIA_LINK_HND1}}` / Swagger: `{{TALARIA_SWAGGER_LINK_HND1}}`
- HND2: `{{TALARIA_LINK_HND2}}` / Swagger: `{{TALARIA_SWAGGER_LINK_HND2}}`

**Expected Result:** All endpoints return HTTP `200` on both data centers for the selected `{{ENVIRONMENT}}`.

**Validation:** Confirm HTTP `200` for every endpoint above, on both data centers for the selected `{{ENVIRONMENT}}`.

| ✅ Condition to mark operation complete                                          | ⛔ Action if condition not met  |
|--------------------------------------------------------------------------------|-----------------------------------|
| All Talaria endpoints return HTTP 200 on both data centers for `{{ENVIRONMENT}}` | Trigger rollback (Section 12)     |

---

## 12. Rollback Procedure
Action type: CHANGE

Why: As part of the rollback executed in case issues occur.

Any or all of the below steps are to be run, as necessary.

### 12.1 Rollback Trigger Criteria

Initiate rollback if **any** of the following conditions are met (identical to Section 10's global abortion criteria):

- [ ] Application Down
- [ ] Smoke tests failed
- [ ] Multiple 500 Errors observed

> ⚠️ **Escalation note:**
> - If rollback is needed, in most cases, the operator and buddy should judge whether a rollback/abortion is warranted.
> - If rollback is needed and the operator and buddy are not able to judge whether a rollback/abortion is warranted or if there is the risk of impact to other systems, contact L4 and L3 to determine which rollback steps are required.
> - If the rollback procedure does not result in full recovery, contact L3 and L2 to decide next actions.

### 12.2 Rollback Steps

**R1 — Switch and release Talaria (old version)**

**If `{{ENVIRONMENT}}` = `QA`:**
```
Jenkins URL: {{SWITCH_APPLICATION_VERSION_JENKINS_URL}}
COMPONENT: talaria
VERSION_TYPE: latest
VERSION: {{CURRENT_TALARIA_VERSION}}
REVISION: {EMPTY}
```

**If `{{ENVIRONMENT}}` = `BDD`:**
```
Jenkins URL: {{SWITCH_APPLICATION_VERSION_JENKINS_URL}}
COMPONENT: talaria
VERSION_TYPE: stable
VERSION: {{CURRENT_TALARIA_VERSION}}
REVISION: {EMPTY}
```

**If `{{ENVIRONMENT}}` = `QA`:**
```
Jenkins URL: {{UPDATE_TALARIA_DATABASE_QA_JENKINS_URL}}
UPDATE_DATABASE: QA
DRY_RUN: {UNCHECKED}
```
Release using `{{RELEASE_TALARIA_QA_JENKINS_URL}}` (`DRY_RUN: {UNCHECKED}`): `TARGET_SERVERS: JPE1Z` then `TARGET_SERVERS: JPE2Z`.

**If `{{ENVIRONMENT}}` = `BDD`:**
```
Jenkins URL: {{UPDATE_TALARIA_DATABASE_BDD_JENKINS_URL}}
UPDATE_DATABASE: JP1
DRY_RUN: {UNCHECKED}
```
Release using `{{RELEASE_TALARIA_BDD_JENKINS_URL}}` (`DRY_RUN: {UNCHECKED}`): `TARGET_SERVERS: HND1` then `TARGET_SERVERS: HND2`.

**R2 — Post-rollback task**
- [ ] Create trouble report.

### 12.3 RTO (Recovery Time Objective)

| Scenario                | Estimated RTO | Notes                        |
|--------------------------|----------------|--------------------------------|
| Rollback of Talaria      | `15 minutes`   | Both data centers for the selected `{{ENVIRONMENT}}` (JPE1Z/JPE2Z for QA, HND1/HND2 for BDD). |

**RTO Commitment:** `A maximum of 15 minutes`.

### 12.4 Post-Rollback Tasks

- Create trouble report.

---

## 13. Post-Operation Monitoring

**Monitoring Duration:** `30 minutes`

### 13.1 Metrics to Watch

| Metric                    | Expected State            | Trigger Rollback If…              |
|-----------------------------|------------------------------|--------------------------------------|
| Talaria logs                | No error spikes              | Application down / multiple 500 errors |
| Smoke test results (Section 11) | HTTP 200 on all endpoints    | Any smoke test failure               |

Smoke testing is conducted **during** the release operation. Regression testing is conducted **after** the release operation.

### 13.2 Sign-off

- [ ] Operator validated command pre-execution
- [ ] Buddy validated command pre-execution
- [ ] Operator validated result post-execution
- [ ] Buddy validated result post-execution
- [ ] Evidence completion evidence captured
- [ ] Execution time logged

 ✅ Monitoring completed. No anomalies detected. Signed off by `{{OPERATOR_NAME}}` / `{{BUDDY_NAME}}` on `YYYY-MM-DD HH:MM UTC`.

📢 Send the completion announcement to all required channels (Section 5) before closing this operation.

If anomalies are detected during monitoring, refer to [Section 12 — Rollback Procedure](#12-rollback-procedure).

---

## 14. Test Evidence

 ℹ️ **CM Requirement:** Test evidence of rollout execution must be included.

### 14.1 Rollout Test Evidence

| Test Date  | Availability zones | Executed By  | Outcome | Notes / Link to Results        |
|------------|-------------|--------------|:-------:|-----------------------------------|
| YYYY-MM-DD | `{{AZS}}` | `{{OPERATOR_NAME}}` | ⬜ | `[Link to results]` |

### 14.2 Rollback Test Evidence

 ⚠️ **Status:** Rollback test evidence is **pending** — must be completed by `YYYY-MM-DD`.

| Test Date  | Availability zones | Executed By  | Outcome | Notes / Link to Results |
|------------|-------------|--------------|:-------:|---------------------------|
| YYYY-MM-DD | `{{AZS}}` | `{{OPERATOR_NAME}}` | ⬜ | `[Link to results]` |

### 14.3 Template Approval Evidence

| Ticket          | Scope                                                                 |
|-----------------|--------------------------------------------------------------------------|
| `CPSDCM-XXXX`   | Official CM approval — identifies operator and buddy for this operation. |

---

## 15. Known Risks & Improvements

| # | Category | Status | Description |
|---|----------|--------|--------------|
| 1 |          |        |              |
| 2 |          |        |              |

 **Additional Notes:**
 If rollback is needed, call L4 and L3 to determine which rollback steps are required. If rollback does not result in full recovery, call L3 and L2 to decide next actions.

---
