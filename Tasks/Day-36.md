# Day 36 – Managing Storage Lifecycle in Azure

---

## Task Overview

The Nautilus DevOps team needs to optimize data retention costs by automating the deletion of old blobs. They plan to implement Blob Lifecycle Management for a specific container in Azure Storage.

**Task:**

**1) Create a Storage Account:**

- Name the storage account `nautilusstor5801`.
- Set the **region** to **East US**.
- Use **Locally-redundant storage (LRS)** as the redundancy option.
  
**2) Create a Blob Container:**

- Name the container `nautilus-container5801`.

**3) Upload a File to the Container:**

- Upload the file named `tempfile.txt` to the container. The file is present under `/root` of the client host.

**4) Configure Blob Lifecycle Management:**

- Apply a **Lifecycle Management** rule named `nautilus-del-rule` to the container `nautilus-container5801` to delete blobs after `7` days of last modification.

**5) Validation:**

- Verify that the Lifecycle Management rule named `nautilus-del-rule` is correctly applied.

---

## Step-by-Step Implementation

### Step 1: Verify Resource Group

```bash
RG=$(az group list --query "[0].name" -o tsv)

echo $RG
```

#### Explanation

Retrieve the resource group that will be used for creating Azure resources.

---

### Step 2: Create Storage Account

Navigate to:

```text
Azure Portal
→ Storage Accounts
→ Create
```

Configure:

| Field                | Value                           |
| -------------------- | ------------------------------- |
| Storage Account Name | `nautilusstor5801`              |
| Region               | `East US`                       |
| Performance          | Standard                        |
| Redundancy           | Locally-redundant storage (LRS) |

#### Explanation

Creates an Azure Storage Account using Standard performance and LRS redundancy.

---

### Step 3: Create Blob Container

Navigate to:

```text
Storage Account
→ Data Storage
→ Containers
→ + Container
```

Configure:

| Field        | Value                    |
| ------------ | ------------------------ |
| Name         | `nautilus-container5801` |
| Access Level | Private                  |

#### Explanation

Creates a private blob container for storing application files.

---

### Step 4: Verify File Exists

On azure-client:

```bash
ls -l /root/tempfile.txt
```

#### Explanation

Ensures the file required for upload exists on the client host.

---

### Step 5: Upload File Using Azure CLI

Get Storage Account Key:

```bash
KEY=$(az storage account keys list \
--resource-group $RG \
--account-name nautilusstor5801 \
--query "[0].value" -o tsv)
```

Upload the file:

```bash
az storage blob upload \
  --account-name nautilusstor5801 \
  --account-key $KEY \
  --container-name nautilus-container5801 \
  --name tempfile.txt \
  --file /root/tempfile.txt
```

#### Explanation

Uploads `tempfile.txt` from the azure-client host to the blob container.

---

### Step 6: Verify Upload

```bash
az storage blob list \
  --account-name nautilusstor5801 \
  --account-key $KEY \
  --container-name nautilus-container5801 \
  -o table
```

#### Explanation

Confirms that `tempfile.txt` exists in the container.

---

### Step 7: Configure Lifecycle Management

Navigate to:

```text
Storage Account
→ Data Management
→ Lifecycle Management
→ Add Rule
```

Configure:

| Field           | Value               |
| --------------- | ------------------- |
| Rule Name       | `nautilus-del-rule` |
| Scope           | Apply to all blobs  |
| Base Blobs Were | Last Modified       |
| More Than       | 7 Days Ago          |
| Action          | Delete Blob         |

#### Explanation

Creates a lifecycle rule that automatically deletes blobs after 7 days of last modification.

---

### Step 8: Save the Rule

Click:

```text
Add
```

Wait until the rule is created successfully.

#### Explanation

Activates the lifecycle management policy.

---

### Step 9: Verify Lifecycle Rule

Navigate to:

```text
Storage Account
→ Lifecycle Management
```

Verify:

```text
Rule Name = nautilus-del-rule
Status = Enabled
```

#### Explanation

Confirms that the lifecycle policy is active.

---

### Step 10: CLI Verification

```bash
az storage account management-policy show \
  --account-name nautilusstor5801 \
  --resource-group $RG
```

#### Explanation

Displays the lifecycle policy and confirms the delete action is configured for 7 days.

---

## Best Practices

* Use Lifecycle Management to reduce long-term storage costs.
* Apply lifecycle policies only to required containers.
* Verify uploaded blobs before applying deletion rules.
* Use LRS storage for cost-effective workloads.
* Regularly audit lifecycle policies and retention settings.

## Key Learnings

* Azure Blob Lifecycle Management automates data retention.
* Storage policies can delete blobs based on last modification time.
* Azure CLI can be used for lifecycle policy creation and verification.
* Blob containers can be managed securely using private access.
* Lifecycle policies help optimize storage costs without manual intervention.
