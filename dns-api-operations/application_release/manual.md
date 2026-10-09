# ⚙️ Release Operation Manual — DNS API Application Release

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
| **Summary**          | Release of a newer version of the DNS API to the DNS API `{{ENV}}` environment with schema incorporation (if applicable), data patches (if applicable) and image update |
| **Details**          | An upgraded version of the DNS API will be released to the DNS API `{{ENV}}` environment. It will consist of bug fixes and improvements. The process may or may not include schema incorporation and data patches, as not every release consists of schema changes/updates. |
| **Release Note — What's changed? (if applicable)** | `{{RELEASE_NOTES_LINK}}` |
| **GHE Tag Link — Application (if applicable)** | `{{APP_ROLLOUT_TAG_LINK}}` |
| **GHE Commit hash Link — Configuration (if applicable)** | `{{CONFIG_ROLLOUT_COMMIT_HASH_LINK}}` |

---

## 2. Classification & Risk Assessment

| Parameter                            | Value                 | Notes                                                           |
|--------------------------------------|-----------------------|-----------------------------------------------------------------|
| **Risk Level**                       | `MEDIUM`        | Procedure risk: `LOW`. Service risk: `MEDIUM`. |
| **Automation Level**                 | `2`          | All CHANGE steps are executed by the operator via GitHub Actions. |
| **GMS Impact Possible**              | `NO` | There is no impact on the Gross Merchandise Sales as this is an internal service. |
| **Data Loss or Corruption Possible** | `NO` | **Impact Status:** Minimal impact; **Downtime:** None; **Risk of Release (worst case):** Release Failure — assessed as low risk because (1) a standby cluster is present to take over if the active cluster goes down, and (2) the release is applied to blue or green but never both at once in the same operation. |
| **External Dependency**              | `NO`                  | Release to an Internal DNS service. No external dependencies. |

---

## 3. Deployment Strategy

- [ ] **Canary** — Route X% of traffic to new version before full rollout
- [x] **Blue/Green** — Maintain two identical environments; switch traffic
- [ ] **Night Time** — Execute during low-traffic window (specify time & TZ)
- [x] **Rolling Update** — Incrementally replace instances
- [ ] **Feature Flag** — Gate new behaviour behind a runtime toggle
- [ ] **Other** — Describe: ___________

> **Justification / Notes:**
> We choose blue-green releases so that we can perform:
> 1. Zero-downtime releases (by switching traffic between the two sides instantaneously)
> 2. Smoke testing on the inactive side (so that we can perform quality checks and testing without impacting users)

---

## 4. Variable Parameters

> ⚠️ All dynamic values **must** be defined here and separated from the procedure body. Variables use `{{PLACEHOLDER}}` notation throughout. "Example Value" is populated only where an example was given. **Validated** defaults to ❌ pending confirmation before execution.

| Variable Name | Description | Example Value | Validated (✅/❌) |
|---|---|---|:---:|
| `{{OPERATION_TICKET_JIRA_LINK}}` | Jira link to the operation ticket | https://jira.rakuten-it.com/jira/browse/CPSDCM-XXXX | ❌ |
| `{{DATE_TIME}}` | Date and time of the operation. Include the start and end time. | `2026-03-21 1400-1600 hrs JST` | ❌ |
| `{{RISK_LEVEL}}` | Risk level of the operation, taking into account service and operational risk | `MEDIUM` | ❌ |
| `{{USER_IMPACT}}` | Impact on existing users if the operation is performed | ✅(Explain)/❌ | ❌ |
| `{{OPERATOR_NAME}}` / `{{BUDDY_NAME}}` | Operator of operation and Buddy (checker) | `ar-srinivass01 / xulun01` | ❌ |
| `{{RELEASE_NOTES_LINK}}` | Link to the release notes of this release | https://confluence.rakuten-it.com/confluence/spaces/UCP/pages/6210496246/DNS+API+Release+Notes+v1.0.0 | ❌ |
| `{{ENV}}` | Environment for which this operation is being performed | `PROD` | ❌ |
| `{{ENV_DOMAIN}}` | Domain of the environment that can be used for API calls | `dns-api.r-local.net` | ❌ |
| `{{CONFIG_REPO_DOMAIN}}` | HTTPS Link needed to access the `apollo-automation` repo | https://github.com/Core-Resource-Group/apollo-automation.git | ❌ |
| `{{ACTIONS_WORKFLOW_TAG}}` | Short Commit hash of the tag whose GitHub Actions workflows would be run | `0eaae2c` | ❌ |
| `{{ACTIONS_WORKFLOW_TAG_LINK}}` | Link to the GHE tag whose GitHub Actions workflows would be run | https://github.com/Core-Resource-Group/apollo-automation/releases/tag/frozen-workflow-v1 | ❌ |
| `{{APP_ROLLOUT_TAG_LINK}}` | Link to the GHE tag for the application release | https://github.com/Core-Resource-Group/apollo/releases/tag/v1.1.0 | ❌ |
| `{{CONFIG_ROLLOUT_COMMIT_HASH}}` | Full Commit hash of the rollout tag in the `apollo-automation` repo | `182248ff3e0cd46d572d0306bb62e365ace489ed` | ❌ |
| `{{CONFIG_ROLLOUT_COMMIT_HASH_LINK}}` | Link to the GHE Commit hash for the configuration release | https://github.com/Core-Resource-Group/apollo-automation/commit/182248ff3e0cd46d572d0306bb62e365ace489ed | ❌ |
| `{{APP_ROLLOUT_VERSION}}` | `apollo` repo version to deploy to the clusters | `v1.1.0` | ❌ |
| `{{APP_ROLLOUT_COMMIT_HASH}}` | Short Commit hash of the rollout tag in the `apollo` repo | `cd435a4` | ❌ |
| `{{APP_ROLLBACK_VERSION}}` | Currently running version of the `apollo` repo before this release | `v1.0.0` | ❌ |
| `{{APP_ROLLBACK_COMMIT_HASH}}` | Short Commit hash of the rollback tag in the `apollo` repo | `01053dc` | ❌ |
| `{{CONFIG_ROLLBACK_COMMIT_HASH}}` | Full Commit hash of the rollout tag in the `apollo-automation` repo - To be used only for K8s deployment rollback. Not to be used for the rollback of data patches as the rollback patch files exist in `{{CONFIG_ROLLOUT_COMMIT_HASH}}` | `4c66ba7d0f756be0436ffeb6c0b88ab439fe6930` | ❌ |
| `{{IS_SCHEMA_MIGRATION_NEEDED}}` | To be selected if any schema changes have been made | ✅/❌ | ❌ |
| `{{SCHEMA_MIGRATION_FILE_COUNT_AS_IS}}` | Number of migrations files that have been migrated already - To be passed if any schema changes have been made | 7 | ❌ |
| `{{SCHEMA_MIGRATION_FILE_COUNT_TO_BE}}` | Number of migrations files that are to be migrated - To be passed if any schema changes have been made | 1 | ❌ |
| `{{IS_DATA_PATCH_NEEDED}}` | To be selected if any data patches need to be made | ✅/❌ | ❌ |
| `{{IS_IMAGE_UPDATE_NEEDED}}` | To be selected if the deployment images are going to be updated | ✅/❌ | ❌ |
| `{{IS_TRAFFIC_REROUTE_NEEDED}}` | To be selected if the traffic is to be switched to the color that was released to after release - Present because we may release a version to one side and route traffic to it. Later on, we may release the same version to the other side as part of another operation to get the versions synced across sides. This operation does not require a traffic reroute. This field is for such a case. | ✅/❌ | ❌ |
| `{{AZ_1}}, {{AZ_2}}, etc.` | Availability zones to be whitelisted — AZ to be added to the list of supported domains | `euc1a,jpc1a` | ❌ |
| `{{DB_PATCH_FILE_PATH}}` | Path of the DB patch file relative to the `apollo-automation` repo root | `operations/rollout.cql` | ❌ |
| `{{DB_ROLLBACK_PATCH_FILE_PATH}}` | Path of the DB rollback patch file relative to the `apollo-automation` repo root | `operations/rollback.cql` | ❌ |
| `{{ACTIVE_CLUSTER}}` (Active) | Active CaaS Cluster to deploy to | `jpe1-caas1-prod2` | ❌ |
| `{{STANDBY_CLUSTER}}` (Standby) | Standby CaaS Cluster to deploy to | `jpe2-caas1-prod2` | ❌ |
| `{{ARE_COMMON_RESOURCES_TO_BE_APPLIED}}` | GitHub Actions — check this if the common resources need to be applied (Consul, secrets, network policies, etc. — non-app, non-virtual-service resources) | ✅/❌ | ❌ |
| `{{ACTIVE_CLUSTER_ACTIVE_COLOR}}` | The color of the side currently serving traffic on the Active cluster. Determined in Step 6D of Section 6. | `blue/green` | ❌ |
| `{{ACTIVE_CLUSTER_INACTIVE_COLOR}}` | The color of the side currently not serving traffic on the Active cluster. Determined in Step 6D of Section 6. | `blue/green` | ❌ |
| `{{STANDBY_CLUSTER_ACTIVE_COLOR}}` | The color of the side currently serving traffic on the Standby cluster. Determined in Step 6E of Section 6. | `blue/green` | ❌ |
| `{{STANDBY_CLUSTER_INACTIVE_COLOR}}` | The color of the side currently not serving traffic on the Standby cluster. Determined in Step 6E of Section 6. | `blue/green` | ❌ |
| `{{CASSANDRA_AZ_1}}` | First AZ of the Cassandra Cluster | `jpe2b` | ❌ |
| `{{CASSANDRA_AZ_1_FQDN_1}}` | FQDN of the first node in the first AZ of the Cassandra cluster | Resembling `omnia-apollo-cassandra-prod-012.dbaas.jpe2b.dcnw.rakuten` | ❌ |
| `{{CASSANDRA_AZ_1_FQDN_2}}` | FQDN of the second node in the first AZ of the Cassandra cluster | Resembling `omnia-apollo-cassandra-prod-012.dbaas.jpe2b.dcnw.rakuten` | ❌ |
| `{{CASSANDRA_AZ_1_FQDN_3}}` | FQDN of the third node in the first AZ of the Cassandra cluster | Resembling `omnia-apollo-cassandra-prod-012.dbaas.jpe2b.dcnw.rakuten` | ❌ |
| `{{CASSANDRA_AZ_2}}` | Second AZ of the Cassandra Cluster |  `jpw2a` | ❌ |
| `{{CASSANDRA_AZ_2_FQDN_1}}` | FQDN of the first node in the second AZ of the Cassandra cluster | Resembling `omnia-apollo-cassandra-prod-012.dbaas.jpe2b.dcnw.rakuten` | ❌ |
| `{{CASSANDRA_AZ_2_FQDN_2}}` | FQDN of the second node in the second AZ of the Cassandra cluster | Resembling `omnia-apollo-cassandra-prod-012.dbaas.jpe2b.dcnw.rakuten` | ❌ |
| `{{CASSANDRA_AZ_2_FQDN_3}}` | FQDN of the third node in the second AZ of the Cassandra cluster | Resembling `omnia-apollo-cassandra-prod-012.dbaas.jpe2b.dcnw.rakuten` | ❌ |
| `{{ACTIVE_CLUSTER_HARBOR}}` (Harbor for Active) | Harbor Registry holding the images used by the `dns-api-apollo` namespace in the Active cluster | `registry-jpe1.r-local.net` | ❌ |
| `{{STANDBY_CLUSTER_HARBOR}}` (Harbor for Standby) | Harbor Registry holding the images used by the `dns-api-apollo` namespace in the Standby cluster | `registry-jpe2.r-local.net` | ❌ |
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
The CLSD Core Resource Group is performing a DNS API release in the {{ENV}} Environment

