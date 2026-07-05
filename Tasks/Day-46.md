# Day 46 – EventHub to Blob Storage Integration Setup

---

## Task Overview

The Nautilus DevOps team wants to integrate an Azure Virtual Machine with Azure Event Hubs and Azure Blob Storage for centralized log collection and backup. Follow these steps to complete the task:

- **Create Azure Event Hubs Namespace:**

  - Create an Event Hubs namespace named `devops-namespace` in the `East US` region.
  - Select the **Standard** pricing tier. Make sure to enable `Enable Auto-inflate`.

- **Create an Event Hub:**

  - Within the namespace, create an Event Hub named `devops-hub`.

- **Set Up Azure Blob Storage for Log Backup:**

  - Create a Storage Account named `devopsst1486` in the `East US` region.
  - Create a container named `devops-backup-6107` within the Storage Account.
  - Ensure the container is publicly accessible for read operations.

- **Verify the Virtual Machine Configuration:**

  - The **client host** already has a Python script named `send_logs.py` located under `/root`. This script is used to send logs to the Event Hub.
  - **Create a Virtual Machine** named `devops-vm` in the `East US` region.
  - Copy the `send_logs.py` script from the **client host** to the `/home/azureuser` directory of the `devops-vm`.
  - Modify the script on the VM to also back up the logs to the `devops-backup-6107` container in the Azure Blob Storage account.

- **Verify Logs:**

  - Ensure the logs are successfully sent to the Event Hub by checking the Event Hubs metrics in the Azure portal.
  - Verify that the logs are backed up to the `devops-backup-6107` container in the Azure Blob Storage.

`Notes:`
- Create the resources only in the `East US` region.
- Use the existing client host to copy the script to the VM.
- Verify both the Event Hubs metrics and the Blob Storage container for successful log ingestion and backup.

---

# Step-by-Step Implementation

### Step 1: Get the Resource Group

```bash
RG=$(az group list --query "[0].name" -o tsv)

echo $RG
```

#### Explanation

Retrieves the default Resource Group where all Azure resources will be created.

---

### Step 2: Create the Event Hubs Namespace

```bash
az eventhubs namespace create \
--resource-group $RG \
--name devops-namespace \
--location eastus \
--sku Standard \
--enable-auto-inflate true \
--maximum-throughput-units 1
```

#### Explanation

Creates an Event Hubs namespace using the **Standard** pricing tier with **Auto-inflate** enabled.

---

### Step 3: Create the Event Hub

```bash
az eventhubs eventhub create \
--resource-group $RG \
--namespace-name devops-namespace \
--name devops-hub
```

#### Explanation

Creates an Event Hub named **devops-hub** inside the namespace.

---

### Step 4: Create the Storage Account

```bash
az storage account create \
--resource-group $RG \
--name devopsst1486 \
--location eastus \
--sku Standard_LRS \
--kind StorageV2
```

#### Explanation

Creates a Storage Account in **East US** using **Standard Locally Redundant Storage (LRS)**.

---

### Step 5: Enable Blob Public Access

```bash
az storage account update \
--resource-group $RG \
--name devopsst1486 \
--allow-blob-public-access true
```

#### Explanation

Allows containers in the storage account to be configured with public read access.

---

### Step 6: Retrieve the Storage Account Key

```bash
KEY=$(az storage account keys list \
--resource-group $RG \
--account-name devopsst1486 \
--query "[0].value" \
-o tsv)

echo $KEY
```

#### Explanation

Retrieves the access key required for Blob Storage operations.

---

### Step 7: Create the Blob Container

```bash
az storage container create \
--account-name devopsst1486 \
--account-key "$KEY" \
--name devops-backup-6107
```

#### Explanation

Creates the Blob container used for log backups.

---

### Step 8: Configure Public Read Access

```bash
az storage container set-permission \
--account-name devopsst1486 \
--account-key "$KEY" \
--name devops-backup-6107 \
--public-access blob
```

Verify:

```bash
az storage container-rm list \
--resource-group $RG \
--storage-account devopsst1486 \
-o table
```

Expected:

```text
Name                  PublicAccess
--------------------  ------------
devops-backup-6107    Blob
```

#### Explanation

Configures the container to allow **public read access for blobs**.

---

### Step 9: Generate SSH Keys

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""
```

Display the public key:

```bash
cat ~/.ssh/id_rsa.pub
```

#### Explanation

Generates an SSH key pair for authenticating to the virtual machine.

---

### Step 10: Create the Virtual Machine

```bash
az vm create \
--resource-group $RG \
--name devops-vm \
--location eastus \
--image Ubuntu2204 \
--size Standard_B1s \
--admin-username azureuser \
--authentication-type ssh \
--ssh-key-values ~/.ssh/id_rsa.pub \
--storage-sku Standard_LRS
```

#### Explanation

Creates an Ubuntu VM using SSH authentication.

---

### Step 11: Retrieve the VM Public IP

```bash
IP=$(az vm show \
--resource-group $RG \
--name devops-vm \
--show-details \
--query publicIps \
-o tsv)

