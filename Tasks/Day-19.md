# Day 19 – Convert Public Azure Blob Container to Private

---

## Task Overview  

The Nautilus DevOps team has been using Azure Blob Storage to manage their data. Recently, they realized that one of their containers, currently public, needs to be restricted for internal use only. Your task is to convert a public Azure Blob container to private.

Two blob containers named `devops-container-1424` and `devops-priv-29943` are available in the `eastus` region within the storage account `devopsst9454`. The `devops-container-1424` is currently public, and `devops-priv-29943` is private.

- Convert the blob container `devops-container-1424` from public to private while leaving `devops-priv-29943` unchanged.

- Make sure the access level for `devops-container-1424` is set to `private` with no public access.

---

## Step-by-Step Implementation  

### Step 1: Login to Azure Portal  

Open the Azure Portal:

https://portal.azure.com

Login using your Azure account credentials.

#### Explanation  
The Azure Portal provides a centralized interface to manage Azure Storage resources and access settings.

---

### Step 2: Open the Storage Account  

Use the top search bar and search for:

| Storage Account |
|---|
| `devopsst9454` |

Open the Storage Account from the search results.

#### Explanation  
This Storage Account contains the Blob containers that need to be reviewed and modified.

---

### Step 3: Navigate to Blob Containers  

From the left-side menu, open:

- **Data Storage**
- **Containers**

Verify the following containers exist:

| Container Name |
|---|
| `devops-container-1424` |
| `devops-priv-29943` |

#### Explanation  
The Containers section displays all Blob containers available inside the Storage Account.

---

### Step 4: Open the Public Container  

Click:

| Container |
|---|
| `devops-container-1424` |

#### Explanation  
This is the container that currently allows public access and must be converted to private.

---

### Step 5: Change Container Access Level  

At the top of the container page, click:

- **Change access level**

Depending on the Azure Portal version, this option may also appear as:

- **Access policy**

#### Explanation  
This option allows administrators to modify the public access settings of the Blob container.

---

### Step 6: Configure Private Access  

Under **Public access level**, select:

| Setting |
|---|
| Private (no anonymous access) |

or

| Setting |
|---|
| Private (No public access) |

#### Important  
This is the required setting for task completion.

Do not select:

- Blob
- Container

These options allow anonymous access and will fail validation.

#### Explanation  
Private access ensures that only authenticated users with proper permissions can access the container and its contents.

---

### Step 7: Save the Configuration  

Click:

- **Save**

or

- **OK**

Wait for the update operation to complete.

#### Explanation  
Azure updates the container permissions and removes all anonymous access.

---

### Step 8: Verify Container Access Level  

Return to the Containers overview page and verify:

| Container Name | Access Level |
|---|---|
| `devops-container-1424` | Private |
| `devops-priv-29943` | Private |

#### Explanation  
This confirms that the public container has been successfully converted to private while the existing private container remains unchanged.

---

## Best Practices  

- Use private containers for sensitive business and application data  
- Regularly review storage access configurations and permissions  
- Enable public access only when explicitly required by business needs  
- Follow the principle of least privilege for storage resources  
- Monitor storage account access using Azure monitoring tools  
- Periodically audit Blob containers for unintended public exposure  

## Key Learnings  

- Azure Blob containers support Private, Blob, and Container access levels  
- Private containers block all anonymous access to container data  
- Access levels can be modified without recreating the container  
- Storage security is a critical part of cloud infrastructure management  
- Azure Storage provides granular access control for Blob resources  
- Proper access management helps protect data from unauthorized exposure  
