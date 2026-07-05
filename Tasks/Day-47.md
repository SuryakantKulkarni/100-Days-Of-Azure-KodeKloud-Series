# Day 47 – SQL Database Migration and Setup

---

## Task Overview

The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to Azure. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. As part of this migration, they are focusing on setting up and managing Azure SQL Databases, implementing backup processes, and ensuring data recovery. Below are the tasks they require you to perform: 

**Task 1: Create an Azure SQL Database** 

  - Create a **publicly accessible** Azure SQL Database instance with the following details:
      
      - **Database Name:** `nautilus-sqldb`.
      - **Server Name:** `nautilus-server-31425`.
      - **Location:** `West US Backup Storage`
      - **Redundancy:** `Locally-redundant backup storage`.
      - **Hardware Configuration:** `Basic (For less demanding workloads)`.
      - **Admin Username:** `nautilus-admin`.
      - **Admin Password:** Set an appropriate password.
      - **Database Size:** Set to `2 GiB`.
      - Keep all other configurations as `default`.

  - Ensure the database is in the `Ready` state.

**Task 2: Create a Storage Account** 

- Create a **Storage Account** named `nautilusst23682`.
- Configure a **Blob Container** named `nautilus-container-31620` within this storage account.

**Task 3: Backup the Azure SQL Database** 

- Take a backup of the Azure SQL Database instance `nautilus-sqldb` and store it in the **Blob Container:**

    - **Storage Account:** `nautilusst23682`.
    - **Blob Container:** `nautilus-container-31620`.
    - **Backup File Name:** `nautilus-db-backup`.

- Ensure the backup is fully exported to the blob container.

**Task 4: Download the Backup** 

- Download the backup file from the Blob Container to the `/opt` directory on the `azure-client` host.
- Ensure the file is accessible and properly named based on its extension.

**Requirements for Completion** 

- Ensure the SQL Database is in the `Ready` state.
- Confirm the backup is stored in the specified Blob Container.
- Verify the backup file is successfully downloaded to the `/opt` directory on the client host

---

# Step-by-Step Implementation

### Step 1: Get the Resource Group

```bash
RG=$(az group list --query "[0].name" -o tsv)

echo $RG
```

#### Explanation

Retrieves the default Azure Resource Group used throughout the lab.

---

### Step 2: Create the Storage Account

```bash
az storage account create \
--resource-group $RG \
--name nautilusst23682 \
--location westus \
--sku Standard_LRS \
--kind StorageV2
```

#### Explanation

Creates a Storage Account using **Standard Locally Redundant Storage (LRS)**.

---

### Step 3: Retrieve the Storage Account Key

```bash
KEY=$(az storage account keys list \
--resource-group $RG \
--account-name nautilusst23682 \
--query "[0].value" \
-o tsv)

echo $KEY
```

#### Explanation

Retrieves the storage account access key required for Blob Storage operations.

---

### Step 4: Create the Blob Container

```bash
az storage container create \
--account-name nautilusst23682 \
--account-key "$KEY" \
--name nautilus-container-31620
```

#### Explanation

Creates a private Blob container where the SQL database backup will be stored.

---

### Step 5: Create the Azure SQL Server

Choose a strong password.

```bash
SQLPASS='Surya@27'
```

```bash
az sql server create \
--resource-group $RG \
--name nautilus-server-31425 \
--location westus \
--admin-user nautilus-admin \
--admin-password "$SQLPASS"
```

#### Explanation

Creates a publicly accessible Azure SQL Server with SQL Authentication enabled.

---

### Step 6: Configure Firewall Rules

Allow Azure Services:

```bash
az sql server firewall-rule create \
--resource-group $RG \
--server nautilus-server-31425 \
--name AllowAzure \
--start-ip-address 0.0.0.0 \
--end-ip-address 0.0.0.0
```

Allow the client machine:

```bash
MYIP=$(curl -s ifconfig.me)

az sql server firewall-rule create \
--resource-group $RG \
--server nautilus-server-31425 \
--name ClientIP \
--start-ip-address $MYIP \
--end-ip-address $MYIP
```

#### Explanation

Allows Azure services and the current client IP to access the SQL Server.

---

### Step 7: Create the Azure SQL Database

```bash
az sql db create \
--resource-group $RG \
--server nautilus-server-31425 \
--name nautilus-sqldb \
--service-objective Basic \
--backup-storage-redundancy Local
```

#### Explanation

Creates the SQL Database using the **Basic** compute tier with **Locally-redundant backup storage**.

---

### Step 8: Verify Database Status

```bash
az sql db show \
--resource-group $RG \
--server nautilus-server-31425 \
--name nautilus-sqldb \
--query status
```

