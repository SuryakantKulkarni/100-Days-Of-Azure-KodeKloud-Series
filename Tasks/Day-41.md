# Day 41 – Working with Azure Table Storage

---

## Task Overview

The Nautilus DevOps team is developing a simple 'To-Do' application using Azure Table Storage to store and manage tasks efficiently. The team needs to create an Azure Table to hold tasks, each identified by a unique `taskId`. Each task will have a description and a status, which indicates the progress of the task (e.g., 'completed' or 'in-progress').

Your task is to:

- Create an Azure Storage Account named `xfusiontablest3283` with a Table Storage table called `tasks`.

- Insert the following tasks into the table:

  - Task 1: PartitionKey: 'tasks', RowKey: '1', description: 'Learn Table Storage', status: 'completed'

  - Task 2: PartitionKey: 'tasks', RowKey: '2', description: 'Build To-Do App', status: 'in-progress'

- Verify that Task 1 has a status of 'completed' and Task 2 has a status of 'in-progress'.

`Note:` Use the Azure CLI to insert these tasks into the table.

---

## Step-by-Step Implementation

### Step 1: Get the Resource Group

On **azure-client**:

```bash
RG=$(az group list --query "[0].name" -o tsv)

echo $RG
```

#### Explanation

Retrieves the existing resource group where the Storage Account will be created.

---

### Step 2: Create Storage Account

```bash
az storage account create \
  --name xfusiontablest3283 \
  --resource-group $RG \
  --location eastus \
  --sku Standard_LRS
```

#### Explanation

Creates a new Azure Storage Account using **Standard Locally Redundant Storage (LRS)** in the **East US** region.

---

### Step 3: Retrieve Storage Account Key

```bash
KEY=$(az storage account keys list \
  --resource-group $RG \
  --account-name xfusiontablest3283 \
  --query "[0].value" \
  -o tsv)

echo $KEY
```

#### Explanation

Retrieves the storage account access key required to perform Table Storage operations.

---

### Step 4: Create Table Storage Table

```bash
az storage table create \
  --name tasks \
  --account-name xfusiontablest3283 \
  --account-key "$KEY"
```

Expected Output:

```json
{
  "created": true
}
```

#### Explanation

Creates a Table Storage table named **tasks**.

---

### Step 5: Insert Task 1

```bash
az storage entity insert \
  --table-name tasks \
  --account-name xfusiontablest3283 \
  --account-key "$KEY" \
  --entity PartitionKey=tasks RowKey=1 description="Learn Table Storage" status="completed"
```

#### Explanation

Inserts the first task into the table.

| Property     | Value               |
| ------------ | ------------------- |
| PartitionKey | tasks               |
| RowKey       | 1                   |
| Description  | Learn Table Storage |
| Status       | completed           |

---

### Step 6: Verify Task 1

```bash
az storage entity show \
  --table-name tasks \
  --account-name xfusiontablest3283 \
  --account-key "$KEY" \
  --partition-key tasks \
  --row-key 1
```

Expected Output:

```json
{
  "description": "Learn Table Storage",
  "status": "completed"
}
```

#### Explanation

Confirms that Task 1 has been inserted successfully.

---

### Step 7: Insert Task 2

```bash
az storage entity insert \
  --table-name tasks \
  --account-name xfusiontablest3283 \
  --account-key "$KEY" \
  --entity PartitionKey=tasks RowKey=2 description="Build To-Do App" status="in-progress"
```

#### Explanation

Adds the second task into the table.

| Property     | Value           |
| ------------ | --------------- |
| PartitionKey | tasks           |
| RowKey       | 2               |
| Description  | Build To-Do App |
| Status       | in-progress     |

---

### Step 8: Verify Task 2

```bash
az storage entity show \
  --table-name tasks \
  --account-name xfusiontablest3283 \
  --account-key "$KEY" \
  --partition-key tasks \
  --row-key 2
```

Expected Output:

```json
{
  "description": "Build To-Do App",
  "status": "in-progress"
}
```

#### Explanation

Verifies that Task 2 has been inserted correctly.

---

## Azure Portal Verification (Optional)

Navigate to:

```text
Azure Portal
→ Storage Accounts
→ xfusiontablest3283
→ Storage Browser
→ Tables
→ tasks
```

You should see:

| PartitionKey | RowKey | Description         | Status      |
| ------------ | ------ | ------------------- | ----------- |
| tasks        | 1      | Learn Table Storage | completed   |
| tasks        | 2      | Build To-Do App     | in-progress |

> **Note:** Some Azure lab subscriptions do not display Table Storage entities in the Azure Portal. In such cases, Azure CLI verification is sufficient.

---

## Verification Commands

### Verify Storage Account

```bash
az storage account show \
  --name xfusiontablest3283 \
  --resource-group $RG \
  --query "{Name:name,Location:location,SKU:sku.name}" \
  -o table
```

Expected:

```text
Name                  Location   SKU
--------------------  ---------  ------------
xfusiontablest3283    eastus     Standard_LRS
```

---

### Verify Table Exists

```bash
az storage table list \
  --account-name xfusiontablest3283 \
  --account-key "$KEY" \
  -o table
```

Expected:

```text
tasks
```

---

### Verify Task 1

```bash
az storage entity show \
  --table-name tasks \
  --account-name xfusiontablest3283 \
  --account-key "$KEY" \
  --partition-key tasks \
  --row-key 1 \
  --query "{Description:description,Status:status}"
```

Expected:

```json
{
  "Description": "Learn Table Storage",
  "Status": "completed"
}
```

---

### Verify Task 2

```bash
az storage entity show \
  --table-name tasks \
  --account-name xfusiontablest3283 \
  --account-key "$KEY" \
  --partition-key tasks \
  --row-key 2 \
  --query "{Description:description,Status:status}"
```

Expected:

```json
{
  "Description": "Build To-Do App",
  "Status": "in-progress"
}
```

---

## Common Errors & Fixes

### Error: ResourceNotFound

```text
The specified resource does not exist.
```

**Cause:**

Attempted to retrieve an entity before inserting it.

**Fix:**

Insert the entity using:

```bash
az storage entity insert ...
```

---

### Error: Authentication Failed

```text
AuthenticationFailed
```

**Cause:**

Incorrect or expired Storage Account Key.

**Fix:**

Retrieve a new key:

```bash
KEY=$(az storage account keys list \
  --resource-group $RG \
  --account-name xfusiontablest3283 \
  --query "[0].value" \
  -o tsv)
```

---

### Error: Table Already Exists

```text
TableAlreadyExists
```

**Cause:**

The table has already been created.

**Fix:**

No action required. Continue with entity insertion.

---

## Best Practices

* Use a meaningful **PartitionKey** to logically group related entities.
* Ensure every entity has a unique **RowKey**.
* Store frequently queried entities within the same partition for better performance.
* Keep sensitive information out of Table Storage unless properly encrypted.
* Use Azure CLI for automation and repeatable deployments.

## Key Learnings

* Creating Azure Storage Accounts.
* Working with Azure Table Storage.
* Creating tables using Azure CLI.
* Inserting entities into tables.
* Retrieving entities using PartitionKey and RowKey.
* Verifying stored data using Azure CLI.