Please refer to the below for details.
[Operation Announcement]
================================================
■ Schedule
{{DATE_TIME}}

■ Outline
Release a new version of the DNS API to the DNS API {{ENV}} environment

{{OPERATION_TICKET_JIRA_LINK}}

■ Purpose
To incorporate new bug fixes and enhancements, data patches, version upgrades and so on.

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

### Step 6A: Pre validation - A

**Action type:** CHECK
**Why:** Ensure that the Image Manifests of the form `{{APP_ROLLOUT_VERSION}}`-`{{APP_ROLLOUT_COMMIT_HASH}}`-`{{ENV}}` exist on `{{ACTIVE_CLUSTER_HARBOR}}` and `{{STANDBY_CLUSTER_HARBOR}}` under `dns-api-apollo/app`.

**Appearance:**

![Harbor UI](https://confluence.rakuten-it.com/confluence/download/attachments/6857865109/harbor.png)

### Step 6B: Pre validation - B

**Action type:** CHECK
**Why:** Ensure the application is up and whether the operator has the necessary permissions to perform this operation.

- Check if the Environment's Swagger is accessible.

Access the Swagger of the environment through this link: `https://{{ENV_DOMAIN}}/swagger/index.html#`. It should load.


### Step 6C: Pre validation - C

**Action type:** CHECK
**Why:** Ensure all the necessary tools and files are available

**kubectl**

On the device being used to monitor the pods, run:
```bash
kubectl version
```
You should see something like this (Sample output. May differ slightly from what you see):
```bash
Client Version: v1.36.1
Kustomize Version: v5.8.1
Server Version: v1.20.12
```

**roc**

On the device being used to monitor the pods, run:
```bash
roc version
```
You should see something like this (Sample output. May differ slightly from what you see):
```bash
roc version: 1.3.3
your version is latest version of roc-cli.
```

### Step 6D — Active Cluster Active Color Determination: Open the "View Istio VirtualService Weights on PROD, DEV or LAB" GitHub Actions Workflow

> **Action type:** CHECK
>
> **Why:** The current weights in the Active cluster need to be checked so that we could release to the side that is currently inactive. This step lets us view the weights on both sides.
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

![GitHub Actions View Virtual Service Workflow Input](https://confluence.rakuten-it.com/confluence/download/attachments/6857865109/virtualserviceget.png)

Click "Run Workflow" and set:
- **Use workflow from:** Tag `{{ACTIONS_WORKFLOW_TAG}}`
- **Cluster whose VirtualService weights you want to view:** `{{ACTIVE_CLUSTER}}`

**Expected Result:**

```
blue=**
green=**
```

**Validation:** One side shows 100% and the other 0%.

> **Convention:** the side currently at 100% traffic is called `{{ACTIVE_CLUSTER_ACTIVE_COLOR}}`; the side at 0% traffic is called `{{ACTIVE_CLUSTER_INACTIVE_COLOR}}`.

### Step 6E — Standby Cluster Active Color Determination: Open the "View Istio VirtualService Weights on PROD, DEV or LAB" GitHub Actions Workflow

> **Action type:** CHECK
>
> **Why:** The current weights in the Standby cluster need to be checked so that we could release to the side that is currently inactive. This step lets us view the weights on both sides.
>
> **Scope:** Read-only check of current Istio VirtualService traffic weights on `{{STANDBY_CLUSTER}}`. No changes made.
>
> **Abortion:** ✅ Safe to abort — this step makes no changes.
>
> **Automation Level:** Pipeline (GitHub Actions Workflow)

Link: `{{CONFIG_REPO_DOMAIN}}/actions/workflows/view-virtualservice-weights.yml`

The below workflow with the passed values includes the following steps:
1. Set up of `kubectl`
2. Viewing and Parsing of the Istio VirtualService resource's details

**Appearance:**

![GitHub Actions View Virtual Service Workflow Input](https://confluence.rakuten-it.com/confluence/download/attachments/6857865109/virtualserviceget.png)

Click "Run Workflow" and set:
- **Use workflow from:** Tag `{{ACTIONS_WORKFLOW_TAG}}`
- **Cluster whose VirtualService weights you want to view:** `{{STANDBY_CLUSTER}}`

**Expected Result:**

```
blue=**
green=**
```

**Validation:** One side shows 100% and the other 0%.

> **Convention:** the side currently at 100% traffic is called `{{STANDBY_CLUSTER_ACTIVE_COLOR}}`; the side at 0% traffic is called `{{STANDBY_CLUSTER_INACTIVE_COLOR}}`.


All checks above must pass before proceeding to Section 8.

---

## 7. Login & Access Procedures

### 7.1 Required Access & Permissions

| System / Tool | Access Method | Permission Level | Notes |
|----------------|----------------|-------------------|-------|
| CaaS Cluster | kubectl | admin | User should be a part of the `dns-api` tenant on OneCloud and should have the required CaaS role. |
| Harbor Registry | Web UI | viewer | User should be a part of the `dns-api` tenant on OneCloud and should have the required Registry-aaS role. |
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

- [ ] Application Down
- [ ] Pods are not coming up even after repeated attempts
- [ ] Smoke tests failed
- [ ] Success Rate of the Application goes below 90%

> **On abortion:**
> - If rollback is needed, in most cases, the operator and buddy should judge whether a rollback/abortion is warranted.
> - If rollback is needed and the operator and buddy are not able to judge whether a rollback/abortion is warranted or if there is the risk of impact to other systems, contact L4 and L3 to determine which rollback steps are required.
> - If the rollback procedure does not result in full recovery, contact L3 and L2 to decide next actions.

---

## 11. Operations Procedure

### Step 1 — Schema Incorporation *(only if `{{IS_SCHEMA_MIGRATION_NEEDED}}` is checked)*

#### Step 1.1 - Schema Incorporation - Dry run

> **Action type:** CHECK
>
> **Why:** See if the expected number of files to be migrated matches what the cluster is expecting.
>
> **Scope:** Generates the list of past migrations and pending migrations.
>
> **Abortion:** ✅ Safe to abort — this step makes no changes.
>
> **Automation Level:** Pipeline (GitHub Actions Workflow)


Link: `{{CONFIG_REPO_DOMAIN}}/actions/workflows/cql-schema-migrate.yml`

The below workflow with the passed values includes the following steps:
1. Checkout of the passed commit hash
2. Restoration of the git-crypt symmetric key and unlocking of the repository
3. Validation of the target version
4. Resolution of the cqltrack profile
5. Display of the current migration status
6. Dry-run of the migration to show what will be done
7. Locking of the repository

**Appearance:**

![GitHub Actions Schema Incorporation Workflow Input](https://confluence.rakuten-it.com/confluence/download/attachments/6857865109/schemaincorporation.png)

Click "Run Workflow" and set:
- **Use workflow from:** Tag `{{ACTIONS_WORKFLOW_TAG}}`
- **Target environment:** `{{ENV}}`
- **Full Configuration commit hash to check out (blank = default branch/tag):** `{{CONFIG_ROLLOUT_COMMIT_HASH}}`
- **Stop at this migration version, e.g. 5 for V005 (blank = migrate to latest):** *(blank)*
- **Show pending migrations without applying them:** ✅

**Expected Result:** The GitHub Actions workflow run completes successfully. A sample of the output is shown below:
```
V001   applied  init                            2026-09-17 11:20:40
V002   applied  bangapi                         2026-09-17 11:20:40
V003   applied  workflowreplace                 2026-09-17 11:20:40
V004   applied  auditmodify                     2026-09-17 11:20:40
V005   applied  addmaintenancemode              2026-09-17 11:20:40
V006   applied  addtalaria                      2026-09-17 11:20:40
V007   applied  addpowerdnssoaconfig            2026-09-17 11:20:40
V008   pending  test 
```
**Validation:** Confirm that the number of migrations marked `applied` in the output equals `{{SCHEMA_MIGRATION_FILE_COUNT_AS_IS}}` and that the number of migrations marked `pending` in the output equals `{{SCHEMA_MIGRATION_FILE_COUNT_TO_BE}}`

| ✅ Condition to proceed to the next step | ⛔ Action if condition not met |
|---|---|
| The GitHub Actions workflow completes without error, the number of migrations marked `applied` in the output equals `{{SCHEMA_MIGRATION_FILE_COUNT_AS_IS}}` and the number of migrations marked `pending` in the output equals `{{SCHEMA_MIGRATION_FILE_COUNT_TO_BE}}` | Escalate per Section 10 as there seems to be a cluster migration mismatch. Do not proceed. |


#### Step 1.2 - Schema Incorporation - Incorporation

> **Action type:** CHANGE
>
> **Why:** New Schema needs to be incorporated.
>
> **Scope:** Applies a schema migration to the shared, multi-AZ Cassandra DB used by both `{{ACTIVE_CLUSTER}}` and `{{STANDBY_CLUSTER}}`. No traffic shift and no pods are restarted by this step alone.
>
> **Abortion:** ⚠️ Not trivially reversible — the migration affects the shared database used by both clusters. If this must be undone, use the rollback schema incorporation procedure (Section 12.2) and escalate per Section 10.
>
> **Automation Level:** Pipeline (GitHub Actions Workflow)


Link: `{{CONFIG_REPO_DOMAIN}}/actions/workflows/cql-schema-migrate.yml`

The below workflow with the passed values includes the following steps:
1. Checkout of the passed commit hash
2. Restoration of the git-crypt symmetric key and unlocking of the repository
3. Validation of the target version
4. Resolution of the cqltrack profile
5. Display of the current migration status
6. Schema migration/incorporation
7. Locking of the repository

**Appearance:**

![GitHub Actions Schema Incorporation Workflow Input](https://confluence.rakuten-it.com/confluence/download/attachments/6857865109/schemaincorporation.png)

Click "Run Workflow" and set:
- **Use workflow from:** Tag `{{ACTIONS_WORKFLOW_TAG}}`
- **Target environment:** `{{ENV}}`
- **Full Configuration commit hash to check out (blank = default branch/tag):** `{{CONFIG_ROLLOUT_COMMIT_HASH}}`
- **Stop at this migration version, e.g. 5 for V005 (blank = migrate to latest):** *(blank)*
- **Show pending migrations without applying them:** ❌

**Expected Result:** The GitHub Actions workflow run completes successfully. A sample of the output is shown below:
```bash
Applying V008  test ... OK (10430ms)

Applied 1 migration(s).
```
**Validation:** Confirm the command completed with no error output.

| ✅ Condition to proceed to the next step | ⛔ Action if condition not met |
|---|---|
| The GitHub Actions workflow completes without error | Investigate possible network connectivity issues and escalate per Section 10 based on the findings. |

---

### Step 2 — Data Patch Incorporation *(only if `{{IS_DATA_PATCH_NEEDED}}` is checked)*

> **Action type:** CHANGE
>
> **Why:** New data needs to be added.
>
> **Scope:** Applies a CQL patch file to the shared Cassandra DB via `cqlsh` within the workflow. No traffic shift and no pods are restarted by this step alone.
>
> **Abortion:** ⚠️ Not trivially reversible — reverse via the rollback patch procedure (Section 12.2) if necessary; escalate per Section 10.
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

![GitHub Actions Data Patch Workflow Input](https://confluence.rakuten-it.com/confluence/download/attachments/6857865109/datapatch.png)

Click "Run Workflow" and set:
- **Use workflow from:** Tag `{{ACTIONS_WORKFLOW_TAG}}`
- **Target environment:** `{{ENV}}`
- **Full Configuration commit hash to check out (blank = default branch/tag):** `{{CONFIG_ROLLOUT_COMMIT_HASH}}`
- **Path to the CQL file to apply, relative to the repository root:** `{{DB_PATCH_FILE_PATH}}`

**Expected Result:** Success.
**Validation:** Confirm the command completed with no error output on the GH Actions Console.

| ✅ Condition to proceed to the next step | ⛔ Action if condition not met |
|---|---|
| Patch applied without error | Do not proceed with a partially-applied patch — investigate the failure by checking the contents of the `{{DB_PATCH_FILE_PATH}}` in the repo and escalate per Section 10 if needed. |

---

### Step 3 — Standby Cluster → `{{STANDBY_CLUSTER_INACTIVE_COLOR}}` Side: Hit the "Deploy the DNS API in a LAB, DEV or PROD cluster" GitHub Actions Workflow *(only if `{{IS_IMAGE_UPDATE_NEEDED}}` is checked)*

> **Action type:** CHANGE
>
> **Why:** Deploy the version to the standby cluster.
>
> **Scope:** Deploys `{{APP_ROLLOUT_VERSION}}` to `{{STANDBY_CLUSTER_INACTIVE_COLOR}}` in `{{STANDBY_CLUSTER}}`. `{{STANDBY_CLUSTER}}` carries no live user traffic under normal operation.
>
> **Abortion:** ✅ Safe to abort.
>
> **Automation Level:** Pipeline (GitHub Actions Workflow)

Link: `{{CONFIG_REPO_DOMAIN}}/actions/workflows/deploy.yml`

The below workflow with the passed values includes the following steps:
1. Checkout of the passed commit hash
2. Restoration of the git-crypt symmetric key and unlocking of the repository
3. Mapping of the cluster to the directory where the manifests reside.
4. Set up of `kubectl`
5. Application of consul resources, network policies, dedicated gateway resources, database credential secrets, HPA configuration, filebeat configuration, domain claim resources and load balancer resources if opted for.
6. Application of app-blue/app-green deployments as applicable.
7. Locking of the repository

**Appearance:**

![GitHub Actions Deployment Workflow Input](https://confluence.rakuten-it.com/confluence/download/attachments/6857865109/deployment.png)

Click "Run Workflow" and set:
- **Use workflow from:** Tag `{{ACTIONS_WORKFLOW_TAG}}`
- **Cluster to deploy to:** `{{STANDBY_CLUSTER}}`
- **Full Configuration commit hash to check out (blank = default branch/tag):** `{{CONFIG_ROLLOUT_COMMIT_HASH}}`
- **Check this if the common resources need to be applied:** `{{ARE_COMMON_RESOURCES_TO_BE_APPLIED}}`
- **Check this if the app-blue resources need to be applied:** ✅ if `{{STANDBY_CLUSTER_ACTIVE_COLOR}}`=green and `{{STANDBY_CLUSTER_INACTIVE_COLOR}}`=blue; ❌ if `{{STANDBY_CLUSTER_ACTIVE_COLOR}}`=blue and `{{STANDBY_CLUSTER_INACTIVE_COLOR}}`=green
- **Check this if the app-green resources need to be applied:** ✅ if `{{STANDBY_CLUSTER_ACTIVE_COLOR}}`=blue and `{{STANDBY_CLUSTER_INACTIVE_COLOR}}`=green; ❌ if `{{STANDBY_CLUSTER_ACTIVE_COLOR}}`=green and `{{STANDBY_CLUSTER_INACTIVE_COLOR}}`=blue
- **Application Release version:** `{{APP_ROLLOUT_VERSION}}`
- **Short Application Commit hash:** `{{APP_ROLLOUT_COMMIT_HASH}}`
- **Check this if you want traffic to be transferred to the blue/green sides:** ❌
- **Percentage of traffic to route to the blue deployment:** `100` (irrelevant — traffic transfer checkbox not checked)
- **Percentage of traffic to route to the green deployment:** `0` (irrelevant — traffic transfer checkbox not checked)

**Expected Result:** The GitHub Actions workflow run completes successfully.
**Validation:** See Step 4 for pod-level rollout validation.

| ✅ Condition to proceed to the next step | ⛔ Action if condition not met |
|---|---|
| Workflow run completes successfully | Workflow fails — do not proceed; investigate before retrying. |

---

### Step 4 — Standby Cluster → `{{STANDBY_CLUSTER_INACTIVE_COLOR}}` Side: Validation of rollout in the Standby cluster *(only if `{{IS_IMAGE_UPDATE_NEEDED}}` is checked)*

> **Action type:** CHECK
>
> **Why:** Check that all pods are up on the `{{STANDBY_CLUSTER_INACTIVE_COLOR}}` side in the standby cluster.
>
> **Scope:** Read-only check of pod status on `{{STANDBY_CLUSTER_INACTIVE_COLOR}}` in `{{STANDBY_CLUSTER}}`. No changes made.
>
> **Abortion:** ✅ Safe to abort.
>
> **Automation Level:** Manual (CLI: `kubectl`)

```bash
roc login -c {{STANDBY_CLUSTER}} -n dns-api-apollo
```

Then, run

```bash
kubectl get pods
```

**Expected Result:**

```
NAME                                   READY   STATUS    RESTARTS   AGE
apollo-consul-0                        2/2     Running   0          103d
apollo-consul-1                        2/2     Running   0          103d
apollo-consul-2                        2/2     Running   0          103d
app-blue-7c4849585b-5v9mh              3/3     Running   0          251d
app-blue-7c4849585b-lcp6z              3/3     Running   0          251d
app-blue-7c4849585b-nvxg5              3/3     Running   0          251d
app-green-864d9d65b4-dcv47             3/3     Running   0          251d
app-green-864d9d65b4-fjk6q             3/3     Running   0          251d
app-green-864d9d65b4-r8h8k             3/3     Running   0          251d
gateway-app-gateway-86598f8c96-9xcjv   1/1     Running   0          345d
gateway-app-gateway-86598f8c96-c4nct   1/1     Running   0          345d
gateway-app-gateway-86598f8c96-f6vjk   1/1     Running   0          345d
```

**Validation:** The `app-{{STANDBY_CLUSTER_INACTIVE_COLOR}}` deployment pods should be new and in `Running` state.

| ✅ Condition to proceed to the next step | ⛔ Action if condition not met |
|---|---|
| All `app-{{STANDBY_CLUSTER_INACTIVE_COLOR}}` pods show `Running` with 0 restarts | Pods unhealthy — do not proceed; investigate. |

---

### Step 5 — Standby Cluster → `{{STANDBY_CLUSTER_INACTIVE_COLOR}}` Side: Smoke Testing

> **Action type:** CHANGE
>
> **Why:** Ensure that the API functionalities are as expected on the `{{STANDBY_CLUSTER_INACTIVE_COLOR}}` side.
>
> **Scope:** Creates, patches, reads and deletes a single test DNS record on the `{{STANDBY_CLUSTER_INACTIVE_COLOR}}` side via the DNS API. Confined to one test hostname; no live user records are affected.
>
> **Abortion:** ✅ Safe to abort — this is a dummy record under the passed tenant and does not impact any service.
>
> **Automation Level:** Pipeline (GitHub Actions Workflow)

**Legend**

- **`{{TENANT}}`:** A valid OneCloud tenant
- **`{{ZONE}}`:** The DNS zone under which we wish to create records. Format: `{{TENANT}}.{{AZ_1}}/{{AZ_2}}/....dcnw.rakuten.`.
- **`{{VALID_IP_LINKED_TO_PASSED_TENANT_IN_THE_PASSED_AZ}}`:** IP from a subnet assigned to the `{{TENANT}}` in `{{AZ_1}}/{{AZ_2}}/...`.
- **`{{ANOTHER_VALID_IP_LINKED_TO_PASSED_TENANT_IN_THE_PASSED_AZ}}`:** Another IP from a subnet assigned to the `{{TENANT}}` in `{{AZ_1}}/{{AZ_2}}/...`.

Link: `{{CONFIG_REPO_DOMAIN}}/actions/workflows/smoke-test-dns-api.yml`

The below workflow with the passed values includes the following steps:
1. Checkout of the repository
2. Set up of the Go GitHub Action
3. Disallowance of smoke tests on the standby side if the target environment is either QA or LAB (as they do not have the standby side)
4. Execution of the smoke test with the passed parameters via a Go script (includes create record, update record, get record and delete record)

**Appearance:**

![GitHub Actions Smoke Test Workflow Input](https://confluence.rakuten-it.com/confluence/download/attachments/6857865109/smoketest.png)

Click "Run Workflow" and set:
- **Use workflow from:** Tag `{{ACTIONS_WORKFLOW_TAG}}`
- **Target Environment:** `{{ENV}}`
- **Cluster to smoke test against (QA and LAB have no Standby cluster — use ACTIVE there):** `STANDBY`
- **Side to smoke test against (Color - blue or green) (choose either if the target is the QA environment as this value is irrelevant there):** `{{STANDBY_CLUSTER_INACTIVE_COLOR}}`
- **OneCloud tenant to smoke test against:** `{{TENANT}}`
- **DNS zone to create the record under, e.g. <tenant>.<az>.dcnw.rakuten., <subdomain>.jp.local., etc.:** `{{ZONE}}`
- **Record type:** `Address`
- **Record hostname:** `test-smoke-{{APP_ROLLOUT_COMMIT_HASH}}`
- **Content to create the record with, e.g. a single IP address for an Address record and a canonical name for a CNAME record:** `{{VALID_IP_LINKED_TO_PASSED_TENANT_IN_THE_PASSED_AZ}}`
- **Different content to patch the record to:** `{{ANOTHER_VALID_IP_LINKED_TO_PASSED_TENANT_IN_THE_PASSED_AZ}}`

**Expected Result:** All four smoke-test calls succeed. A sample of the output can be found in Section 15.1.
**Validation:** You will see success (HTTP Status Code 200) in all 4 API calls with POST, PATCH and DELETE have 2, 3 and 2 entries in the `success` field of the response.


| ✅ Condition to proceed to the next step | ⛔ Action if condition not met |
|---|---|
| All four smoke-test calls return the results described above | Any call fails, or logs an unexpected error — abort per Section 10; do not proceed to the traffic cutover in Step 6. |

---

### Step 6 — Standby Cluster: Hit the deploy workflow to shift traffic to the `{{STANDBY_CLUSTER_INACTIVE_COLOR}}` side *(only if `{{IS_TRAFFIC_REROUTE_NEEDED}}` and `{{IS_IMAGE_UPDATE_NEEDED}}` are checked)*

> **Action type:** CHANGE
>
> **Why:** Switch the traffic to the `{{STANDBY_CLUSTER_INACTIVE_COLOR}}` side so that users may start using the new version on the `{{STANDBY_CLUSTER_INACTIVE_COLOR}}` side
>
> **Scope:** Shifts internal routing on `{{STANDBY_CLUSTER}}` to `{{STANDBY_CLUSTER_INACTIVE_COLOR}}` from `{{STANDBY_CLUSTER_ACTIVE_COLOR}}`.
>
> **Abortion:** ✅ Safe to abort/revert — shift back to `{{STANDBY_CLUSTER_ACTIVE_COLOR}}` if needed.
>
> **Automation Level:** Pipeline (GitHub Actions Workflow)

Link: `{{CONFIG_REPO_DOMAIN}}/actions/workflows/deploy.yml`

The below workflow with the passed values includes the following steps:
1. Checkout of the passed commit hash
2. Restoration of the git-crypt symmetric key and unlocking of the repository
3. Mapping of the cluster to the directory where the manifests reside.
4. Set up of `kubectl`
5. Blue-Green Traffic routing switch by re-applying the Istio VirtualService
6. Locking of the repository

**Appearance:**

![GitHub Actions Deployment Workflow Input](https://confluence.rakuten-it.com/confluence/download/attachments/6857865109/deployment.png)

Click "Run Workflow" and set:
- **Use workflow from:** Tag `{{ACTIONS_WORKFLOW_TAG}}`
- **Cluster to deploy to:** `{{STANDBY_CLUSTER}}`
- **Full Configuration commit hash to check out (blank = default branch/tag):** `{{CONFIG_ROLLOUT_COMMIT_HASH}}`
- **Check this if the common resources need to be applied:** ❌
- **Check this if the app-blue resources need to be applied:** ❌
- **Check this if the app-green resources need to be applied:** ❌
- **Application Release version:** *(blank)*
- **Short Application Commit hash:** *(blank)*
- **Check this if you want traffic to be transferred to the blue/green sides:** ✅
- **Percentage of traffic to route to the blue deployment:** `100` if `{{STANDBY_CLUSTER_INACTIVE_COLOR}}`=blue and `{{STANDBY_CLUSTER_ACTIVE_COLOR}}`=green; `0` if `{{STANDBY_CLUSTER_INACTIVE_COLOR}}`=green and `{{STANDBY_CLUSTER_ACTIVE_COLOR}}`=blue
- **Percentage of traffic to route to the green deployment:** `0` if `{{STANDBY_CLUSTER_INACTIVE_COLOR}}`=blue and `{{STANDBY_CLUSTER_ACTIVE_COLOR}}`=green; `100` if `{{STANDBY_CLUSTER_INACTIVE_COLOR}}`=green and `{{STANDBY_CLUSTER_ACTIVE_COLOR}}`=blue

**Expected Result:** The workflow run completes; VirtualService weights on `{{STANDBY_CLUSTER}}` now show `{{STANDBY_CLUSTER_INACTIVE_COLOR}}`=100%, `{{STANDBY_CLUSTER_ACTIVE_COLOR}}`=0%.
**Validation:** Re-run the "View Istio VirtualService Weights" workflow (Step 6E of Section 6) against `{{STANDBY_CLUSTER}}` and confirm `{{STANDBY_CLUSTER_INACTIVE_COLOR}}` is now at 100%.

| ✅ Condition to proceed to the next step | ⛔ Action if condition not met |
|---|---|
| Workflow run completes successfully and weights confirm the shift | Workflow fails — do not proceed; investigate before retrying. |

---

### Step 7 — Active Cluster → `{{ACTIVE_CLUSTER_INACTIVE_COLOR}}` Side: Hit the "Deploy the DNS API in a LAB, DEV or PROD cluster" GitHub Actions Workflow *(only if `{{IS_IMAGE_UPDATE_NEEDED}}` is checked)*

> **Action type:** CHANGE
>
> **Why:** Deploy the version to the active cluster.
>
> **Scope:** Deploys `{{APP_ROLLOUT_VERSION}}` to the `{{ACTIVE_CLUSTER_INACTIVE_COLOR}}` side of `{{ACTIVE_CLUSTER}}`, currently at 0% traffic. No live user traffic is affected.
>
> **Abortion:** ✅ Safe to abort — `{{ACTIVE_CLUSTER_INACTIVE_COLOR}}` is not yet serving live traffic.
>
> **Automation Level:** Pipeline (GitHub Actions Workflow)

Link: `{{CONFIG_REPO_DOMAIN}}/actions/workflows/deploy.yml`

The below workflow with the passed values includes the following steps:
1. Checkout of the passed commit hash
2. Restoration of the git-crypt symmetric key and unlocking of the repository
3. Mapping of the cluster to the directory where the manifests reside.
4. Set up of `kubectl`
5. Application of consul resources, network policies, dedicated gateway resources, database credential secrets, HPA configuration, filebeat configuration, domain claim resources and load balancer resources if opted for.
6. Application of app-blue/app-green deployments as applicable.
7. Locking of the repository

**Appearance:**

![GitHub Actions Deployment Workflow Input](https://confluence.rakuten-it.com/confluence/download/attachments/6857865109/deployment.png)

Click "Run Workflow" and set:
- **Use workflow from:** Tag `{{ACTIONS_WORKFLOW_TAG}}`
- **Cluster to deploy to:** `{{ACTIVE_CLUSTER}}`
- **Full Configuration commit hash to check out (blank = default branch/tag):** `{{CONFIG_ROLLOUT_COMMIT_HASH}}`
- **Check this if the common resources need to be applied:** `{{ARE_COMMON_RESOURCES_TO_BE_APPLIED}}`
- **Check this if the app-blue resources need to be applied:** ✅ if `{{ACTIVE_CLUSTER_ACTIVE_COLOR}}`=green and `{{ACTIVE_CLUSTER_INACTIVE_COLOR}}`=blue; ❌ if `{{ACTIVE_CLUSTER_ACTIVE_COLOR}}`=blue and `{{ACTIVE_CLUSTER_INACTIVE_COLOR}}`=green
- **Check this if the app-green resources need to be applied:** ✅ if `{{ACTIVE_CLUSTER_ACTIVE_COLOR}}`=blue and `{{ACTIVE_CLUSTER_INACTIVE_COLOR}}`=green; ❌ if `{{ACTIVE_CLUSTER_ACTIVE_COLOR}}`=green and `{{ACTIVE_CLUSTER_INACTIVE_COLOR}}`=blue
- **Application Release version:** `{{APP_ROLLOUT_VERSION}}`
- **Short Application Commit hash:** `{{APP_ROLLOUT_COMMIT_HASH}}`
- **Check this if you want traffic to be transferred to the blue/green sides:** ❌
- **Percentage of traffic to route to the blue deployment:** `100` (irrelevant — traffic transfer checkbox not checked)
- **Percentage of traffic to route to the green deployment:** `0` (irrelevant — traffic transfer checkbox not checked)

**Expected Result:** The GitHub Actions workflow run completes successfully.
**Validation:** See Step 8 for pod-level rollout validation.

| ✅ Condition to proceed to the next step | ⛔ Action if condition not met |
|---|---|
| Workflow run completes successfully | Workflow fails — do not proceed; investigate the failed run before retrying. |

---

### Step 8 — Active Cluster → `{{ACTIVE_CLUSTER_INACTIVE_COLOR}}` Side: Validation of rollout in the Active cluster *(only if `{{IS_IMAGE_UPDATE_NEEDED}}` is checked)*

> **Action type:** CHECK
>
> **Why:** Check that all pods are up on the `{{ACTIVE_CLUSTER_INACTIVE_COLOR}}` side in the active cluster.
>
> **Scope:** Read-only check of pod status on `{{ACTIVE_CLUSTER_INACTIVE_COLOR}}` in `{{ACTIVE_CLUSTER}}`. No changes made.
>
> **Abortion:** ✅ Safe to abort.
>
> **Automation Level:** Manual (CLI: `kubectl`)

```bash
roc login -c {{ACTIVE_CLUSTER}} -n dns-api-apollo
```

Then, run

```bash
kubectl get pods
```

**Expected Result:**

```
NAME                                   READY   STATUS    RESTARTS   AGE
apollo-consul-0                        2/2     Running   0          103d
apollo-consul-1                        2/2     Running   0          103d
apollo-consul-2                        2/2     Running   0          103d
app-blue-7c4849585b-5v9mh              3/3     Running   0          251d
app-blue-7c4849585b-lcp6z              3/3     Running   0          251d
app-blue-7c4849585b-nvxg5              3/3     Running   0          251d
app-green-864d9d65b4-dcv47             3/3     Running   0          251d
app-green-864d9d65b4-fjk6q             3/3     Running   0          251d
app-green-864d9d65b4-r8h8k             3/3     Running   0          251d
gateway-app-gateway-86598f8c96-9xcjv   1/1     Running   0          345d
gateway-app-gateway-86598f8c96-c4nct   1/1     Running   0          345d
gateway-app-gateway-86598f8c96-f6vjk   1/1     Running   0          345d
```

**Validation:** The `app-{{ACTIVE_CLUSTER_INACTIVE_COLOR}}` deployment pods should be new and in `Running` state.

| ✅ Condition to proceed to the next step | ⛔ Action if condition not met |
|---|---|
| All `app-{{ACTIVE_CLUSTER_INACTIVE_COLOR}}` pods show `Running` with 0 restarts | Pods are `CrashLoopBackOff`, `Error`, or not `Running` after repeated checks — abort per Section 10; do not proceed. |

---

### Step 9 — Active Cluster → `{{ACTIVE_CLUSTER_INACTIVE_COLOR}}` Side: Smoke Testing

> **Action type:** CHANGE
>
> **Why:** Ensure that the API functionalities are as expected on the `{{ACTIVE_CLUSTER_INACTIVE_COLOR}}` side.
>
> **Scope:** Creates, patches, reads and deletes a single test DNS record on the `{{ACTIVE_CLUSTER_INACTIVE_COLOR}}` side via the DNS API. Confined to one test hostname; no live user records are affected.
>
> **Abortion:** ✅ Safe to abort — this is a dummy record under the passed tenant and does not impact any service.
>
> **Automation Level:** Pipeline (GitHub Actions Workflow)

**Legend**

- **`{{TENANT}}`:** A valid OneCloud tenant
- **`{{ZONE}}`:** The DNS zone under which we wish to create records. Format: `{{TENANT}}.{{AZ_1}}/{{AZ_2}}/....dcnw.rakuten.`.
- **`{{VALID_IP_LINKED_TO_PASSED_TENANT_IN_THE_PASSED_AZ}}`:** IP from a subnet assigned to the `{{TENANT}}` in `{{AZ_1}}/{{AZ_2}}/...`.
- **`{{ANOTHER_VALID_IP_LINKED_TO_PASSED_TENANT_IN_THE_PASSED_AZ}}`:** Another IP from a subnet assigned to the `{{TENANT}}` in `{{AZ_1}}/{{AZ_2}}/...`.

Link: `{{CONFIG_REPO_DOMAIN}}/actions/workflows/smoke-test-dns-api.yml`

The below workflow with the passed values includes the following steps:
1. Checkout of the repository
2. Set up of the Go GitHub Action
3. Disallowance of smoke tests on the standby side if the target environment is either QA or LAB (as they do not have the standby side)
4. Execution of the smoke test with the passed parameters via a Go script (includes create record, update record, get record and delete record)

**Appearance:**

![GitHub Actions Smoke Test Workflow Input](https://confluence.rakuten-it.com/confluence/download/attachments/6857865109/smoketest.png)

Click "Run Workflow" and set:
- **Use workflow from:** Tag `{{ACTIONS_WORKFLOW_TAG}}`
- **Target Environment:** `{{ENV}}`
- **Cluster to smoke test against (QA and LAB have no Standby cluster — use ACTIVE there):** `ACTIVE`
- **Side to smoke test against (Color - blue or green) (choose either if the target is the QA environment as this value is irrelevant there):** `{{ACTIVE_CLUSTER_INACTIVE_COLOR}}`
- **OneCloud tenant to smoke test against:** `{{TENANT}}`
- **DNS zone to create the record under, e.g. <tenant>.<az>.dcnw.rakuten., <subdomain>.jp.local., etc.:** `{{ZONE}}`
- **Record type:** `Address`
- **Record hostname:** `test-smoke-{{APP_ROLLOUT_COMMIT_HASH}}`
- **Content to create the record with, e.g. a single IP address for an Address record and a canonical name for a CNAME record:** `{{VALID_IP_LINKED_TO_PASSED_TENANT_IN_THE_PASSED_AZ}}`
- **Different content to patch the record to:** `{{ANOTHER_VALID_IP_LINKED_TO_PASSED_TENANT_IN_THE_PASSED_AZ}}`

**Expected Result:** All four smoke-test calls succeed. A sample of the output can be found in Section 15.1.
**Validation:** You will see success (HTTP Status Code 200) in all 4 API calls with POST, PATCH and DELETE have 2, 3 and 2 entries in the `success` field of the response.


| ✅ Condition to proceed to the next step | ⛔ Action if condition not met |
|---|---|
| All four smoke-test calls return the results described above | Any call fails, or logs an unexpected error — abort per Section 10; do not proceed to the traffic cutover in Step 10. |

---

### Step 10 — Active Cluster: Hit the deploy workflow to shift traffic to the `{{ACTIVE_CLUSTER_INACTIVE_COLOR}}` side *(only if `{{IS_TRAFFIC_REROUTE_NEEDED}}` and `{{IS_IMAGE_UPDATE_NEEDED}}` are checked)*

> **Action type:** CHANGE
>
> **Why:** Switch the traffic to the `{{ACTIVE_CLUSTER_INACTIVE_COLOR}}` side so that users may start using the new version on the `{{ACTIVE_CLUSTER_INACTIVE_COLOR}}` side
>
> **Scope:** Shifts internal routing on `{{ACTIVE_CLUSTER}}` to `{{ACTIVE_CLUSTER_INACTIVE_COLOR}}` from `{{ACTIVE_CLUSTER_ACTIVE_COLOR}}`
>
> **Abortion:** ✅ Safe to abort/revert — shift back to `{{ACTIVE_CLUSTER_ACTIVE_COLOR}}` if needed.
>
> **Automation Level:** Pipeline (GitHub Actions Workflow)

Link: `{{CONFIG_REPO_DOMAIN}}/actions/workflows/deploy.yml`

The below workflow with the passed values includes the following steps:
1. Checkout of the passed commit hash
2. Restoration of the git-crypt symmetric key and unlocking of the repository
3. Mapping of the cluster to the directory where the manifests reside.
4. Set up of `kubectl`
5. Blue-Green Traffic routing switch by re-applying the Istio VirtualService
6. Locking of the repository

**Appearance:**

![GitHub Actions Deployment Workflow Input](https://confluence.rakuten-it.com/confluence/download/attachments/6857865109/deployment.png)

Click "Run Workflow" and set:
- **Use workflow from:** Tag `{{ACTIONS_WORKFLOW_TAG}}`
- **Cluster to deploy to:** `{{ACTIVE_CLUSTER}}`
- **Full Configuration commit hash to check out (blank = default branch/tag):** `{{CONFIG_ROLLOUT_COMMIT_HASH}}`
- **Check this if the common resources need to be applied:** ❌
- **Check this if the app-blue resources need to be applied:** ❌
- **Check this if the app-green resources need to be applied:** ❌
- **Application Release version:** *(blank)*
- **Short Application Commit hash:** *(blank)*
- **Check this if you want traffic to be transferred to the blue/green sides:** ✅
- **Percentage of traffic to route to the blue deployment:** `100` if `{{ACTIVE_CLUSTER_INACTIVE_COLOR}}`=blue and `{{ACTIVE_CLUSTER_ACTIVE_COLOR}}`=green; `0` if `{{ACTIVE_CLUSTER_INACTIVE_COLOR}}`=green and `{{ACTIVE_CLUSTER_ACTIVE_COLOR}}`=blue
- **Percentage of traffic to route to the green deployment:** `0` if `{{ACTIVE_CLUSTER_INACTIVE_COLOR}}`=blue and `{{ACTIVE_CLUSTER_ACTIVE_COLOR}}`=green; `100` if `{{ACTIVE_CLUSTER_INACTIVE_COLOR}}`=green and `{{ACTIVE_CLUSTER_ACTIVE_COLOR}}`=blue

**Expected Result:** The workflow run completes; VirtualService weights on `{{ACTIVE_CLUSTER}}` now show `{{ACTIVE_CLUSTER_INACTIVE_COLOR}}`=100%, `{{ACTIVE_CLUSTER_ACTIVE_COLOR}}`=0%.
**Validation:** Re-run the "View Istio VirtualService Weights" workflow (Step 6D of Section 6) against `{{ACTIVE_CLUSTER}}` and confirm `{{ACTIVE_CLUSTER_INACTIVE_COLOR}}` is now at 100%.

| ✅ Condition to proceed to the next step | ⛔ Action if condition not met |
|---|---|
| Workflow run completes successfully and weights confirm the shift | Workflow fails — do not proceed; investigate before retrying. |

---

## 12. Rollback Procedure

> **Action type:** CHANGE
>
> **Why:** As part of the rollback executed in case issues occur.

> Any or all of the below steps are to be run, as necessary.

### 12.1 Rollback Trigger Criteria

Initiate rollback if **any** of the following conditions are met (identical to Section 10's global abortion criteria):

- [ ] Application Down
- [ ] Pods are not coming up even after repeated attempts
- [ ] Smoke tests failed
- [ ] Success Rate of the Application goes below 90%

> ⚠️ **Escalation note:**
> - If rollback is needed, in most cases, the operator and buddy should judge whether a rollback/abortion is warranted.
> - If rollback is needed and the operator and buddy are not able to judge whether a rollback/abortion is warranted or if there is the risk of impact to other systems, contact L4 and L3 to determine which rollback steps are required.
> - If the rollback procedure does not result in full recovery, contact L3 and L2 to decide next actions.

### 12.2 Rollback Steps

**If a schema rollback is deemed necessary**
*(applicable only if `{{IS_SCHEMA_MIGRATION_NEEDED}}` is checked)*
Link: `{{CONFIG_REPO_DOMAIN}}/actions/workflows/cql-schema-rollback.yml`

The below workflow with the passed values includes the following steps:
1. Checkout of the passed commit hash
2. Restoration of the git-crypt symmetric key and unlocking of the repository
3. Validation of the target version
4. Resolution of the cqltrack profile
5. Determination and validation of rollback
6. Schema rollback
7. Locking of the repository

**Appearance:**

![GitHub Actions Schema Incorporation Rollback Workflow Input](https://confluence.rakuten-it.com/confluence/download/attachments/6857865109/schemaincorporationrollback.png)

Click "Run Workflow" and set:
- **Use workflow from:** Tag `{{ACTIONS_WORKFLOW_TAG}}`
- **Target environment:** `{{ENV}}`
- **Full Configuration commit hash to check out (blank = default branch/tag):** `{{CONFIG_ROLLOUT_COMMIT_HASH}}`
- **Roll back to this migration version, e.g. 5 to undo everything after V005:** `{{SCHEMA_MIGRATION_FILE_COUNT_AS_IS}}`

**Expected Result:** The GitHub Actions workflow run completes successfully. A sample of the output is shown below:

```bash
Rolling back V008  test ... OK (2200ms)

Rolled back 1 migration(s).
```
**Validation:** Confirm the command completed with no error output.

**If a rollback data patch is deemed necessary:**
*(applicable only if `{{IS_DATA_PATCH_NEEDED}}` is checked)*

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

![GitHub Actions Data Patch Workflow Input](https://confluence.rakuten-it.com/confluence/download/attachments/6857865109/datapatch.png)

Click "Run Workflow" and set:
- **Use workflow from:** Tag `{{ACTIONS_WORKFLOW_TAG}}`
- **Target environment:** `{{ENV}}`
- **Full Configuration commit hash to check out (blank = default branch/tag):** `{{CONFIG_ROLLOUT_COMMIT_HASH}}`
- **Path to the CQL file to apply, relative to the repository root** `{{DB_ROLLBACK_PATCH_FILE_PATH}}`

**Expected Result:** Success.
**Validation:** Confirm the command completed with no error output on the GH Actions Console.

**If an image patch is deemed necessary:**
*(applicable only if `{{IS_IMAGE_UPDATE_NEEDED}}` is checked)*

Link: `{{CONFIG_REPO_DOMAIN}}/actions/workflows/deploy.yml`

The below workflow with the passed values includes the following steps:
1. Checkout of the passed commit hash
2. Restoration of the git-crypt symmetric key and unlocking of the repository
3. Mapping of the cluster to the directory where the manifests reside.
4. Set up of `kubectl`
5. Application of consul resources, network policies, dedicated gateway resources, database credential secrets, HPA configuration, filebeat configuration, domain claim resources and load balancer resources if opted for.
6. Application of app-blue/app-green deployments as applicable.
7. Blue-Green Traffic routing switch by re-applying the Istio VirtualService if opted for
8. Locking of the repository

**Appearance:**

![GitHub Actions Deployment Workflow Input](https://confluence.rakuten-it.com/confluence/download/attachments/6857865109/deployment.png)

Click "Run Workflow" and set:
- **Use workflow from:** Tag `{{ACTIONS_WORKFLOW_TAG}}`
- **Cluster to deploy to:** `{{ACTIVE_CLUSTER}}`/`{{STANDBY_CLUSTER}}` as applicable
- **Full Configuration commit hash to check out (blank = default branch/tag):** `{{CONFIG_ROLLBACK_COMMIT_HASH}}`
- **Check this if the common resources need to be applied:** ✅ or ❌ as necessary
- **Check this if the app-blue resources need to be applied:** ✅ or ❌ as necessary
- **Check this if the app-green resources need to be applied:** ✅ or ❌ as necessary
- **Application Release version:** `{{APP_ROLLBACK_VERSION}}`
- **Short Application Commit hash:** `{{APP_ROLLBACK_COMMIT_HASH}}`
- **Check this if you want traffic to be transferred to the blue/green sides:** ✅ or ❌ as necessary
- **Percentage of traffic to route to the blue deployment:** `100` or `0` as necessary
- **Percentage of traffic to route to the green deployment:** `100` or `0` as necessary

**Expected Result:** The GitHub Actions workflow run completes successfully.
**Validation:** See the pod-level rollout validation below.

```bash
roc login -c {{ACTIVE_CLUSTER}}/{{STANDBY_CLUSTER}} -n dns-api-apollo
```

Then, run

```bash
kubectl get pods
```

**Expected Result:**

```
NAME                                   READY   STATUS    RESTARTS   AGE
apollo-consul-0                        2/2     Running   0          103d
apollo-consul-1                        2/2     Running   0          103d
apollo-consul-2                        2/2     Running   0          103d
app-blue-7c4849585b-5v9mh              3/3     Running   0          251d
app-blue-7c4849585b-lcp6z              3/3     Running   0          251d
app-blue-7c4849585b-nvxg5              3/3     Running   0          251d
app-green-864d9d65b4-dcv47             3/3     Running   0          251d
app-green-864d9d65b4-fjk6q             3/3     Running   0          251d
app-green-864d9d65b4-r8h8k             3/3     Running   0          251d
gateway-app-gateway-86598f8c96-9xcjv   1/1     Running   0          345d
gateway-app-gateway-86598f8c96-c4nct   1/1     Running   0          345d
gateway-app-gateway-86598f8c96-f6vjk   1/1     Running   0          345d
```

**Validation:** The `app-{{ACTIVE_CLUSTER_INACTIVE_COLOR}}` or `app-{{STANDBY_CLUSTER_INACTIVE_COLOR}}` deployment pods should be new and in `Running` state.

**Smoke Testing after Rollback:**

**Legend**

- **`{{TENANT}}`:** A valid OneCloud tenant
- **`{{ZONE}}`:** The DNS zone under which we wish to create records. Format: `{{TENANT}}.{{AZ_1}}/{{AZ_2}}/....dcnw.rakuten.`.
- **`{{VALID_IP_LINKED_TO_PASSED_TENANT_IN_THE_PASSED_AZ}}`:** IP from a subnet assigned to the `{{TENANT}}` in `{{AZ_1}}/{{AZ_2}}/...`.
- **`{{ANOTHER_VALID_IP_LINKED_TO_PASSED_TENANT_IN_THE_PASSED_AZ}}`:** Another IP from a subnet assigned to the `{{TENANT}}` in `{{AZ_1}}/{{AZ_2}}/...`.

Link: `{{CONFIG_REPO_DOMAIN}}/actions/workflows/smoke-test-dns-api.yml`

The below workflow with the passed values includes the following steps:
1. Checkout of the repository
2. Set up of the Go GitHub Action
3. Disallowance of smoke tests on the standby side if the target environment is either QA or LAB (as they do not have the standby side)
4. Execution of the smoke test with the passed parameters via a Go script (includes create record, update record, get record and delete record)

**Appearance:**

![GitHub Actions Smoke Test Workflow Input](https://confluence.rakuten-it.com/confluence/download/attachments/6857865109/smoketest.png)

Click "Run Workflow" and set:
- **Use workflow from:** Tag `{{ACTIONS_WORKFLOW_TAG}}`
- **Target Environment:** `{{ENV}}`
- **Cluster to smoke test against (QA and LAB have no Standby cluster — use ACTIVE there):** `ACTIVE` or `STANDBY` as necessary
- **Side to smoke test against (Color - blue or green) (choose either if the target is the QA environment as this value is irrelevant there):** `{{ACTIVE_CLUSTER_INACTIVE_COLOR}}` or `{{STANDBY_CLUSTER_INACTIVE_COLOR}}` as necessary
- **OneCloud tenant to smoke test against:** `{{TENANT}}`
- **DNS zone to create the record under, e.g. <tenant>.<az>.dcnw.rakuten., <subdomain>.jp.local., etc.:** `{{ZONE}}`
- **Record type:** `Address`
- **Record hostname:** `test-smoke-rollback-{{APP_ROLLOUT_COMMIT_HASH}}`
- **Content to create the record with, e.g. a single IP address for an Address record and a canonical name for a CNAME record:** `{{VALID_IP_LINKED_TO_PASSED_TENANT_IN_THE_PASSED_AZ}}`
- **Different content to patch the record to:** `{{ANOTHER_VALID_IP_LINKED_TO_PASSED_TENANT_IN_THE_PASSED_AZ}}`

**Expected Result:** All four smoke-test calls succeed. A sample of the output can be found in Section 15.1.
**Validation:** You will see success (HTTP Status Code 200) in all 4 API calls with POST, PATCH and DELETE have 2, 3 and 2 entries in the `success` field of the response.


### 12.3 RTO (Recovery Time Objective)

| Scenario                                           | Estimated RTO | Notes                                                       |
|-----------------------------------------------------|---------------|--------------------------------------------------------------|
| Rollback after rollout but before traffic switch    | No downtime   | Only pods on one side affected                                |
| Rollback after rollout and traffic switch           | ~2 minutes    | Switch to the other side which has the older stable version  |
| Rollback with config change                         | ~20 minutes   | Config revert + pod restart possibly required                 |

> **RTO Commitment:** A maximum of `30 minutes`.


### 12.4 Post-Rollback Tasks

- Create trouble report.
- Capture evidence of tests performed after rollback.

---

## 13. Post-Operation Monitoring

**Monitoring Duration:** `30 minutes` *(Recommended minimum: 10 min for Low risk, 30 min for Medium, 60 min for High)*

Navigate to `{{GRAFANA_LINK}}` and open the "API Health - `{{GRAFANA_ENV}}`" and "DNS API Latency and Request Overview - `{{GRAFANA_ENV}}`" dashboards.

### 13.1 Metrics to Watch

| Metric | Dashboard | Trigger Rollback If… |
|--------|-----------|------------------------|
| API Health | API Health - `{{GRAFANA_ENV}}` | Health = Down |
| Application Success Rate | DNS API Latency and Request Overview - `{{GRAFANA_ENV}}` | Success Rate less than 90% |

**API Health Dashboard Reference**

**Appearance:**

![Grafana Dashboard for API Health](https://confluence.rakuten-it.com/confluence/download/attachments/6857865109/apihealth.png)

**Success Rate Dashboard Reference**

**Appearance:**

![Grafana Dashboard for Application Success Rate](https://confluence.rakuten-it.com/confluence/download/attachments/6857865109/successrate.png)

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
| 1 | Cassandra Migration | Under Planning | The schema incorporation step is prone to failure due to network issues. Should check with the DBaaS team how to stabilise cluster connectivity |


> **Additional Notes:** _Describe any further edge cases, known limitations, or improvement suggestions here._

---