echo $IP
```

#### Explanation

Retrieves the VM's public IP address.

---

### Step 12: Copy the Python Script to the VM

```bash
scp -o StrictHostKeyChecking=no \
/root/send_logs.py \
azureuser@$IP:/home/azureuser/
```

#### Explanation

Copies the existing `send_logs.py` script from the client host to the VM.

---

### Step 13: Connect to the VM

```bash
ssh azureuser@$IP
```

#### Explanation

Logs into the virtual machine.

---

### Step 14: Install Required Python Packages

```bash
python3 -m pip install azure-eventhub azure-storage-blob
```

#### Explanation

Installs the Azure SDK packages required by the Python script.

---

### Step 15: Retrieve the Event Hub Connection String

(On the Azure Client)

```bash
EH_CONN=$(az eventhubs namespace authorization-rule keys list \
--resource-group $RG \
--namespace-name devops-namespace \
--name RootManageSharedAccessKey \
--query primaryConnectionString \
-o tsv)

echo "$EH_CONN"
```

#### Explanation

Retrieves the Event Hubs namespace connection string.

---

### Step 16: Retrieve the Storage Connection String

```bash
ST_CONN=$(az storage account show-connection-string \
--resource-group $RG \
--name devopsst1486 \
--query connectionString \
-o tsv)

echo "$ST_CONN"
```

#### Explanation

Retrieves the Storage Account connection string.

---

### Step 17: Modify the Python Script

Edit the script:

```bash
nano /home/azureuser/send_logs.py
```

Replace the existing contents with the updated script provided in the task.

Replace the placeholders:

```text
<EVENT_HUB_CONNECTION_STRING>

<STORAGE_CONNECTION_STRING>
```

with the values obtained in **Steps 15 and 16**.

Save the file.

#### Explanation

Updates the script to:

* Send logs to Azure Event Hub.
* Upload the logs as **logs.txt** to Azure Blob Storage.

---

### Step 18: Execute the Script

```bash
python3 /home/azureuser/send_logs.py
```

Expected:

```text
Log entry 1: Sample log message
...
Log entry 10: Sample log message

logs.txt uploaded successfully
```

#### Explanation

Sends logs to the Event Hub and uploads **logs.txt** to Blob Storage.

---

### Step 19: Verify Blob Storage Backup

Exit the VM:

```bash
exit
```

List the uploaded blobs:

```bash
az storage blob list \
--account-name devopsst1486 \
--account-key "$KEY" \
--container-name devops-backup-6107 \
-o table
```

Expected:

```text
Name
--------
logs.txt
```

Download the blob:

```bash
az storage blob download \
--account-name devopsst1486 \
--account-key "$KEY" \
--container-name devops-backup-6107 \
--name logs.txt \
--file logs.txt
```

Verify the contents:

```bash
cat logs.txt
```

Expected:

```text
Log entry 1: Sample log message
...
Log entry 10: Sample log message
```

#### Explanation

Confirms that the logs were successfully backed up to Blob Storage.

---

### Step 20: Verify Event Hub Metrics

Navigate to:

```text
Azure Portal
→ Event Hubs Namespace
→ devops-namespace
→ devops-hub
→ Metrics
```

Configure:

| Setting     | Value             |
| ----------- | ----------------- |
| Metric      | Incoming Messages |
| Aggregation | Sum               |
| Time Range  | Last 30 Minutes   |

Expected:

```text
Incoming Messages > 0
```

Typically:

```text
10
```

#### Explanation

Confirms that logs were successfully ingested by Azure Event Hubs.

---

## Common Errors & Fixes

### Error: Connection String is Blank or Invalid

**Cause**

The Event Hub or Storage Account connection string was not updated in the script.

**Fix**

Retrieve the correct values:

```bash
az eventhubs namespace authorization-rule keys list ...
```

and

```bash
az storage account show-connection-string ...
```

---

### Error: Blob Container is Private

**Cause**

Public access was not enabled.

**Fix**

Run:

```bash
az storage container set-permission \
--account-name devopsst1486 \
--account-key "$KEY" \
--name devops-backup-6107 \
--public-access blob
```

---

### Error: Validator Fails Even Though Upload Succeeds

**Cause**

The uploaded blob name is different.

Example:

```text
logs_20260704.txt
```

instead of:

```text
logs.txt
```

**Fix**

Always upload the file as:

```text
logs.txt
```

---

### Error: Python Module Not Found

**Cause**

Azure SDK packages are not installed.

**Fix**

```bash
python3 -m pip install azure-eventhub azure-storage-blob
```

---

## Best Practices

* Use the **Standard** Event Hubs tier for production workloads.
* Enable **Auto-inflate** to automatically scale throughput.
* Store sensitive connection strings securely using Azure Key Vault.
* Use descriptive blob names when appropriate (except in this lab, where `logs.txt` is required).
* Verify Event Hub metrics after sending events.

## Key Learnings

* Creating Azure Event Hubs namespaces and hubs.
* Configuring Blob Storage with public read access.
* Creating Azure Virtual Machines.
* Using Python with the Azure Event Hubs SDK.
* Uploading files to Azure Blob Storage using the Azure Storage SDK.
* Verifying Event Hub metrics and Blob Storage uploads.
