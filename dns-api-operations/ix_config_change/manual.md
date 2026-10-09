# ⚙️ Release Operation Manual — DNS API I\*\*\*\*\*\*x Cutover

> **Document Status:** [CPSDCM-XXXX](https://jira.rakuten-it.com/jira/browse/CPSDCM-XXXX) — `Draft`
> **Operation ID:** `CPSDCM-XXXX`
> **Last Updated:** `YYYY-MM-DD`
> **Initiator:** `{{OPERATOR_NAME}}`

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
15. [Output Samples](#15-output-samples)
16. [Known Risks & Improvements](#16-known-risks--improvements)

---

## 1. Summary

| Field                | Value                                                                 |
|----------------------|-----------------------------------------------------------------------|
| **Operation ticket** | [CPSDCM-XXXX](https://jira.rakuten-it.com/jira/browse/CPSDCM-XXXX)   |
| **Summary**          | Cutover to a new I\*\*\*\*\*\*x server in the DNS API `{{ENV}}` environment. |
| **Details**          | A cutover to a new I\*\*\*\*\*\*x server is necessary so that newer versions of the I\*\*\*\*\*\*x service can be made available. |
| **GHE Commit hash Link — Configuration (if applicable)** | `{{CONFIG_ROLLOUT_COMMIT_HASH_LINK}}` |

---

## 2. Classification & Risk Assessment

| Parameter                            | Value                 | Notes                                                           |
|--------------------------------------|-----------------------|-----------------------------------------------------------------|
| **Risk Level**                       | `MEDIUM`        | Procedure risk: `LOW`. Service risk: `MEDIUM`. |
| **Automation Level**                 | `2`         | All CHANGE steps are executed by the operator via GitHub Actions. |
| **GMS Impact Possible**              | `NO` | There is no impact on the Gross Merchandise Sales as this is an internal service. |
| **Data Loss or Corruption Possible** | `NO` | **Impact Status:** No impact; **Downtime:** None (application pods are not touched — only DB configuration, K8s network policies and DNS server maintenance-mode flags change) |
| **External Dependency**              | `NO`                  | Internal DNS Administrator operation. No external dependencies. |

---

## 3. Deployment Strategy

- [ ] **Canary** — Route X% of traffic to new version before full rollout
- [ ] **Blue/Green** — Maintain two identical environments; switch traffic
- [ ] **Night Time** — Execute during low-traffic window (specify time & TZ)
- [ ] **Rolling Update** — Incrementally replace instances
- [ ] **Feature Flag** — Gate new behaviour behind a runtime toggle
- [x] **Other** — Describe: Direct data-patch rollout with a maintenance-mode gate

> **Justification / Notes:**
> This operation does not deploy application code and touches no pods. It involves setting a server in maintenance and the application of some K8s manifests and a Cassandra data patch. The old I\*\*\*\*\*\*x server is set in maintenance (i.e., it would only serve read requests and not write requests). After the DNS Group switches to the new I\*\*\*\*\*\*x server, a temporary network policy manifest is applied so that until the cache refreshes, both, the old and the new I\*\*\*\*\*\*x servers are accessible. Then, the DB data patch containing the configuration of the new I\*\*\*\*\*\*x server is applied. Once the cache is refreshed, the final network policy manifest is applied. Therefore, no downtime is seen.

---

## 4. Variable Parameters

> ⚠️ All dynamic values **must** be defined here and separated from the procedure body. Variables use `{{PLACEHOLDER}}` notation throughout. "Example Value" is populated only where an example was given. **Validated** defaults to ❌ pending confirmation before execution.

| Variable Name | Description | Example Value | Validated (✅/❌) |
|---|---|---|:---:|
| `{{OPERATION_TICKET_JIRA_LINK}}` | Jira link to the operation ticket | https://jira.rakuten-it.com/jira/browse/CPSDCM-XXXX | ❌ |
| `{{DATE_TIME}}` | Date and time of the operation. Include the start and end time. | `2026-03-21 1400-1600 hrs JST` | ❌ |
| `{{RISK_LEVEL}}` | Risk level of the operation, taking into account service and operational risk | `MEDIUM` | ❌ |
| `{{USER_IMPACT}}` | Impact on existing users if the operation is performed |  Read-only access to records in the DNS server from the start of the operation until the cutover is complete | ❌ |
| `{{OPERATOR_NAME}}` / `{{BUDDY_NAME}}` | Operator of operation and Buddy (checker) | `ar-srinivass01 / xulun01` | ❌ |
| `{{ENV}}` | Environment for which this operation is being performed | `PROD` | ❌ |
| `{{ENV_DOMAIN}}` | Domain of the environment that can be used for API calls | `dns-api.r-local.net` | ❌ |
| `{{ACTIVE_CLUSTER}}` (Active) | Active CaaS Cluster where the DNS API has been deployed | `jpe1-caas1-prod2` | ❌ |
| `{{STANDBY_CLUSTER}}` (Standby) | Standby CaaS Cluster where the DNS API has been deployed | `jpe2-caas1-prod2` | ❌ |
| `{{ACTIVE_CLUSTER_ACTIVE_COLOR}}` | The color of the side currently serving traffic on the Active cluster. Determined in Step 6B of Section 6. | `blue/green` | ❌ |
| `{{CONFIG_REPO_DOMAIN}}` | HTTPS Link needed to access the `apollo-automation` repo | https://github.com/Core-Resource-Group/apollo-automation.git | ❌ |
| `{{ACTIONS_WORKFLOW_TAG}}` | Short Commit hash of the tag whose GitHub Actions workflows would be run | `0eaae2c` | ❌ |
| `{{ACTIONS_WORKFLOW_TAG_LINK}}` | Link to the GHE tag whose GitHub Actions workflows would be run | https://github.com/Core-Resource-Group/apollo-automation/releases/tag/frozen-workflow-v1 | ❌ |
| `{{CONFIG_ROLLOUT_COMMIT_HASH}}` | Full Commit hash of the rollout tag in the `apollo-automation` repo | `182248ff3e0cd46d572d0306bb62e365ace489ed` | ❌ |
| `{{CONFIG_ROLLOUT_COMMIT_HASH_LINK}}` | Link to the GHE Commit hash for the configuration release | https://github.com/Core-Resource-Group/apollo-automation/commit/182248ff3e0cd46d572d0306bb62e365ace489ed | ❌ |
| `{{K8S_TEMPORARY_MANIFEST_FILE_PATH}}` | Path of the K8s temporary manifest file relative to the `apollo-automation` repo root - added so that until the cache refreshes, both, the new and old I\*\*\*\*\*\*x servers are accessible | `operations/temporary-rollout.yaml` | ❌ |
| `{{K8S_MANIFEST_FILE_PATH}}` | Path of the K8s manifest file to be applied finally relative to the `apollo-automation` repo root | `operations/rollout.yaml` | ❌ |
| `{{K8S_ROLLBACK_MANIFEST_FILE_PATH}}` | Path of the K8s rollback manifest file relative to the `apollo-automation` repo root | `operations/rollback.yaml` | ❌ |
| `{{DB_PATCH_FILE_PATH}}` | Path of the DB patch file relative to the `apollo-automation` repo root | `operations/rollout.cql` | ❌ |
| `{{DB_ROLLBACK_PATCH_FILE_PATH}}` | Path of the DB rollback patch file relative to the `apollo-automation` repo root | `operations/rollback.cql` | ❌ |
| `{{CASSANDRA_AZ_1}}` | First AZ of the Cassandra Cluster | `jpe2b` | ❌ |
| `{{CASSANDRA_AZ_1_FQDN_1}}` | FQDN of the first node in the first AZ of the Cassandra cluster | Resembling `omnia-apollo-cassandra-prod-012.dbaas.jpe2b.dcnw.rakuten` | ❌ |
| `{{CASSANDRA_AZ_1_FQDN_2}}` | FQDN of the second node in the first AZ of the Cassandra cluster | Resembling `omnia-apollo-cassandra-prod-012.dbaas.jpe2b.dcnw.rakuten` | ❌ |
| `{{CASSANDRA_AZ_1_FQDN_3}}` | FQDN of the third node in the first AZ of the Cassandra cluster | Resembling `omnia-apollo-cassandra-prod-012.dbaas.jpe2b.dcnw.rakuten` | ❌ |
| `{{CASSANDRA_AZ_2}}` | Second AZ of the Cassandra Cluster |  `jpw2a` | ❌ |
| `{{CASSANDRA_AZ_2_FQDN_1}}` | FQDN of the first node in the second AZ of the Cassandra cluster | Resembling `omnia-apollo-cassandra-prod-012.dbaas.jpe2b.dcnw.rakuten` | ❌ |
| `{{CASSANDRA_AZ_2_FQDN_2}}` | FQDN of the second node in the second AZ of the Cassandra cluster | Resembling `omnia-apollo-cassandra-prod-012.dbaas.jpe2b.dcnw.rakuten` | ❌ |
| `{{CASSANDRA_AZ_2_FQDN_3}}` | FQDN of the third node in the second AZ of the Cassandra cluster | Resembling `omnia-apollo-cassandra-prod-012.dbaas.jpe2b.dcnw.rakuten` | ❌ |
| `{{GRAFANA_LINK}}` | Grafana used to monitor application health — powered by the Blackbox Exporter | https://monitor.rakuten-it.com/v2/d/sOhyjvovk/api-health-dev?orgId=1&refresh=10s | ❌ |
| `{{GRAFANA_ENV}}` | Grafana Health Dashboard Environment Name | `Dev` / `Prod` | ❌ |

> ✅ **Pre-execution sign-off:** All variables above confirmed correct by `{{OPERATOR_NAME}}` / `{{BUDDY_NAME}}` on `YYYY-MM-DD HH:MM UTC`.

---

## 5. Communication & Announcements

An announcement must be sent **before** the operation starts. Contacts/recipients of questions:

- `sathvik.srinivas@rakuten.com`
- `nissar.pulikkil@rakuten.com`
- `lun.xu@rakuten.com`

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
[Operation Announcements]

Hi all,
The CLSD Core Resource Group is performing a DNS API enhancement in the {{ENV}} Environment

Please refer to the below for details.
[Operation Announcement]
================================================
■ Schedule
{{DATE_TIME}}

■ Outline
Cutover to a new DNS server in the DNS API {{ENV}} environment

{{OPERATION_TICKET_JIRA_LINK}}

■ Purpose
DNS Server Upgrade

■ Environment
{{ENV}}

■ Impact
{{USER_IMPACT}}

■ Risk
{{RISK_LEVEL}}

■ Target Component
CaaS Clusters: {{ACTIVE_CLUSTER}} (Active) and {{STANDBY_CLUSTER}} (Standby)
CaaS Namespace: dns-api-apollo

■ Contact
sathvik.srinivas@rakuten.com
nissar.pulikkil@rakuten.com
lun.xu@rakuten.com

■ Escalation Matrix
| Level    | Email                        | Designation |
| -------- | ---------------------------- | ----------- |
| Level 1  | lun.xu@rakuten.com           | L4          |
| Level 2  | ryo.kimura@rakuten.com       | L3          |
| Level 3  | daichi.hasegawa@rakuten.com  | L2          |
=========================================================

If you have any questions and/or concerns, please let us know and we will assist you.
Thank you.
```

---

## 6. Prerequisite Check

> ✅ Verify the following pre-validation checks pass before the maintenance window starts.

### Step 6A: Pre validation

**Action type:** CHECK
**Why:** Ensure the application is up and whether the operator has the necessary permissions to perform this operation.

- Check if the Environment's Swagger is accessible.

Access the Swagger of the environment through this link: `https://{{ENV_DOMAIN}}/swagger/index.html#`. It should load.

### Step 6B — Active Cluster Active Color Determination: Open the "View Istio VirtualService Weights on PROD, DEV or LAB" GitHub Actions Workflow

> **Action type:** CHECK
>
> **Why:** The current weights in the Active cluster need to be checked so that we can perform smoke tests on the active side of the active cluster.
>
> **Scope:** Read-only check of current Istio VirtualService traffic weights on `{{ACTIVE_CLUSTER}}`. No changes made.
>
> **Abortion:** ✅ Safe to abort — this step makes no changes.
>
> **Automation Level:** Pipeline (GitHub Actions Workflow)

Link: `{{CONFIG_REPO_DOMAIN}}/actions/workflows/view-virtualservice-weights.yml`

The below workflow with the passed values includes the following steps:
1. Set up of `kubectl`
2. Viewing and Parsing of the Istio VirtualService resource's details

**Appearance:**

![GitHub Actions View Virtual Service Workflow Input](https://confluence.rakuten-it.com/confluence/download/attachments/6941345438/virtualserviceget.png)

Click "Run Workflow" and set:
- **Use workflow from:** Tag `{{ACTIONS_WORKFLOW_TAG}}`
- **Cluster whose VirtualService weights you want to view:** `{{ACTIVE_CLUSTER}}`

**Expected Result:**

```
blue=**
green=**
```

**Validation:** One side shows 100% and the other 0%.

> **Convention:** the side currently at 100% traffic is called `{{ACTIVE_CLUSTER_ACTIVE_COLOR}}`.

All checks above must pass before proceeding to Section 8.

---

## 7. Login & Access Procedures

### 7.1 Required Access & Permissions

| System / Tool | Access Method | Permission Level | Notes |
|----------------|----------------|-------------------|-------|
| GitHub Actions | Web UI | admin | User should have access to view the application and the configuration repos and trigger GitHub Actions |
| Grafana Dashboard | Web UI | viewer | User should be a part of the `dns-api` tenant on OneCloud and should have the required Mon-aaS role. |

---

## 8. Pre-Execution Checklist

> ✅ All items must be checked before starting Step 1 of the Operations Procedure (Section 11). Note the execution time against each item.

| #  | Check                                                           | Owner      | Status |
|----|-------------------------------------------------------------------|------------|:------:|
| 1  | Jira ticket is in `Approved` status                                | Initiator  | ❌     |
| 2  | Ensure that within commit `{{CONFIG_ROLLOUT_COMMIT_HASH}}`, ensure that `{{DB_PATCH_FILE_PATH}}` and `{{DB_ROLLBACK_PATCH_FILE_PATH}}` exist and their contents are subjected to a visual inspection and an approval process                             | Initiator  | ❌     |
| 3  | All variable parameters validated                                  | Initiator  | ❌     |
| 4  | Operator acknowledges pre-validation steps (Section 6) completed with expected result before executing change | Operator | ❌ |
| 5  | Operator acknowledges that the GHE Commit hash linked to `{{ACTIONS_WORKFLOW_TAG_LINK}}` is `0eaae2c` | Operator | ❌ |
| 6  | Buddy acknowledges pre-validation steps (Section 6) completed with expected result before executing change | Buddy | ❌ |
| 7  | On-call engineer is available and notified                         | Team Lead  | ❌     |
| 8  | Announcement sent to all required channels (Section 5)             | Initiator  | ❌     |
| 9  | Time is logged                                                      | Operator   | ❌     |

---

## 9. Buddy Role

> ℹ️ The buddy is a second operator who monitors the execution in real time and is authorized to abort the operation at any point if something goes wrong. The operator/buddy pair is recorded via the `{{OPERATOR_NAME}} / {{BUDDY_NAME}}` variable (Section 4); a rollback decision requires the Buddy to weigh in. Only in situations where there is a risk of impact to other systems do we go for L3/L4 escalation (Section 10).

| Ticket | Scope |
|--------|-------|
| `CPSDCM-XXXX` | Official CM approval ticket — operator (`{{OPERATOR_NAME}}`) and buddy (`{{BUDDY_NAME}}`) assignment for this operation. |

---

## 10. Abortion Criteria

> These are the global rollback/abortion criteria. These criteria apply throughout the entire Operations Procedure (Section 11).

If even one of the below is met, the operation must be aborted and rolled back:

- [ ] Smoke tests failed
- [ ] Multiple 500 Errors observed

> **On abortion:**
> - If rollback is needed, in most cases, the operator and buddy should judge whether a rollback/abortion is warranted.
> - If rollback is needed and the operator and buddy are not able to judge whether a rollback/abortion is warranted or if there is the risk of impact to other systems, contact L4 and L3 to determine which rollback steps are required.
> - If the rollback procedure does not result in full recovery, contact L3 and L2 to decide next actions.

---

## 11. Operations Procedure

### Step 1 — Setting of the I\*\*\*\*\*\*x server in maintenance

> **Action type:** CHANGE
>
> **Why:** To enable read-only access so that no records are created/modified/deleted while the operation is going on.
>
> **Scope:** ⚠️ This is the step that enables read-only access to records residing on the I\*\*\*\*\*\*x server so that no records are created/modified/deleted on the server while the operation is going on.
>
> **Abortion:** ✅ Safe to abort — this is the first step. No impact.
>
> **Automation Level:** Pipeline (GitHub Actions Workflow)


Link: `{{CONFIG_REPO_DOMAIN}}/actions/workflows/manage-maintenance-status.yml`

The below workflow with the passed values includes the following steps:
1. Bearer token generation using the OneCloud Token API
2. Setting the I\*\*\*\*\*\*x server in maintenance mode

**Appearance:**

![GitHub Actions Manage Maintenance Status Workflow Input](https://confluence.rakuten-it.com/confluence/download/attachments/6941345438/managemaintenancestatus.png)

Click "Run Workflow" and set:
- **Use workflow from:** Tag `{{ACTIONS_WORKFLOW_TAG}}`
- **Operation to perform:** `PUT`
- **Target environment:** `{{ENV}}`
- **Availability zone:** `local`
- **Set maintenance status (defaults to false):** `true`

**Expected Result:** Success
**Validation:** You will see success (HTTP Status Code 200) with the message `Maintenance status of availability zone local has been set to true` under the `success` field.


| ✅ Condition to proceed to the next step | ⛔ Action if condition not met |
|---|---|
| Maintenance mode set successfully for `local` | If the AZ fails to set — do not proceed to the next step; investigate and escalate per Section 10 if needed. |

---

**WAIT FOR THE DNS GROUP to complete their migration**

---

### Step 2 — Active Cluster: Application of Temporary Manifest

> **Action type:** CHANGE
>
> **Why:** So that in the time between the DB patch and the cache refresh, the old and new I\*\*\*\*\*\*x servers are accessible from the deployments in the active cluster.
>
> **Scope:** Applies a temporary manifest containing a network policy that permits the IPs of both, the old and new I\*\*\*\*\*\*x servers. No application pods are touched.
>
> **Abortion:** ✅ Safe to abort — the temporary manifest only adds access to the new I\*\*\*\*\*\*x server alongside the existing old I\*\*\*\*\*\*x server access; if aborted, apply the rollback manifest (Section 12.2) and escalate per Section 10.
>
> **Automation Level:** Pipeline (GitHub Actions Workflow)

Link: `{{CONFIG_REPO_DOMAIN}}/actions/workflows/apply-k8s-manifest.yml`

The below workflow with the passed values includes the following steps:
1. Checkout of the passed commit hash
2. Restoration of the git-crypt symmetric key and unlocking of the repository
3. Validation of the manifest (path validation, validation of existence in the repo, format validation)
4. Application of the manifest
5. Locking of the repository

**Appearance:**

![GitHub Actions K8s Manifest Application Workflow Input](https://confluence.rakuten-it.com/confluence/download/attachments/6941345438/k8smanifestapplication.png)

Click "Run Workflow" and set:
- **Use workflow from:** Tag `{{ACTIONS_WORKFLOW_TAG}}`
- **Cluster to apply to:** `{{ACTIVE_CLUSTER}}`
- **Full Configuration commit hash to check out (blank = default branch):** `{{CONFIG_ROLLOUT_COMMIT_HASH}}`
- **Path to the Kubernetes manifest YAML to apply, relative to the repository root:** `{{K8S_TEMPORARY_MANIFEST_FILE_PATH}}`

**Expected Result:** Success.
**Validation:** Confirm the command completed with no error output on the GH Actions Console.

| ✅ Condition to proceed to the next step | ⛔ Action if condition not met |
|---|---|
| Manifest applied without error | Do not proceed — investigate the failure by checking the contents of the `{{K8S_TEMPORARY_MANIFEST_FILE_PATH}}` in the repo and escalate per Section 10 if needed. |

---

### Step 3 — Standby Cluster: Application of Temporary Manifest

> **Action type:** CHANGE
>
> **Why:** So that in the time between the DB patch and the cache refresh, the old and new I\*\*\*\*\*\*x servers are accessible from the deployments in the standby cluster.
>
> **Scope:** Applies a temporary manifest containing a network policy that permits the IPs of both, the old and new I\*\*\*\*\*\*x servers. No application pods are touched.
>
> **Abortion:** ✅ Safe to abort — the temporary manifest only adds access to the new I\*\*\*\*\*\*x server alongside the existing old I\*\*\*\*\*\*x server access; if aborted, apply the rollback manifest (Section 12.2) and escalate per Section 10.
>
> **Automation Level:** Pipeline (GitHub Actions Workflow)

Link: `{{CONFIG_REPO_DOMAIN}}/actions/workflows/apply-k8s-manifest.yml`

The below workflow with the passed values includes the following steps:
1. Checkout of the passed commit hash
2. Restoration of the git-crypt symmetric key and unlocking of the repository
3. Validation of the manifest (path validation, validation of existence in the repo, format validation)
4. Application of the manifest
5. Locking of the repository

**Appearance:**

![GitHub Actions K8s Manifest Application Workflow Input](https://confluence.rakuten-it.com/confluence/download/attachments/6941345438/k8smanifestapplication.png)

Click "Run Workflow" and set:
- **Use workflow from:** Tag `{{ACTIONS_WORKFLOW_TAG}}`
- **Cluster to apply to:** `{{STANDBY_CLUSTER}}`
- **Full Configuration commit hash to check out (blank = default branch):** `{{CONFIG_ROLLOUT_COMMIT_HASH}}`
- **Path to the Kubernetes manifest YAML to apply, relative to the repository root:** `{{K8S_TEMPORARY_MANIFEST_FILE_PATH}}`

**Expected Result:** Success.
**Validation:** Confirm the command completed with no error output on the GH Actions Console.

| ✅ Condition to proceed to the next step | ⛔ Action if condition not met |
|---|---|
| Manifest applied without error | Do not proceed — investigate the failure by checking the contents of the `{{K8S_TEMPORARY_MANIFEST_FILE_PATH}}` in the repo and escalate per Section 10 if needed. |

---

### Step 4 — Data Patch Incorporation after successful migration

> **Action type:** CHANGE
>
> **Why:** The configuration of the new I\*\*\*\*\*\*x server needs to be added.
>
> **Scope:** Applies a CQL patch to the shared, multi-AZ Cassandra DB. The patch contains the configuration of the new I\*\*\*\*\*\*x server (which is not set in maintenance). No application pods are touched.
>
> **Abortion:** ⚠️ Not trivially reversible — the patch affects the shared database. If this must be undone, use the rollback data patch procedure (Section 12.2) and escalate per Section 10.
>
> **Automation Level:** Pipeline (GitHub Actions Workflow)

Link: `{{CONFIG_REPO_DOMAIN}}/actions/workflows/apply-cql-data-patch.yml`

The below workflow with the passed values includes the following steps:
1. Checkout of the passed commit hash
2. Restoration of the git-crypt symmetric key and unlocking of the repository
3. Validation of the data patch file (path validation, validation of existence in the repo, format validation)
4. Creation of a Cassandra credentials file containing the database credentials (used for logging in to the database)
5. Application of the data patch
6. Removal of the Cassandra credentials file
7. Locking of the repository

**Appearance:**

![GitHub Actions Data Patch Workflow Input](https://confluence.rakuten-it.com/confluence/download/attachments/6941345438/datapatch.png)

Click "Run Workflow" and set:
- **Use workflow from:** Tag `{{ACTIONS_WORKFLOW_TAG}}`
- **Target environment:** `{{ENV}}`
- **Full Configuration commit hash to check out (blank = default branch):** `{{CONFIG_ROLLOUT_COMMIT_HASH}}`
- **Path to the CQL file to apply, relative to the repository root:** `{{DB_PATCH_FILE_PATH}}`

**Expected Result:** Success.
**Validation:** Confirm the command completed with no error output on the GH Actions Console.

| ✅ Condition to proceed to the next step | ⛔ Action if condition not met |
|---|---|
| Patch applied without error | Do not proceed with a partially-applied patch — investigate the failure by checking the contents of the `{{DB_PATCH_FILE_PATH}}` in the repo and escalate per Section 10 if needed. |

---

### Step 5 — Active Cluster: Application of Final Manifest

> **Action type:** CHANGE
>
> **Why:** So that only the new I\*\*\*\*\*\*x server is made accessible from the deployments in the active cluster.
>
> **Scope:** Applies the final manifest containing a network policy that permits the IPs of only the new I\*\*\*\*\*\*x server. No application pods are touched.
>
> **Abortion:** ⚠️ Not a simple abortion — this step removes access to the old I\*\*\*\*\*\*x server; if aborted, apply the rollback data patch and the rollback manifest (Section 12.2) and escalate per Section 10.
>
> **Automation Level:** Pipeline (GitHub Actions Workflow)

Link: `{{CONFIG_REPO_DOMAIN}}/actions/workflows/apply-k8s-manifest.yml`

The below workflow with the passed values includes the following steps:
1. Checkout of the passed commit hash
2. Restoration of the git-crypt symmetric key and unlocking of the repository
3. Validation of the manifest (path validation, validation of existence in the repo, format validation)
4. Application of the manifest
5. Locking of the repository

**Appearance:**

![GitHub Actions K8s Manifest Application Workflow Input](https://confluence.rakuten-it.com/confluence/download/attachments/6941345438/k8smanifestapplication.png)

After 5 minutes, Click "Run Workflow" and set:
- **Use workflow from:** Tag `{{ACTIONS_WORKFLOW_TAG}}`
- **Cluster to apply to:** `{{ACTIVE_CLUSTER}}`
- **Full Configuration commit hash to check out (blank = default branch):** `{{CONFIG_ROLLOUT_COMMIT_HASH}}`
- **Path to the Kubernetes manifest YAML to apply, relative to the repository root:** `{{K8S_MANIFEST_FILE_PATH}}`

**Expected Result:** Success.
**Validation:** Confirm the command completed with no error output on the GH Actions Console.

| ✅ Condition to proceed to the next step | ⛔ Action if condition not met |
|---|---|
| Manifest applied without error | Do not proceed — investigate the failure by checking the contents of the `{{K8S_MANIFEST_FILE_PATH}}` in the repo and escalate per Section 10 if needed. |

---

### Step 6 — Standby Cluster: Application of Final Manifest

> **Action type:** CHANGE
>
> **Why:** So that only the new I\*\*\*\*\*\*x server is made accessible from the deployments in the standby cluster.
>
> **Scope:** Applies the final manifest containing a network policy that permits the IPs of only the new I\*\*\*\*\*\*x server. No application pods are touched.
>
> **Abortion:** ⚠️ Not a simple abortion — this step removes access to the old I\*\*\*\*\*\*x server; if aborted, apply the rollback data patch and the rollback manifest (Section 12.2) and escalate per Section 10.
>
> **Automation Level:** Pipeline (GitHub Actions Workflow)

Link: `{{CONFIG_REPO_DOMAIN}}/actions/workflows/apply-k8s-manifest.yml`

The below workflow with the passed values includes the following steps:
1. Checkout of the passed commit hash
2. Restoration of the git-crypt symmetric key and unlocking of the repository
3. Validation of the manifest (path validation, validation of existence in the repo, format validation)
4. Application of the manifest
5. Locking of the repository

**Appearance:**

![GitHub Actions K8s Manifest Application Workflow Input](https://confluence.rakuten-it.com/confluence/download/attachments/6941345438/k8smanifestapplication.png)

Click "Run Workflow" and set:
- **Use workflow from:** Tag `{{ACTIONS_WORKFLOW_TAG}}`
- **Cluster to apply to:** `{{STANDBY_CLUSTER}}`
- **Full Configuration commit hash to check out (blank = default branch):** `{{CONFIG_ROLLOUT_COMMIT_HASH}}`
- **Path to the Kubernetes manifest YAML to apply, relative to the repository root:** `{{K8S_MANIFEST_FILE_PATH}}`

**Expected Result:** Success.
**Validation:** Confirm the command completed with no error output on the GH Actions Console.

| ✅ Condition to proceed to the next step | ⛔ Action if condition not met |
|---|---|
| Manifest applied without error | Do not proceed — investigate the failure by checking the contents of the `{{K8S_MANIFEST_FILE_PATH}}` in the repo and escalate per Section 10 if needed. |

---

### Step 7 — Smoke Testing

> **Action type:** CHANGE
>
> **Why:** Ensure that the API functionalities are as expected.
>
> **Scope:** Creates, patches, reads and deletes a single test DNS record (`test-smoke`) via the DNS API. Confined to one test hostname; no live user records are affected.
>
> **Abortion:** ✅ Safe to abort — this is a dummy record under the passed tenant and does not impact any service.
>
> **Automation Level:** Pipeline (GitHub Actions Workflow)

**Legend**

- **`{{TENANT}}`:** A valid OneCloud tenant marked as a `tam` tenant by the admins of the DNS API
- **`{{SUBDOMAIN}}`:** A valid subdomain under `jp.local` like `dev`, `stg` and `prod`.
- **`{{ZONE}}`:** The DNS zone under which we wish to create records. Format: `{{SUBDOMAIN}}.jp.local.`.
- **`{{VALID_IP_LINKED_TO_THE_PASSED_ZONE}}`:** IP from a subnet assigned to the `{{TENANT}}` on the NMC.
- **`{{ANOTHER_VALID_IP_LINKED_TO_THE_PASSED_ZONE}}`:** Another IP from a subnet assigned to the `{{TENANT}}` on the NMC.

Link: `{{CONFIG_REPO_DOMAIN}}/actions/workflows/smoke-test-dns-api.yml`

The below workflow with the passed values includes the following steps:
1. Checkout of the repository
2. Set up of the Go GitHub Action
3. Execution of the smoke test with the passed parameters via a Go script (includes create record, update record, get record and delete record)

**Appearance:**

![GitHub Actions Smoke Test Workflow Input](https://confluence.rakuten-it.com/confluence/download/attachments/6941345438/smoketest.png)

Click "Run Workflow" and set:
- **Use workflow from:** Tag `{{ACTIONS_WORKFLOW_TAG}}`
- **Target Environment:** `{{ENV}}`
- **Side to smoke test against (Color - blue or green) (choose either if the target is the QA environment as this value is irrelevant there):** `{{ACTIVE_CLUSTER_ACTIVE_COLOR}}`
- **Cluster to smoke test against (QA and LAB have no Standby cluster — use ACTIVE there):** `ACTIVE`
- **OneCloud tenant to smoke test against:** `{{TENANT}}`
- **DNS zone to create the record under, e.g. <tenant>.<az>.dcnw.rakuten., <subdomain>.jp.local., etc.:** `{{ZONE}}`
- **Record type:** `Address`
- **Record hostname:** `test-smoke-{{CONFIG_ROLLOUT_COMMIT_HASH}}`
- **Content to create the record with, e.g. a single IP address for an Address record and a canonical name for a CNAME record:** `{{VALID_IP_LINKED_TO_THE_PASSED_ZONE}}`
- **Different content to patch the record to:** `{{ANOTHER_VALID_IP_LINKED_TO_THE_PASSED_ZONE}}`

**Expected Result:** All four smoke-test calls succeed. A sample of the output can be found in Section 15.1.
**Validation:** You will see success (HTTP Status Code 200) in all 4 API calls with POST, PATCH and DELETE have 2, 3 and 2 entries in the `success` field of the response.


| ✅ Condition to proceed to the next step | ⛔ Action if condition not met |
|---|---|
| All four smoke-test calls return the results described above | Any call fails, or logs an unexpected error — investigate and abort per Section 10 if necessary |

---

## 12. Rollback Procedure

> **Action type:** CHANGE
>
> **Why:** As part of the rollback executed in case issues occur.

> Any or all of the below steps are to be run, as necessary.

### 12.1 Rollback Trigger Criteria

Initiate rollback if **any** of the following conditions are met (identical to Section 10's global abortion criteria):

- [ ] Smoke tests failed
- [ ] Multiple 500 Errors observed

> ⚠️ **Escalation note:**
> - If rollback is needed, in most cases, the operator and buddy should judge whether a rollback/abortion is warranted.
> - If rollback is needed and the operator and buddy are not able to judge whether a rollback/abortion is warranted or if there is the risk of impact to other systems, contact L4 and L3 to determine which rollback steps are required.
> - If the rollback procedure does not result in full recovery, contact L3 and L2 to decide next actions.

### 12.2 Rollback Steps

**Rollback duration:** ~5 minutes

#### Step 12.2.1 — Active Cluster: Application of Temporary Manifest

Link: `{{CONFIG_REPO_DOMAIN}}/actions/workflows/apply-k8s-manifest.yml`

The below workflow with the passed values includes the following steps:
1. Checkout of the passed commit hash
2. Restoration of the git-crypt symmetric key and unlocking of the repository
3. Validation of the manifest (path validation, validation of existence in the repo, format validation)
4. Application of the manifest
5. Locking of the repository

**Appearance:**

![GitHub Actions K8s Manifest Application Workflow Input](https://confluence.rakuten-it.com/confluence/download/attachments/6941345438/k8smanifestapplication.png)

Click "Run Workflow" and set:
- **Use workflow from:** Tag `{{ACTIONS_WORKFLOW_TAG}}`
- **Cluster to apply to:** `{{ACTIVE_CLUSTER}}`
- **Full Configuration commit hash to check out (blank = default branch):** `{{CONFIG_ROLLOUT_COMMIT_HASH}}`
- **Path to the Kubernetes manifest YAML to apply, relative to the repository root:** `{{K8S_TEMPORARY_MANIFEST_FILE_PATH}}`

**Expected Result:** Success.
**Validation:** Confirm the command completed with no error output on the GH Actions Console.

---

#### Step 12.2.2 — Standby Cluster: Application of Temporary Manifest

Link: `{{CONFIG_REPO_DOMAIN}}/actions/workflows/apply-k8s-manifest.yml`

The below workflow with the passed values includes the following steps:
1. Checkout of the passed commit hash
2. Restoration of the git-crypt symmetric key and unlocking of the repository
3. Validation of the manifest (path validation, validation of existence in the repo, format validation)
4. Application of the manifest
5. Locking of the repository

**Appearance:**

![GitHub Actions K8s Manifest Application Workflow Input](https://confluence.rakuten-it.com/confluence/download/attachments/6941345438/k8smanifestapplication.png)

Click "Run Workflow" and set:
- **Use workflow from:** Tag `{{ACTIONS_WORKFLOW_TAG}}`
- **Cluster to apply to:** `{{STANDBY_CLUSTER}}`
- **Full Configuration commit hash to check out (blank = default branch):** `{{CONFIG_ROLLOUT_COMMIT_HASH}}`
- **Path to the Kubernetes manifest YAML to apply, relative to the repository root:** `{{K8S_TEMPORARY_MANIFEST_FILE_PATH}}`

**Expected Result:** Success.
**Validation:** Confirm the command completed with no error output on the GH Actions Console.

---

#### Step 12.2.3 — Data Patch Incorporation

Please note that the patch contains the configuration of the old I\*\*\*\*\*\*x server (which is not set in maintenance).

Link: `{{CONFIG_REPO_DOMAIN}}/actions/workflows/apply-cql-data-patch.yml`

The below workflow with the passed values includes the following steps:
1. Checkout of the passed commit hash
2. Restoration of the git-crypt symmetric key and unlocking of the repository
3. Validation of the data patch file (path validation, validation of existence in the repo, format validation)
4. Creation of a Cassandra credentials file containing the database credentials (used for logging in to the database)
5. Application of the data patch
6. Removal of the Cassandra credentials file
7. Locking of the repository

**Appearance:**

![GitHub Actions Data Patch Workflow Input](https://confluence.rakuten-it.com/confluence/download/attachments/6941345438/datapatch.png)

Click "Run Workflow" and set:
- **Use workflow from:** Tag `{{ACTIONS_WORKFLOW_TAG}}`
- **Target environment:** `{{ENV}}`
- **Full Configuration commit hash to check out (blank = default branch):** `{{CONFIG_ROLLOUT_COMMIT_HASH}}`
- **Path to the CQL file to apply, relative to the repository root:** `{{DB_ROLLBACK_PATCH_FILE_PATH}}`

**Expected Result:** Success.
**Validation:** Confirm the command completed with no error output on the GH Actions Console.

---

#### Step 12.2.4 — Active Cluster: Application of Rollback Manifest

Link: `{{CONFIG_REPO_DOMAIN}}/actions/workflows/apply-k8s-manifest.yml`

The below workflow with the passed values includes the following steps:
1. Checkout of the passed commit hash
2. Restoration of the git-crypt symmetric key and unlocking of the repository
3. Validation of the manifest (path validation, validation of existence in the repo, format validation)
4. Application of the manifest
5. Locking of the repository

**Appearance:**

![GitHub Actions K8s Manifest Application Workflow Input](https://confluence.rakuten-it.com/confluence/download/attachments/6941345438/k8smanifestapplication.png)

After 5 minutes, Click "Run Workflow" and set:
- **Use workflow from:** Tag `{{ACTIONS_WORKFLOW_TAG}}`
- **Cluster to apply to:** `{{ACTIVE_CLUSTER}}`
- **Full Configuration commit hash to check out (blank = default branch):** `{{CONFIG_ROLLOUT_COMMIT_HASH}}`
- **Path to the Kubernetes manifest YAML to apply, relative to the repository root:** `{{K8S_ROLLBACK_MANIFEST_FILE_PATH}}`

**Expected Result:** Success.
**Validation:** Confirm the command completed with no error output on the GH Actions Console.

---

#### Step 12.2.5 — Standby Cluster: Application of Rollback Manifest

Link: `{{CONFIG_REPO_DOMAIN}}/actions/workflows/apply-k8s-manifest.yml`

The below workflow with the passed values includes the following steps:
1. Checkout of the passed commit hash
2. Restoration of the git-crypt symmetric key and unlocking of the repository
3. Validation of the manifest (path validation, validation of existence in the repo, format validation)
4. Application of the manifest
5. Locking of the repository

**Appearance:**

![GitHub Actions K8s Manifest Application Workflow Input](https://confluence.rakuten-it.com/confluence/download/attachments/6941345438/k8smanifestapplication.png)

Click "Run Workflow" and set:
- **Use workflow from:** Tag `{{ACTIONS_WORKFLOW_TAG}}`
- **Cluster to apply to:** `{{STANDBY_CLUSTER}}`
- **Full Configuration commit hash to check out (blank = default branch):** `{{CONFIG_ROLLOUT_COMMIT_HASH}}`
- **Path to the Kubernetes manifest YAML to apply, relative to the repository root:** `{{K8S_ROLLBACK_MANIFEST_FILE_PATH}}`

**Expected Result:** Success.
**Validation:** Confirm the command completed with no error output on the GH Actions Console.

---

#### Step 12.2.6 — Smoke Testing after Rollback

**Legend**

- **`{{TENANT}}`:** A valid OneCloud tenant marked as a `tam` tenant by the admins of the DNS API
- **`{{SUBDOMAIN}}`:** A valid subdomain under `jp.local` like `dev`, `stg` and `prod`.
- **`{{ZONE}}`:** The DNS zone under which we wish to create records. Format: `{{SUBDOMAIN}}.jp.local.`.
- **`{{VALID_IP_LINKED_TO_THE_PASSED_ZONE}}`:** IP from a subnet assigned to the `{{TENANT}}` on the NMC.
- **`{{ANOTHER_VALID_IP_LINKED_TO_THE_PASSED_ZONE}}`:** Another IP from a subnet assigned to the `{{TENANT}}` on the NMC.

Link: `{{CONFIG_REPO_DOMAIN}}/actions/workflows/smoke-test-dns-api.yml`

The below workflow with the passed values includes the following steps:
1. Checkout of the repository
2. Set up of the Go GitHub Action
3. Execution of the smoke test with the passed parameters via a Go script (includes create record, update record, get record and delete record)

**Appearance:**

![GitHub Actions Smoke Test Workflow Input](https://confluence.rakuten-it.com/confluence/download/attachments/6941345438/smoketest.png)

Click "Run Workflow" and set:
- **Use workflow from:** Tag `{{ACTIONS_WORKFLOW_TAG}}`
- **Target Environment:** `{{ENV}}`
- **Side to smoke test against (Color - blue or green) (choose either if the target is the QA environment as this value is irrelevant there):** `{{ACTIVE_CLUSTER_ACTIVE_COLOR}}`
- **Cluster to smoke test against (QA and LAB have no Standby cluster — use ACTIVE there):** `ACTIVE`
- **OneCloud tenant to smoke test against:** `{{TENANT}}`
- **DNS zone to create the record under, e.g. <tenant>.<az>.dcnw.rakuten., <subdomain>.jp.local., etc.:** `{{ZONE}}`
- **Record type:** `Address`
- **Record hostname:** `test-smoke-{{CONFIG_ROLLOUT_COMMIT_HASH}}`
- **Content to create the record with, e.g. a single IP address for an Address record and a canonical name for a CNAME record:** `{{VALID_IP_LINKED_TO_THE_PASSED_ZONE}}`
- **Different content to patch the record to:** `{{ANOTHER_VALID_IP_LINKED_TO_THE_PASSED_ZONE}}`

**Expected Result:** All four smoke-test calls succeed. A sample of the output can be found in Section 15.1.
**Validation:** You will see success (HTTP Status Code 200) in all 4 API calls with POST, PATCH and DELETE have 2, 3 and 2 entries in the `success` field of the response.

### 12.3 RTO (Recovery Time Objective)

| Scenario | Estimated RTO | Notes |
|----------|----------------|-------|
| Rollback data patch | ~5 minutes | Data Patch application plus cache-refresh wait. |

> **RTO Commitment:** A maximum of `10 minutes`.

### 12.4 Post-Rollback Tasks

- Create trouble report.
- Capture evidence of tests performed after rollback.

---

## 13. Post-Operation Monitoring

**Monitoring Duration:** `30 minutes` *(Recommended minimum: 10 min for Low risk, 30 min for Medium, 60 min for High)*

Navigate to `{{GRAFANA_LINK}}` and open the "DNS API Latency and Request Overview - `{{GRAFANA_ENV}}`" dashboard.

### 13.1 Metrics to Watch

| Metric | Dashboard | Trigger Rollback If… |
|--------|-----------|------------------------|
| Application Success Rate | DNS API Latency and Request Overview - `{{GRAFANA_ENV}}` | Success Rate less than 90% |

**Success Rate Dashboard Reference**

**Appearance:**

![Grafana Dashboard for Application Success Rate](https://confluence.rakuten-it.com/confluence/download/attachments/6941345438/successrate.png)

### 13.2 Sign-off

**Action type:** CHECK
**Why:** Acknowledge success.

Please note execution time against each item:

- [ ] Operator acknowledges pre-validation steps completed with expected result before executing change
- [ ] Buddy acknowledges pre-validation steps completed with expected result before executing change
- [ ] Operator validated command pre-execution
- [ ] Buddy validated command pre-execution
- [ ] Operator validated result post execution
- [ ] Buddy validated result post execution
- [ ] Evidence: Pre-execution, process and completion evidence captured
- [ ] Time is logged

> ✅ Monitoring completed. No anomalies detected. Signed off by `{{OPERATOR_NAME}}` / `{{BUDDY_NAME}}` on `YYYY-MM-DD HH:MM UTC`.
>
> 📢 Send the completion announcement to all required channels (Section 5) before closing this operation.

If anomalies are detected during monitoring, refer to [Section 12 — Rollback Procedure](#12-rollback-procedure).

---

## 14. Test Evidence

> ⚠️ **CM Requirement:** Test evidence of rollout execution must be included.

### 14.1 Rollout Test Evidence

| Test Date  | Environment | Executed By  | Outcome | Notes / Link to Results |
|------------|-------------|--------------|:-------:|--------------------------|
| `2026-10-08` | `QA`   | `ar-srinivass01` | ✅ | https://confluence.rakuten-it.com/confluence/spaces/UCP/pages/6970873712/Tests+run+in+the+QA+Environment+of+the+DNS+API |

### 14.2 Rollback Test Evidence

> ⚠️ **Status:** Rollback test evidence is **pending** — must be completed by `2027-06-01` (within 6 months of approval).

| Test Date  | Environment | Executed By  | Outcome | Notes / Link to Results |
|------------|-------------|--------------|:-------:|--------------------------|
| `YYYY-MM-DD` | `QA`   | `ar-srinivass01` | ⬜ | `[Link to results]` |

### 14.3 Template Approval Evidence

| Ticket | Scope |
|--------|-------|
| `CPSDCM-XXXX` | Official CM approval — identifies operator (`{{OPERATOR_NAME}}`) and buddy (`{{BUDDY_NAME}}`) for this operation. |

---

## 15. Output Samples

### 15.1 Smoke Testing
```bash
2026/09/15 09:19:49 DNS API release smoke test — tenant=dns-api zone=dns-api.jpw2x.dcnw.rakuten. record_type=Address color=blue cluster=ACTIVE environment_domain=*** target_ip=*** hostname=test-smoke-cd435a4
2026/09/15 09:19:49 Obtained ***
2026/09/15 09:19:49 --> POST https://***/api/v1/tenant/dns-api/zone/dns-api.jpw2x.dcnw.rakuten./record_type/Address/record_hostname/test-smoke-cd435a4?color=blue
Host: ***
Accept: application/json
Authorization: ***
Content-Type: application/json
body:
{
  "address_record_content": [
    "100.124.66.152"
  ]
}
2026/09/15 09:19:50 <-- 200 POST https://***/api/v1/tenant/dns-api/zone/dns-api.jpw2x.dcnw.rakuten./record_type/Address/record_hostname/test-smoke-cd435a4?color=blue
Content-Length: 304
Content-Type: application/json
Date: Tue, 15 Sep 2026 09:19:50 GMT
Strict-Transport-Security: max-age=31536000; includeSubDomains
Vary: Origin
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
X-Request-Id: 2804c45e-01bd-4e79-a8b2-d4d1441c7c3a
body:
{
  "success": [
    "test-smoke-cd435a4.dns-api.jpw2x.dcnw.rakuten. -\u003e 100.124.66.152 with ttl = 3600 and purpose =  created/updated.",
    "152.66.124.100.in-addr.arpa. -\u003e test-smoke-cd435a4.dns-api.jpw2x.dcnw.rakuten. with ttl = 3600 and purpose =  created/updated."
  ],
  "request_id": "2804c45e-01bd-4e79-a8b2-d4d1441c7c3a"
}
2026/09/15 09:19:50 Create record succeeded — record and PTR record created

--------------------------------------------------------------------------------

2026/09/15 09:19:50 --> PATCH https://***/api/v1/tenant/dns-api/zone/dns-api.jpw2x.dcnw.rakuten./record_type/Address/record_hostname/test-smoke-cd435a4?color=blue
Host: ***
Accept: application/json
Authorization: ***
Content-Type: application/json
body:
{
  "address_record_content": [
    "100.124.66.153"
  ]
}
2026/09/15 09:19:51 <-- 200 PATCH https://***/api/v1/tenant/dns-api/zone/dns-api.jpw2x.dcnw.rakuten./record_type/Address/record_hostname/test-smoke-cd435a4?color=blue
Content-Length: 391
Content-Type: application/json
Date: Tue, 15 Sep 2026 09:19:51 GMT
Strict-Transport-Security: max-age=31536000; includeSubDomains
Vary: Origin
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
X-Request-Id: f850468d-792f-4bdf-8e89-4ecea4b167e1
body:
{
  "success": [
    "test-smoke-cd435a4.dns-api.jpw2x.dcnw.rakuten. -\u003e 100.124.66.153 with ttl = 3600 and purpose =  created/updated.",
    "153.66.124.100.in-addr.arpa. -\u003e test-smoke-cd435a4.dns-api.jpw2x.dcnw.rakuten. with ttl = 3600 and purpose =  created/updated.",
    "152.66.124.100.in-addr.arpa. -\u003e test-smoke-cd435a4.dns-api.jpw2x.dcnw.rakuten. deleted."
  ],
  "request_id": "f850468d-792f-4bdf-8e89-4ecea4b167e1"
}
2026/09/15 09:19:51 Patch record succeeded — record updated, PTR record swapped

--------------------------------------------------------------------------------

2026/09/15 09:19:51 --> GET https://***/api/v1/tenant/dns-api/zone/dns-api.jpw2x.dcnw.rakuten./record_type/Address/record_hostname/test-smoke-cd435a4?color=blue
Host: ***
Accept: application/json
Authorization: ***
body:
(empty)
2026/09/15 09:19:52 <-- 200 GET https://***/api/v1/tenant/dns-api/zone/dns-api.jpw2x.dcnw.rakuten./record_type/Address/record_hostname/test-smoke-cd435a4?color=blue
Content-Length: 235
Content-Type: application/json
Date: Tue, 15 Sep 2026 09:19:52 GMT
Strict-Transport-Security: max-age=31536000; includeSubDomains
Vary: Origin
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
X-Request-Id: 64abcf60-761e-4b66-8e86-aa509072b87f
body:
{
  "records": {
    "zone": "dns-api.jpw2x.dcnw.rakuten.",
    "address": [
      {
        "hostname": "test-smoke-cd435a4",
        "fqdn": "test-smoke-cd435a4.dns-api.jpw2x.dcnw.rakuten.",
        "ip": "100.124.66.153",
        "ttl": 3600,
        "purpose": ""
      }
    ]
  },
  "request_id": "64abcf60-761e-4b66-8e86-aa509072b87f"
}
2026/09/15 09:19:52 Get record succeeded — record content verified

--------------------------------------------------------------------------------

2026/09/15 09:19:52 Deleting test-smoke-cd435a4 record
2026/09/15 09:19:52 --> DELETE https://***/api/v1/tenant/dns-api/zone/dns-api.jpw2x.dcnw.rakuten./record_type/Address/record_hostname/test-smoke-cd435a4?color=blue
Host: ***
Accept: application/json
Authorization: ***
Content-Type: application/json
body:
{}
2026/09/15 09:19:52 <-- 200 DELETE https://***/api/v1/tenant/dns-api/zone/dns-api.jpw2x.dcnw.rakuten./record_type/Address/record_hostname/test-smoke-cd435a4?color=blue
Content-Length: 226
Content-Type: application/json
Date: Tue, 15 Sep 2026 09:19:52 GMT
Strict-Transport-Security: max-age=31536000; includeSubDomains
Vary: Origin
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
X-Request-Id: ff51597d-7868-4de0-b108-f5740ccd03f4
body:
{
  "success": [
    "test-smoke-cd435a4.dns-api.jpw2x.dcnw.rakuten. -\u003e 100.124.66.153 deleted.",
    "153.66.124.100.in-addr.arpa. -\u003e test-smoke-cd435a4.dns-api.jpw2x.dcnw.rakuten. deleted."
  ],
  "request_id": "ff51597d-7868-4de0-b108-f5740ccd03f4"
}
2026/09/15 09:19:52 Delete record succeeded — record and PTR record removed

--------------------------------------------------------------------------------

2026/09/15 09:19:52 SMOKE TEST PASSED
```

## 16. Known Risks & Improvements

| # | Category | Status | Description |
|---|----------|:------:|--------------|
|  |  |  |  |

> **Additional Notes:** _Describe any further edge cases, known limitations, or improvement suggestions here._

---
