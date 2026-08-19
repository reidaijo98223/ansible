# ansible

# Secure Middleware Deployment Playbook (Keeper Vault + Nexus Integration)

This Ansible playbook provides automated orchestrations to provision OpenJDK, Apache HTTP Server, and Apache Tomcat. It combines secure, zero-hardcode credential harvesting with private repository artifact delivery.

## Secure Automation Pipeline

```text
[Ansible Control Node Execution]
           │
           ▼
[Authorize Keeper Secrets Manager] (Loads local KSM_CONFIG profiles)
           │
           ▼
[Query Dynamic Credentials] ──► Fetch Username/Password via Keeper UID Lookup
           │
           ▼
[Establish Target SSH Sessions] (Elevate to became: yes sudo access)
           │
           ▼
[Pull Nexus Repositories Artifacts]
           ├─── OpenJDK 21 LTS Tarball (Authenticated)
           ├─── Apache HTTPD Source Code (Authenticated)
           └─── Apache Tomcat Engine (Authenticated)
           │
           ▼
[Deploy Python Orchestrator Utility] ──► Uploads `middleware_tomcat_java_http_installer.py`
           │
           ▼
[Trigger Provisioning Actions] ──► Local compiles, permission masks, and symlinks
           │
           ▼
[Sanitize Staging Filesystem] ──► Purge /tmp workspace elements ──► [Done]
```

## Key Features

* **Zero-Trust Vault Management:** Leverages the official `keepersecurity.keeper_secrets_manager` lookup engine plugin. Credentials exist solely within temporary process memory during execution and are never saved to disk, written to task blocks, or leaked in Git commits.
* **Private Warehouse Sourcing:** Routes artifact retrievals exclusively through your internal corporate Nexus Repository Manager, ensuring network containment without public internet dependencies.
* **Atomic Workspace Cleaning:** Automatically deletes the execution utility from the remote target server's `/tmp` folder right after the installation completes, keeping the target clean.

## Prerequisites & Setup

### 1. Control Node Core Packages
The execution engine requires the specialized Keeper integration library dependencies:
```bash
# Install the core Keeper Secrets Manager collection
ansible-galaxy collection install keepersecurity.keeper_secrets_manager
```

### 2. Export Vault Token Profiler
Ensure your local system profile exports your authenticated Keeper client integration token before kicking off deployment jobs:
```bash
export KSM_CONFIG="Base64EncodedConfigTokenStringHere=="
```

## Deployment Instructions

1. Map your specific targets inside your local Ansible inventory tracking configurations.
2. Update the `keeper_nexus_record_uid` variable inside the playbook with the corresponding alphanumeric Record UID from your Keeper Security Vault.
3. Trigger the deployment workflow:

```bash
ansible-playbook -i inventories/prod_hosts deploy_middleware.yml
```
---

# 3rd Party Software Inventory Report Automation

An automated Ansible solution that inventories third-party software, operating system specs, and configuration states across all infrastructure nodes. It compiles the metrics into a formatted CSV report, publishes the asset to an Azure-hosted Sonatype Nexus repository, and emails the final deliverable to designated stakeholders.

## Workflow Overview

```text
[Ansible Control Node Execution]
           │
           ▼
[Authorize Keeper Secrets Manager] (Loads local KSM_CONFIG profiles)
           │
           ▼
[Query Dynamic Credentials] ──► Fetch Nexus API Username/Password via Keeper UID Lookup
           │
           ▼
[Establish Target SSH Sessions] (Elevate to become: yes sudo access)
           │
           ▼
[Execute Remote Shell Inventory] ──► Run `check-3rdPtySoftwareInventory.sh` on endpoints
           │
           ▼
[Aggregate Data Streams] ──► Save multi-node outputs to localized CSV text files
           │
           ▼
[Sanitize Data Format] ──► Run sed strings to inject headers, strip brackets, and align columns
           │
           ▼
[Synchronize Nexus Repository Artifacts]
           ├─── Authenticate using dynamic Keeper credentials
           ├─── Issue HTTP REST API DELETE commands to purge expired reports
           └─── Issue HTTP REST API PUT/UPLOAD commands to push fresh CSV files
           │
           ▼
[Package Timestamped Deliverables] ──► Create dynamic date-stamped file masks
           │
           ▼
[Dispatch Email Reports] ──► Route CSV attachments to mail_id via SMTP Relay ──► [Done]
```