Expected:

```text
Online
```

#### Explanation

Ensures that the database is fully provisioned and ready for use.

---

### Step 9: Prepare the Backup Destination URI

```bash
URI="https://nautilusst23682.blob.core.windows.net/nautilus-container-31620/nautilus-db-backup.bacpac"
```

#### Explanation

Defines the destination path where the exported database will be stored.

---

### Step 10: Export the SQL Database

```bash
az sql db export \
--resource-group $RG \
--server nautilus-server-31425 \
--name nautilus-sqldb \
--admin-user nautilus-admin \
--admin-password "$SQLPASS" \
--storage-key "$KEY" \
--storage-key-type StorageAccessKey \
--storage-uri "$URI"
```

Wait until the export completes successfully.

#### Explanation

Exports the Azure SQL Database as a **.bacpac** backup into the Blob Storage container.

---

### Step 11: Verify the Backup Blob

```bash
az storage blob list \
--account-name nautilusst23682 \
--account-key "$KEY" \
--container-name nautilus-container-31620 \
-o table
```

Expected:

```text
Name
--------------------------
nautilus-db-backup.bacpac
```

#### Explanation

Confirms that the database export was successfully uploaded to Blob Storage.

---

### Step 12: Download the Backup

```bash
az storage blob download \
--account-name nautilusst23682 \
--account-key "$KEY" \
--container-name nautilus-container-31620 \
--name nautilus-db-backup.bacpac \
--file /opt/nautilus-db-backup.bacpac
```

#### Explanation

Downloads the backup file to the required directory on the Azure client host.

---

### Step 13: Verify the Download

```bash
ls -lh /opt/nautilus-db-backup.bacpac
```

Expected:

```text
-rw-r--r-- ... /opt/nautilus-db-backup.bacpac
```

#### Explanation

Verifies that the backup file exists in the required location.

---

## Azure Portal (GUI) Steps

### Create SQL Server

Navigate to:

```text
Azure Portal
→ SQL Servers
→ Create
```

Configure:

| Setting        | Value                 |
| -------------- | --------------------- |
| Server Name    | nautilus-server-31425 |
| Region         | West US               |
| Authentication | SQL Authentication    |
| Admin Username | nautilus-admin        |
| Password       | Strong Password       |

Enable **Allow Azure Services** and create the server.

---

### Create SQL Database

Navigate to:

```text
SQL Databases
→ Create
```

Configure:

| Setting           | Value                 |
| ----------------- | --------------------- |
| Database Name     | nautilus-sqldb        |
| Server            | nautilus-server-31425 |
| Compute Tier      | Basic                 |
| Database Size     | 2 GB                  |
| Backup Redundancy | Locally-redundant     |

Deploy the database and wait until its status becomes **Online**.

---

### Create Storage Account

Navigate to:

```text
Storage Accounts
→ Create
```

Configure:

| Setting     | Value           |
| ----------- | --------------- |
| Name        | nautilusst23682 |
| Region      | West US         |
| Performance | Standard        |
| Redundancy  | LRS             |

Deploy the Storage Account.

---

### Create Blob Container

Navigate to:

```text
Storage Account
→ Containers
→ + Container
```

Create:

```text
nautilus-container-31620
```

Keep Public Access:

```text
Private
```

---

### Export the SQL Database

Navigate to:

```text
SQL Databases
→ nautilus-sqldb
→ Export
```

Configure:

| Setting         | Value                     |
| --------------- | ------------------------- |
| Storage Account | nautilusst23682           |
| Container       | nautilus-container-31620  |
| File Name       | nautilus-db-backup.bacpac |
| Authentication  | SQL Authentication        |
| Username        | nautilus-admin            |
| Password        | Your Password             |

Start the export.

---

### Download the Backup

Navigate to:

```text
Storage Account
→ Containers
→ nautilus-container-31620
```

Select:

```text
nautilus-db-backup.bacpac
```

Download it to the Azure client host under:

```text
/opt/nautilus-db-backup.bacpac
```

---

## Best Practices

* Use **Locally-redundant Backup Storage** for cost-effective backup storage.
* Always configure firewall rules before connecting to Azure SQL.
* Verify the SQL Database status is **Online** before exporting.
* Store backups in Blob Storage for easy recovery.
* Validate downloaded backups before deleting any production database.

## Key Learnings

* Creating Azure SQL Servers and Databases.
* Configuring SQL Authentication and firewall rules.
* Creating Azure Storage Accounts and Blob Containers.
* Exporting Azure SQL Databases as **BACPAC** files.
* Downloading database backups from Azure Blob Storage.
* Verifying SQL Database health and backup integrity.
