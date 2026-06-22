# Day 39 – Deploying a Static Website Using Azure Storage

---

## Task Overview

The Nautilus DevOps team has been tasked with creating an internal information portal for public access. As part of this project, they need to host a static website on Azure using an Azure Storage account. The Storage account must be configured for public access to allow external users to access the static website directly via the Azure Storage URL.

Task Requirements:

- Create an Azure Storage account named `nautiluswebst7959` in an existing resource group.
- Configure the Storage account for static website hosting with `index.html` as the index document.
- Allow public access to the static website so that the website is publicly accessible.
- Upload the `index.html` file from the `/root/` directory of the Azure client host to the Storage account's `$web` container.
- Verify that the website is accessible directly through the Azure Storage static website URL.

`Notes:`

- Create the resources only in the **East US** region.
- Use the Azure Storage account's `$web` container to host the static website files.

---

## Step-by-Step Implementation

### Step 1: Verify Resource Group

On **azure-client**:

```bash
RG=$(az group list --query "[0].name" -o tsv)

echo $RG
```

#### Explanation

Retrieves the existing resource group that will be used for deploying the storage account.

---

### Step 2: Verify Source File Exists

```bash
ls -l /root/index.html
```

#### Explanation

Confirms that the website file provided by the lab exists before uploading it to Azure Storage.

---

### Step 3: Create Storage Account (Azure Portal)

Navigate to:

```text
Azure Portal
→ Storage Accounts
→ Create
```

Configure:

| Field                | Value                           |
| -------------------- | ------------------------------- |
| Resource Group       | Existing RG                     |
| Storage Account Name | `nautiluswebst7959`             |
| Region               | `East US`                       |
| Performance          | Standard                        |
| Redundancy           | Locally-redundant storage (LRS) |

Under **Networking**:

```text
Public endpoint (all networks)
```

Click:

```text
Review + Create
→ Create
```

#### Explanation

Creates a new storage account in East US with Standard LRS redundancy.

---

### Step 4: Enable Static Website Hosting

Navigate to:

```text
Storage Account
→ Data Management
→ Static Website
```

Configure:

| Setting             | Value       |
| ------------------- | ----------- |
| Static Website      | Enabled     |
| Index Document Name | index.html  |
| Error Document Path | Leave Blank |

Click:

```text
Save
```

#### Explanation

Enables Azure Static Website Hosting and automatically creates the `$web` container.

---

### Step 5: Verify Static Website Endpoint

After enabling static website hosting, Azure displays:

```text
Primary Endpoint
```

Example:

```text
https://nautiluswebst7959.z13.web.core.windows.net/
```

#### Explanation

This endpoint is the public URL used to access the hosted website.

---

### Step 6: Retrieve Storage Account Key

On **azure-client**:

```bash
KEY=$(az storage account keys list \
  --resource-group $RG \
  --account-name nautiluswebst7959 \
  --query "[0].value" \
  -o tsv)

echo $KEY
```

#### Explanation

Retrieves the storage account access key required for blob uploads.

---

### Step 7: Upload index.html to $web Container

```bash
az storage blob upload \
  --account-name nautiluswebst7959 \
  --account-key $KEY \
  --container-name '$web' \
  --name index.html \
  --file /root/index.html \
  --overwrite
```

Expected:

```text
Finished[#############################################################] 100.0000%
```

#### Explanation

Uploads the website file into the `$web` container, making it available through the static website endpoint.

---

### Step 8: Verify Uploaded File

```bash
az storage blob list \
  --account-name nautiluswebst7959 \
  --account-key $KEY \
  --container-name '$web' \
  -o table
```

Expected:

```text
Name
----------
index.html
```

#### Explanation

Confirms that the file was uploaded successfully.

---

### Step 9: Retrieve Website URL

```bash
az storage account show \
  --resource-group $RG \
  --name nautiluswebst7959 \
  --query "primaryEndpoints.web" \
  -o tsv
```

Example:

```text
https://nautiluswebst7959.z13.web.core.windows.net/
```

#### Explanation

Displays the public static website URL.

---

### Step 10: Verify Website Accessibility

Open the URL in a browser:

```text
https://nautiluswebst7959.z13.web.core.windows.net/
```

Or test using CLI:

```bash
curl $(az storage account show \
  --resource-group $RG \
  --name nautiluswebst7959 \
  --query "primaryEndpoints.web" \
  -o tsv)
```

#### Explanation

Verifies that the website is publicly accessible and serving the uploaded `index.html`.

---

## Complete Azure CLI Method

### Create Storage Account

```bash
az storage account create \
  --name nautiluswebst7959 \
  --resource-group $RG \
  --location eastus \
  --sku Standard_LRS
```

### Retrieve Storage Key

```bash
KEY=$(az storage account keys list \
  --resource-group $RG \
  --account-name nautiluswebst7959 \
  --query "[0].value" \
  -o tsv)
```

### Enable Static Website

```bash
az storage blob service-properties update \
  --account-name nautiluswebst7959 \
  --account-key $KEY \
  --static-website \
  --index-document index.html
```

### Upload Website File

```bash
az storage blob upload \
  --account-name nautiluswebst7959 \
  --account-key $KEY \
  --container-name '$web' \
  --name index.html \
  --file /root/index.html \
  --overwrite
```

---

## Verification Commands

### Verify Storage Account

```bash
az storage account show \
  --resource-group $RG \
  --name nautiluswebst7959 \
  --query "{Name:name,Location:location}" \
  -o table
```

### Verify Static Website Status

```bash
az storage blob service-properties show \
  --account-name nautiluswebst7959 \
  --account-key $KEY \
  --query staticWebsite
```

Expected:

```json
{
  "enabled": true,
  "indexDocument": "index.html"
}
```

### Verify $web Container

```bash
az storage container list \
  --account-name nautiluswebst7959 \
  --account-key $KEY \
  -o table
```

Expected:

```text
$web
```

### Verify Uploaded File

```bash
az storage blob list \
  --account-name nautiluswebst7959 \
  --account-key $KEY \
  --container-name '$web' \
  -o table
```

Expected:

```text
index.html
```

---

## Best Practices

* Enable static website hosting only when public access is required.
* Keep website files inside the `$web` container.
* Use LRS storage for low-cost static website hosting.
* Verify uploaded content before sharing the public URL.
* Use Azure CLI for automation and repeatable deployments.

## Key Learnings

* Creating Azure Storage Accounts.
* Configuring Azure Static Website Hosting.
* Working with the `$web` container.
* Uploading website content using Azure CLI.
* Retrieving and validating static website endpoints.
* Hosting publicly accessible static websites without a virtual machine.
