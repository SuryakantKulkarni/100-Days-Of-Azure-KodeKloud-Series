# Day 18 – Copy Data to an Azure Blob Storage Container

---

## Task Overview  

The Nautilus DevOps team is presently immersed in data migrations, transferring data from on-premise storage systems to Azure Blob containers. They have recently received some data that they intend to copy to one of the Blob containers.

A Blob container named `nautilus-blob-17567` already exists in the `southcentralus` region under the storage account `nautilusst5299`. Copy the file `/tmp/nautilus.txt` to the Blob container `nautilus-blob-17567`.

---

## Step-by-Step Implementation  

### Step 1: Connect to Azure Client Host  

Open the lab terminal and connect to the Azure client machine.

Verify the hostname:

```bash
hostname
```

Expected output:

```bash
azure-client
```

#### Explanation  
The Azure client machine contains the required file and Azure CLI tools needed to complete the task.

---

### Step 2: Verify the Source File Exists  

Run:

```bash
ls -l /tmp/nautilus.txt
```

Expected output:

```bash
-rw-r--r-- 1 root root ...
```

Optionally verify file contents:

```bash
cat /tmp/nautilus.txt
```

#### Explanation  
This confirms that the file exists and is available for upload.

---

### Step 3: Verify Azure Login  

Run:

```bash
az account show
```

#### Explanation  
This command confirms that the Azure CLI session is authenticated and connected to the correct subscription.

---

### Step 4: Upload File to Blob Container  

Run:

```bash
az storage blob upload \
  --account-name nautilusst5299 \
  --container-name nautilus-blob-17567 \
  --name nautilus.txt \
  --file /tmp/nautilus.txt \
  --auth-mode login
```

#### Explanation  
This command uploads the local file from the Azure client machine to the specified Blob container.

---

### Step 5: Wait for Upload Completion  

Expected output:

```json
{
  "etag": "...",
  "lastModified": "...",
  "requestId": "...",
  "versionId": "...",
  "contentMd5": "..."
}
```

#### Explanation  
This indicates that the blob was uploaded successfully to Azure Storage.

---

### Step 6: Verify Blob Upload  

Run:

```bash
az storage blob list \
  --account-name nautilusst5299 \
  --container-name nautilus-blob-17567 \
  --output table \
  --auth-mode login
```

Expected output:

```text
Name
------------
nautilus.txt
```

#### Explanation  
This command lists all blobs inside the container and confirms that the file was uploaded successfully.

---

### Step 7: Verify Blob from Azure Portal (Optional)  

Open Azure Portal and navigate to:

- Storage Accounts
- `nautilusst5299`
- Containers
- `nautilus-blob-17567`

Verify that:

| Blob Name |
|---|
| `nautilus.txt` |

is visible inside the container.

#### Explanation  
This provides visual confirmation that the uploaded file exists in Azure Blob Storage.

---

## Best Practices  

- Verify source files before starting data migration tasks  
- Use Azure CLI for efficient and repeatable storage operations  
- Follow consistent naming conventions for storage resources and blobs  
- Validate uploads after data transfer completion  
- Use appropriate access controls on storage accounts and containers  
- Monitor storage usage and data transfer activities regularly  

## Key Learnings  

- Azure Blob Storage is designed for storing unstructured data at scale  
- Azure CLI can be used to upload and manage blob data efficiently  
- Blob containers act as logical storage units within Storage Accounts  
- File uploads can be verified using Azure CLI and Azure Portal  
- Authentication is required to perform storage operations securely  
- Proper validation ensures successful and reliable data migration processes  
