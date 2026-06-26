# Day 42 – Backup and Delete Azure Storage Blob Container

---

## Task Overview

The Nautilus DevOps team is currently engaged in a cleanup process, focusing on removing unnecessary data and services from their Azure environment. As part of the migration process, several resources were created for one-time use only, necessitating a cleanup effort to optimize their Azure environment.

A private blob container named `xfusion-blob-32514` already exists in the `southcentralus` region under storage account `xfusionst30003`.

1) Copy the contents of `xfusion-blob-32514` blob container to the `/opt` directory on the `azure-client` host (the landing host once you load this lab).

2) Delete the blob container `xfusion-blob-32514` from the storage account.

---

## Step-by-Step Implementation

### Step 1: Get the Resource Group

On **azure-client**:

```bash id="c0u5qn"
RG=$(az group list --query "[0].name" -o tsv)

echo $RG
```

#### Explanation

Retrieves the existing Azure Resource Group containing the storage account.

---

### Step 2: Retrieve the Storage Account Key

```bash id="o42mry"
KEY=$(az storage account keys list \
  --resource-group $RG \
  --account-name xfusionst30003 \
  --query "[0].value" \
  -o tsv)

echo $KEY
```

#### Explanation

Obtains the access key required to manage blobs and containers.

---

### Step 3: Verify the Blob Container Exists

```bash id="mjlwmx"
az storage container list \
  --account-name xfusionst30003 \
  --account-key "$KEY" \
  -o table
```

Expected:

```text id="0g9i2l"
Name
-------------------
xfusion-blob-32514
```

#### Explanation

Confirms that the required blob container exists before performing the backup.

---

### Step 4: List Blob Files

```bash id="9dh40x"
az storage blob list \
  --account-name xfusionst30003 \
  --account-key "$KEY" \
  --container-name xfusion-blob-32514 \
  -o table
```

Example:

```text id="qwf6fj"
Name
-----------------
sample.txt
data.zip
```

#### Explanation

Displays all blobs stored inside the container.

---

### Step 5: Download All Blobs to `/opt`

```bash id="n4b7ea"
az storage blob download-batch \
  --account-name xfusionst30003 \
  --account-key "$KEY" \
  --source xfusion-blob-32514 \
  --destination /opt
```

Expected:

```text id="g39vjr"
Finished[############################################] 100%
```

#### Explanation

Downloads every blob from the container into the `/opt` directory on the **azure-client** host.

---

### Step 6: Verify Backup

```bash id="j5u3cx"
ls -lh /opt
```

or

```bash id="r5dr3m"
find /opt -maxdepth 1 -type f
```

#### Explanation

Confirms that all blob files have been successfully copied to `/opt`.

---

### Step 7: Delete the Blob Container

```bash id="1h0qca"
az storage container delete \
  --account-name xfusionst30003 \
  --account-key "$KEY" \
  --name xfusion-blob-32514
```

Expected:

```json id="fxyvki"
{
  "deleted": true
}
```

#### Explanation

Deletes the blob container after confirming that the backup has completed successfully.

---

### Step 8: Verify Container Deletion

```bash id="09z0gl"
az storage container list \
  --account-name xfusionst30003 \
  --account-key "$KEY" \
  -o table
```

The container:

```text id="g5ynlt"
xfusion-blob-32514
```

should **no longer appear**.

#### Explanation

Ensures that the container has been removed successfully.

---

## Complete Azure CLI Method

### Get Resource Group

```bash id="ob8bwr"
RG=$(az group list --query "[0].name" -o tsv)
```

### Get Storage Account Key

```bash id="qzjlwm"
KEY=$(az storage account keys list \
  --resource-group $RG \
  --account-name xfusionst30003 \
  --query "[0].value" \
  -o tsv)
```

### Backup All Blobs

```bash id="rk5lpr"
az storage blob download-batch \
  --account-name xfusionst30003 \
  --account-key "$KEY" \
  --source xfusion-blob-32514 \
  --destination /opt
```

### Delete Container

```bash id="77nyrv"
az storage container delete \
  --account-name xfusionst30003 \
  --account-key "$KEY" \
  --name xfusion-blob-32514
```

---

## Verification Commands

### Verify Backup Files

```bash id="wz4f9r"
ls -lh /opt
```

---

### Verify Container Has Been Deleted

```bash id="65u4eh"
az storage container exists \
  --account-name xfusionst30003 \
  --account-key "$KEY" \
  --name xfusion-blob-32514
```

Expected:

```json id="zhdnha"
{
  "exists": false
}
```

---

### Verify Remaining Containers

```bash id="zqzbfu"
az storage container list \
  --account-name xfusionst30003 \
  --account-key "$KEY" \
  -o table
```

The deleted container should no longer be listed.

---

## Common Errors & Fixes

### Error: ContainerNotFound

```text
ContainerNotFound
```

**Cause**

Incorrect container name.

**Fix**

```bash id="g01nhd"
az storage container list \
  --account-name xfusionst30003 \
  --account-key "$KEY" \
  -o table
```

---

### Error: AuthenticationFailed

```text
AuthenticationFailed
```

**Cause**

Incorrect or expired Storage Account Key.

**Fix**

```bash id="3hnsu7"
KEY=$(az storage account keys list \
  --resource-group $RG \
  --account-name xfusionst30003 \
  --query "[0].value" \
  -o tsv)
```

---

### Error: Destination Directory Does Not Exist

```text
Destination directory does not exist
```

**Fix**

```bash id="nlywoj"
sudo mkdir -p /opt
```

Normally, `/opt` already exists on the Azure client.

---

### Error: Container Already Deleted

```text
ContainerAlreadyDeleted
```

**Fix**

Verify:

```bash id="w03l8o"
az storage container exists \
  --account-name xfusionst30003 \
  --account-key "$KEY" \
  --name xfusion-blob-32514
```

Expected:

```json
{
  "exists": false
}
```

---

## Best Practices

* Always back up blob data before deleting containers.
* Verify downloaded files after backup.
* Use Azure CLI batch operations for faster downloads.
* Confirm container deletion before considering the cleanup complete.
* Store backups securely before removing cloud resources.

## Key Learnings

* Working with Azure Blob Storage.
* Downloading multiple blobs using Azure CLI.
* Managing Storage Account access keys.
* Deleting Blob containers safely.
* Verifying backup and cleanup operations.