## Components

### 1. Ansible Playbook (`site.yml`)
Initializes the execution environment across all hosts, sets a critical 20-minute execution timeout threshold (`ansible_command_timeout: 1200`), and maps targeted infrastructure to the dedicated reporting role.

### 2. Custom Data Collection Script (`check-3rdPtySoftwareInventory.sh`)
An advanced bash utility executed locally on endpoint machines to parse system details. It captures a comprehensive 45-column data matrix including:
* **Host Metrics:** OS Version, patch history, total memory, CPU/core counts.
* **System Daemons:** Qualys agent status, FireEye, Filebeat, Metricbeat, Stunnel configurations.
* **Runtimes & Frameworks:** Python, Redis, Postgres, Apache, Tomcat, Node.js, Git.
* **Symlink Mapping:** Granular version mapping for Java (8, 11, 17, 21) and Apache/Tomcat runtimes.

### 3. Dedicated Role Execution (`roles/3rdPtySoftwareInventoryReport/tasks/main.yml`)
Orchestrates the lifecycle of the compiled data on the delegated control node (`rg90-ansible-cxp.abc.rg`):
* **Aggregation:** Aggregates unstructured script output streams globally into a temporary CSV file (`/tmp/3rdPtySoftwareInventoryReport.csv`).
* **Sanitization:** Executes programmatic `sed` filtering strings to strip brackets, format text anomalies, repair syntax truncations, and cleanly inject structural CSV headers.
* **Artifact Publishing:** Communicates with Azure Nexus using `curl` API methods to sequentially delete expired inventory assets and push live, updated artifacts.
* **Notification Deliverables:** Applies date-hour timestamps to the physical reporting files and passes the file to an SMTP relay (`://abc.com`) for email distribution.

## CSV Inventory Schema Reference

The script aggregates data strictly mapping to the following tabular layout:

| Category | Columns |
| :--- | :--- |
| **System Identity** | `SERVER_NAME`, `SERVER_ALIAS`, `OS_VERSION`, `OS_PATCH_VERSION`, `OS_PATCH_STATUS`, `OS_LAST_PATCHED` |
| **Hardware Resources** | `VM_CPU_COUNT`, `VM_CORES_COUNT`, `VM_MEMORY_TOTAL_GB` |
| **Security & Monitoring** | `STUNNEL_CONFIG`, `QUALYS_AGENT`, `FILEBEAT`, `FIREEYE`, `HEARTBEAT`, `METRICBEAT` |
| **Build Tools & Runtimes** | `ANT`, `MAVEN`, `NODE`, `GIT`, `GITLFS`, `GRAFANA`, `KONG_NGINX`, `POSTGRES`, `PYTHON_VERSION_1`, `PYTHON_VERSION_2`, `REDIS`, `SOLR` |
| **Java Ecosystem** | `ACTIVE_JAVA_APPS`, `INSTALLED_JAVA`, `APP_JAVA_VERSION`, `JAVA-8_SYMLINK_VERSION`, `JAVA-11_SYMLINK_VERSION`, `JAVA-17_SYMLINK_VERSION`, `JAVA-21_SYMLINK_VERSION` |
| **Tomcat Ecosystem** | `ACTIVE_TOMCAT_APPS`, `INSTALLED_TOMCAT`, `APP_TOMCAT_VERSION`, `TOMCAT-8_SYMLINK_VERSION`, `TOMCAT-9_SYMLINK_VERSION`, `TOMCAT-10_SYMLINK_VERSION`, `TOMCAT-11_SYMLINK_VERSION` |
| **Apache Web Server** | `ACTIVE_APACHE_APPS`, `INSTALLED_APACHE`, `APACHE_SYMLINK_VERSION`, `APACHE-2_SYMLINK_VERSION` |

---
_Note: Code blocks inside the roles contain structural scaffolding for future migrations to **Keeper Secrets Manager** to replace plain-text variable handling for Nexus authentication tokens._



