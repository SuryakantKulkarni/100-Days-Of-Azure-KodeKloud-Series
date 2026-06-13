# Day 30 – Create Azure SQL Database

---

## Task Overview

The Nautilus Devops team is strategizing the migration of a portion of their infrastructure to Azure. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. Recently, they started working on creating and configuring some database instances on Azure.

For this task, create one `publicly` accessible Azure SQL Database instance along with the following details:

1) The name of the Azure SQL Database must be `devops-sqldb`.

2) The server name must be `devops-server-11257` under `southcentralus`.

3) The compute + storage configuration should be **Basic (For less demanding workloads)**.

4) The backup storage redundancy should be **Locally-redundant backup storage**.

5) Set the login admin username to `devops-admin` and set an appropriate password.

6) Set the database size to **2 GiB**.

7) Keep the rest of the configurations as `default`. Finally, make sure the database is in the `Ready` state before submitting this task.

---

# Method 1: Azure Portal (GUI)

### Step 1: Open Azure SQL Databases

In Azure Portal navigate to:

```text
SQL Databases
→ Create
```

#### Explanation

This starts the deployment process for a new Azure SQL Database.

---

### Step 2: Configure Basic Database Details

Under the **Basics** tab configure:

| Field | Value |
|---|---|
| Resource Group | Existing Lab Resource Group |
| Database Name | `devops-sqldb` |

For Server, click:

```text
Create New
```

#### Explanation

The database must be deployed under a SQL Server instance.

---

### Step 3: Create SQL Server

Configure:

| Field | Value |
|---|---|
| Server Name | `devops-server-11257` |
| Region | `South Central US` |
| Authentication Method | SQL Authentication |
| Server Admin Login | `devops-admin` |
| Password | Strong Password |
| Confirm Password | Same Password |

Click:

```text
OK
```

#### Explanation

This creates the Azure SQL Server that will host the database.

---

### Step 4: Configure Compute and Storage

Click:

```text
Configure Database
```

Select:

```text
Basic
```

Set:

| Setting | Value |
|---|---|
| Compute Tier | Basic |
| Maximum Data Size | 2 GB |

Click:

```text
Apply
```

#### Explanation

The Basic tier is suitable for lightweight workloads and meets the task requirements.

---

### Step 5: Configure Backup Storage

Under Backup Storage Redundancy select:

```text
Locally-redundant backup storage
```

#### Explanation

This stores backups within the same Azure region and satisfies the required redundancy setting.

---

### Step 6: Configure Networking

Navigate to:

```text
Networking
```

Select:

```text
Public Endpoint
```

Enable:

```text
Allow Azure Services and Resources to Access This Server
```

Set:

```text
Yes
```

#### Explanation

This makes the SQL Database publicly accessible while keeping other settings at default values.

---

### Step 7: Review and Create

Verify:

| Setting | Value |
|---|---|
| Database | devops-sqldb |
| Server | devops-server-11257 |
| Region | South Central US |
| Tier | Basic |
| Backup | Locally-redundant |

Click:

```text
Review + Create
```

After validation succeeds:

```text
Create
```

#### Explanation

Azure validates the configuration and begins deployment.

---

### Step 8: Verify Database Status

After deployment completes:

```text
SQL Databases
→ devops-sqldb
```

Verify:

| Property | Expected Value |
|---|---|
| Status | Online |
| Server | devops-server-11257 |
| Compute Tier | Basic |
| Backup Storage | Local |

#### Explanation

The database must show **Online** or **Ready** before task submission.

---

# Method 2: Azure CLI

### Step 1: Identify Resource Group

Run:

```bash
RG=$(az group list --query "[0].name" -o tsv)

echo $RG
```

#### Explanation

The Resource Group is required for creating Azure SQL resources.

---

### Step 2: Create Azure SQL Server

Run:

```bash
az sql server create \
  --resource-group $RG \
  --name devops-server-11257 \
  --location southcentralus \
  --admin-user devops-admin \
  --admin-password 'StrongPassword123!'
```

#### Explanation

This creates the Azure SQL Server in South Central US.

---

### Step 3: Create Azure SQL Database

Run:

```bash
az sql db create \
  --resource-group $RG \
  --server devops-server-11257 \
  --name devops-sqldb \
  --service-objective Basic \
  --backup-storage-redundancy Local
```

#### Explanation

This creates the SQL Database using the Basic compute tier and locally redundant backups.

---

### Step 4: Configure Public Access

Allow Azure services to connect:

```bash
az sql server firewall-rule create \
  --resource-group $RG \
  --server devops-server-11257 \
  --name AllowAzureServices \
  --start-ip-address 0.0.0.0 \
  --end-ip-address 0.0.0.0
```

#### Explanation

This creates the firewall rule required for public accessibility.

---

# Verification Steps

### Step 5: Verify SQL Server

Run:

```bash
az sql server show \
  --resource-group $RG \
  --name devops-server-11257 \
  --query "{Server:name,Location:location,Admin:administratorLogin}" \
  -o table
```

Expected output:

```text
Server               Location        Admin
-------------------  --------------  ------------
devops-server-11257  southcentralus  devops-admin
```

---

### Step 6: Verify SQL Database

Run:

```bash
az sql db show \
  --resource-group $RG \
  --server devops-server-11257 \
  --name devops-sqldb \
  --query "{Database:name,Status:status,Tier:currentSku.tier,SKU:currentSku.name,Backup:requestedBackupStorageRedundancy}" \
  -o table
```

Expected output:

```text
Database      Status   Tier   SKU     Backup
------------  -------  -----  ------  ------
devops-sqldb  Online   Basic  Basic   Local
```

#### Explanation

This confirms the database is deployed successfully and is in the required Online state.

---

### Step 7: Final Validation

Run:

```bash
echo "=== SERVER ==="

az sql server show \
  --resource-group $RG \
  --name devops-server-11257 \
  --query "{Server:name,Location:location,Admin:administratorLogin}" \
  -o table

echo

echo "=== DATABASE ==="

az sql db show \
  --resource-group $RG \
  --server devops-server-11257 \
  --name devops-sqldb \
  --query "{Database:name,Status:status,Tier:currentSku.tier,SKU:currentSku.name,Backup:requestedBackupStorageRedundancy}" \
  -o table
```

#### Expected Output

```text
=== SERVER ===

Server               Location        Admin
-------------------  --------------  ------------
devops-server-11257  southcentralus  devops-admin

=== DATABASE ===

Database      Status   Tier   SKU     Backup
------------  -------  -----  ------  ------
devops-sqldb  Online   Basic  Basic   Local
```

---

## Best Practices

- Use strong administrator passwords for Azure SQL Servers
- Select the appropriate compute tier based on workload requirements
- Configure backup redundancy according to recovery objectives
- Restrict firewall access to trusted networks whenever possible
- Monitor database performance and storage consumption regularly
- Validate database availability immediately after deployment

## Key Learnings

- Azure SQL Database is a fully managed relational database service
- Azure SQL Server acts as the logical host for SQL databases
- Compute tiers determine performance and resource allocation
- Backup redundancy affects data protection and disaster recovery
- Firewall rules control connectivity to Azure SQL resources
- Azure CLI and Portal both support end-to-end database deployment
