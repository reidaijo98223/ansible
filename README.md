# ansible

# Secure Middleware Deployment Playbook (Keeper Vault + Nexus Integration)

This Ansible playbook provides automated orchestrations to provision OpenJDK, Apache HTTP Server, and Apache Tomcat. It combines secure, zero-hardcode credential harvesting with private repository artifact delivery.

## ⚙️ Secure Automation Pipeline

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

## 🚀 Key Features

* **Zero-Trust Vault Management:** Leverages the official `keepersecurity.keeper_secrets_manager` lookup engine plugin. Credentials exist solely within temporary process memory during execution and are never saved to disk, written to task blocks, or leaked in Git commits.
* **Private Warehouse Sourcing:** Routes artifact retrievals exclusively through your internal corporate Nexus Repository Manager, ensuring network containment without public internet dependencies.
* **Atomic Workspace Cleaning:** Automatically deletes the execution utility from the remote target server's `/tmp` folder right after the installation completes, keeping the target clean.

## 📋 Prerequisites & Setup

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

## 🛠️ Deployment Instructions

1. Map your specific targets inside your local Ansible inventory tracking configurations.
2. Update the `keeper_nexus_record_uid` variable inside the playbook with the corresponding alphanumeric Record UID from your Keeper Security Vault.
3. Trigger the deployment workflow:

```bash
ansible-playbook -i inventories/prod_hosts deploy_middleware.yml
```
---





