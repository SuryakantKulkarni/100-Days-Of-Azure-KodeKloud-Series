# Day 32 – Synchronizing Containers Using Azure CLI

---

## Task Overview

As part of a data migration project, the team lead has tasked the team with migrating data from an existing Azure Blob container to a new Blob container. The existing container contains a substantial amount of data that must be accurately transferred to the new container. The team is responsible for creating the new Blob container and ensuring that all data from the existing container is copied or synced to the new container completely and accurately. It is imperative to perform thorough verification steps to confirm that all data has been successfully transferred to the new container without any loss or corruption.

As a member of the Nautilus DevOps Team, your task is to perform the following:

- **Create a New Private Azure Blob Container:** Name the container `nautilus-dest-24685` under the storage account `nautilusst24524`.

- **Data Migration:** Migrate the file `nautilus.txt` from the existing `nautilus-source-27136` container to the new `nautilus-dest-24685` container.

- **Ensure Data Consistency:** Ensure that both containers have the file `nautilus.txt` and confirm the file content is identical in both containers.

- **Use Azure CLI:** Use the Azure CLI to perform the creation and data migration tasks.

---

# Step-by-Step Implementation

### Step 1: Verify Azure Login

Confirm Azure CLI authentication.

```bash
az account show --output table
```

#### Explanation

This verifies that Azure CLI is authenticated and ready to access Azure resources.

---

### Step 2: Identify the Resource Group

Retrieve the available Resource Group.

```bash
RG=$(az group list --query "[0].name" -o tsv)

echo $RG
```

#### Explanation

The Resource Group is required to access the storage account and retrieve authentication keys.

---

### Step 3: Retrieve Storage Account Key

Get the access key for the storage account.

```bash
ACCOUNT_KEY=$(az storage account keys list \
  --resource-group $RG \
  --account-name nautilusst24524 \
  --query "[0].value" -o tsv)
```

Verify:

```bash
echo $ACCOUNT_KEY
```

#### Explanation

The storage account key is required for Blob container and Blob object operations.

---

### Step 4: Create the Destination Container

Create the new private Blob container.

```bash
az storage container create \
  --name nautilus-dest-24685 \
  --account-name nautilusst24524 \
  --account-key $ACCOUNT_KEY \
  --public-access off
```

#### Explanation

This creates the destination container with private access, preventing anonymous access.

---

### Step 5: Verify Container Creation

List available containers.

```bash
az storage container list \
  --account-name nautilusst24524 \
  --account-key $ACCOUNT_KEY \
  --output table
```

Expected output:

```text
nautilus-source-27136
nautilus-dest-24685
```

#### Explanation

This confirms that the destination container was successfully created.

---

### Step 6: Verify Source Blob Exists

Check the contents of the source container.

```bash
az storage blob list \
  --container-name nautilus-source-27136 \
  --account-name nautilusst24524 \
  --account-key $ACCOUNT_KEY \
  --output table
```

Expected output:

```text
nautilus.txt
```

#### Explanation

Before copying, confirm that the source file exists.

---

### Step 7: Copy the Blob to the Destination Container

Start the Blob copy operation.

```bash
az storage blob copy start \
  --account-name nautilusst24524 \
  --account-key $ACCOUNT_KEY \
  --destination-container nautilus-dest-24685 \
  --destination-blob nautilus.txt \
  --source-account-name nautilusst24524 \
  --source-container nautilus-source-27136 \
  --source-blob nautilus.txt
```

#### Explanation

This copies the existing Blob from the source container to the destination container without modifying the source file.

---

### Step 8: Verify Copy Status

Check whether the copy operation completed successfully.

```bash
az storage blob show \
  --container-name nautilus-dest-24685 \
  --name nautilus.txt \
  --account-name nautilusst24524 \
  --account-key $ACCOUNT_KEY \
  --query properties.copy.status
```

Expected output:

```text
success
```

#### Explanation

A status of `success` confirms that the file was copied successfully.

---

### Step 9: Verify File Exists in Both Containers

Verify the source container.

```bash
az storage blob list \
  --container-name nautilus-source-27136 \
  --account-name nautilusst24524 \
  --account-key $ACCOUNT_KEY \
  --output table
```

Verify the destination container.

```bash
az storage blob list \
  --container-name nautilus-dest-24685 \
  --account-name nautilusst24524 \
  --account-key $ACCOUNT_KEY \
  --output table
```

Expected output in both containers:

```text
nautilus.txt
```

#### Explanation

This confirms that both containers now contain the required file.

---

### Step 10: Download Source File

Download the source Blob locally.

```bash
az storage blob download \
  --container-name nautilus-source-27136 \
  --name nautilus.txt \
  --file /tmp/source.txt \
  --account-name nautilusst24524 \
  --account-key $ACCOUNT_KEY
```

#### Explanation

This creates a local copy of the source file for comparison.

---

### Step 11: Download Destination File

Download the copied Blob locally.

```bash
az storage blob download \
  --container-name nautilus-dest-24685 \
  --name nautilus.txt \
  --file /tmp/dest.txt \
  --account-name nautilusst24524 \
  --account-key $ACCOUNT_KEY
```

#### Explanation

This creates a local copy of the destination file for verification.

---

### Step 12: Verify Data Consistency

Compare both files.

```bash
diff /tmp/source.txt /tmp/dest.txt
```

Expected output:

```text
(no output)
```

#### Explanation

If the `diff` command returns no output, both files are identical and the migration was successful.

---

## Final Verification Commands

Verify containers:

```bash
az storage container list \
  --account-name nautilusst24524 \
  --account-key $ACCOUNT_KEY \
  --output table
```

Verify source file:

```bash
az storage blob list \
  --container-name nautilus-source-27136 \
  --account-name nautilusst24524 \
  --account-key $ACCOUNT_KEY \
  --output table
```

Verify destination file:

```bash
az storage blob list \
  --container-name nautilus-dest-24685 \
  --account-name nautilusst24524 \
  --account-key $ACCOUNT_KEY \
  --output table
```

Verify file integrity:

```bash
diff /tmp/source.txt /tmp/dest.txt
```

---

## Best Practices

- Always verify source data before starting migration operations
- Use private Blob containers for sensitive or internal data
- Validate copy status after Blob transfers complete
- Compare file contents after migration to ensure integrity
- Use storage account keys securely and avoid exposing them in logs
- Perform post-migration verification before deleting source data

## Key Learnings

- Azure Blob Storage supports container-to-container data migration
- Azure CLI provides efficient Blob management and automation capabilities
- Blob copy operations preserve source files during migration
- Private containers prevent anonymous access to stored data
- File integrity validation is essential after migration tasks
- Azure storage operations can be fully automated using CLI commands
