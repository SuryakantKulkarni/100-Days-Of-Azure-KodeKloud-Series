# Day 38 – Azure VM + Blob Storage Integration

---

## Task Overview

The Nautilus DevOps team needs to set up an Azure Virtual Machine (VM) to interact with an Azure Blob Storage container for storing and retrieving data. The team must create a private storage account, configure Blob Storage, and test the functionality.

**Task:**

**1) Azure Virtual Machine Setup:** 

- The VM named `nautilus-vm` already exists in the **East US** region.

**2) Create a Private Storage Account and Blob Container:**

- Create a storage account named `nautilusstor32123` in the **East US** region with **Locally-redundant storage (LRS)**.
- Create a private Blob container named `nautilus-container32123`.

**3) Retrieve Storage Account Key:**

- Get the storage account's access key to configure access for the application.

**4) Create a Test File:**

- SSH into the VM and create a file named `testfile.txt` in the `/home/azureuser` directory with content: "this is a test file".

**5) Upload the File to Blob Storage:**

- Upload the `testfile.txt` file to the Blob container `nautilus-container32123` using the Azure CLI command:

```bash
az storage blob upload --account-name nautilusstor32123 --account-key <access-key> --container-name nautilus-container32123 --name testfile.txt --file /home/azureuser/testfile.txt
```

`Notes:`

- Create the resources only in the **East US** region.
- Use the Azure Portal or Azure CLI for resource creation.
- Ensure the storage account is private and secure.

---

## Step-by-Step Implementation

### Step 1: Verify Existing VM

On azure-client:

```bash id="p8x4qw"
RG=$(az group list --query "[0].name" -o tsv)

az vm list -o table
```

#### Explanation

Verify that the existing virtual machine `nautilus-vm` is available in the Azure environment.

---

### Step 2: Get VM Public IP Address

```bash id="l4m8nk"
az vm list -d \
--resource-group $RG \
--query "[].{VM:name,PublicIP:publicIps}" \
-o table
```

#### Explanation

Retrieve the public IP address required for SSH access to the virtual machine.

---

### Step 3: Create Storage Account

Navigate to:

```text id="r7t9vx"
Azure Portal
→ Storage Accounts
→ Create
```

Configure:

| Field                | Value                           |
| -------------------- | ------------------------------- |
| Storage Account Name | `nautilusstor32123`             |
| Region               | `East US`                       |
| Performance          | Standard                        |
| Redundancy           | Locally-redundant storage (LRS) |

Click:

```text id="g2n8dm"
Review + Create
→ Create
```

#### Explanation

Creates a new Azure Storage Account with Standard performance and LRS redundancy.

---

### Step 4: Create Private Blob Container

Navigate to:

```text id="w5q1cz"
Storage Account
→ Data Storage
→ Containers
→ + Container
```

Configure:

| Field               | Value                         |
| ------------------- | ----------------------------- |
| Name                | `nautilus-container32123`     |
| Public Access Level | Private (No anonymous access) |

Click:

```text id="m3v9ka"
Create
```

#### Explanation

Creates a private blob container that can only be accessed using authorized credentials.

---

### Step 5: Retrieve Storage Account Key

On azure-client:

```bash id="j6u2ph"
KEY=$(az storage account keys list \
  --resource-group $RG \
  --account-name nautilusstor32123 \
  --query "[0].value" \
  -o tsv)

echo $KEY
```

#### Explanation

Retrieves the storage account access key required for blob operations.

---

### Step 6: Connect to the Virtual Machine

```bash id="f9y5tr"
ssh azureuser@<VM_PUBLIC_IP>
```

#### Explanation

Connect to the existing VM where the test file will be created.

---

### Step 7: Create Test File

Inside the VM:

```bash id="b4k7ne"
echo "this is a test file" > /home/azureuser/testfile.txt
```

Verify:

```bash id="s1m8vw"
cat /home/azureuser/testfile.txt
```

Expected output:

```text id="t6p3rd"
this is a test file
```

#### Explanation

Creates the required file and verifies its contents.

---

### Step 8: Upload File to Blob Storage

```bash id="q2x8lu"
az storage blob upload \
  --account-name nautilusstor32123 \
  --account-key <STORAGE_ACCOUNT_KEY> \
  --container-name nautilus-container32123 \
  --name testfile.txt \
  --file /home/azureuser/testfile.txt
```

#### Explanation

Uploads the file from the virtual machine to Azure Blob Storage.

---

### Step 9: Verify Blob Upload

```bash id="c8r5mt"
az storage blob list \
  --account-name nautilusstor32123 \
  --account-key <STORAGE_ACCOUNT_KEY> \
  --container-name nautilus-container32123 \
  -o table
```

Expected:

```text id="n5z4hx"
testfile.txt
```

#### Explanation

Confirms that the blob exists in the container.

---

### Step 10: Verify Blob Content

Download the blob:

```bash id="v1k9as"
az storage blob download \
  --account-name nautilusstor32123 \
  --account-key <STORAGE_ACCOUNT_KEY> \
  --container-name nautilus-container32123 \
  --name testfile.txt \
  --file downloaded.txt
```

View content:

```bash id="e3m7qy"
cat downloaded.txt
```

Expected:

```text id="y8d2gp"
this is a test file
```

#### Explanation

Validates that the uploaded file content matches the original file.

---

## Best Practices

* Use private containers for storing sensitive application data.
* Store storage account keys securely and avoid exposing them in scripts.
* Verify file contents after upload to ensure data integrity.
* Use LRS storage for cost-effective non-critical workloads.
* Follow the principle of least privilege when granting storage access.

## Key Learnings

* Creating Azure Storage Accounts with LRS redundancy.
* Configuring private Azure Blob Containers.
* Retrieving storage account access keys using Azure CLI.
* Uploading files from Azure VMs to Blob Storage.
* Verifying blob uploads and data consistency.
