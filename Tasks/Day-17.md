# Day 17 – Create a Public Azure Blob Storage Container

---

## Task Overview  

As part of the data migration process, the Nautilus DevOps team is actively creating several storage containers on Azure. They plan to utilize public Blob containers to store the relevant data. Given the ongoing migration of other infrastructure to Azure, it is logical to consolidate data storage within the Azure environment as well.

Create a new storage account named `devopsst21727` and a `public Blob` container named `devops-blob-7217` within the storage account. Make sure `anonymous read access for containers and blobs` is enabled.

---

## Step-by-Step Implementation  

### Step 1: Login to Azure Portal  

Open the Azure Portal:

https://portal.azure.com

Login using your Azure account credentials.

#### Explanation  
The Azure Portal is Microsoft Azure’s web-based management interface used to create and manage cloud resources.

---

### Step 2: Navigate to Storage Accounts  

Use the top search bar and search for:

| Search |
|---|
| Storage accounts |

Open the **Storage accounts** service from the results.

#### Explanation  
Storage Accounts provide a unique namespace in Azure for storing blobs, files, queues, and tables.

---

### Step 3: Start Creating Storage Account  

Click:

- **+ Create**

#### Explanation  
This starts the process of creating a new Azure Storage Account.

---

### Step 4: Configure Storage Account Details  

Under **Project Details**, configure the following settings:

| Setting | Value |
|---|---|
| Subscription | Default Azure Subscription |
| Resource Group | Existing Resource Group |

Example:

| Resource Group |
|---|
| `datacenter-rg` |

---

Under **Instance Details**, configure:

| Setting | Value |
|---|---|
| Storage Account Name | `devopsst21727` |
| Region | Same as lab resources |
| Performance | Standard |
| Redundancy | Locally-redundant storage (LRS) |

#### Important  
Storage Account names:
- Must be lowercase
- Must be globally unique
- Cannot contain spaces or special characters

#### Explanation  
These settings define the Storage Account configuration and storage redundancy level.

---

### Step 5: Enable Anonymous Blob Access  

Click:

- **Next : Advanced**

Locate:

| Setting | Value |
|---|---|
| Allow Blob Anonymous Access | Enabled |

#### Important  
This setting must be enabled to allow public access to Blob containers.

#### Explanation  
Anonymous Blob Access allows containers and blobs to be accessed without authentication when public access is configured.

---

### Step 6: Review and Create Storage Account  

Click:

- **Review + Create**

After validation passes successfully, click:

- **Create**

Wait for deployment to complete.

#### Explanation  
Azure validates all configuration settings before provisioning the Storage Account.

---

### Step 7: Open Storage Account  

After deployment completes, click:

- **Go to resource**

Verify that you are inside:

| Storage Account |
|---|
| `devopsst21727` |

#### Explanation  
This opens the management page for the newly created Storage Account.

---

### Step 8: Navigate to Containers  

From the left-side menu, open:

- **Data Storage**
- **Containers**

#### Explanation  
The Containers section is used to create and manage Blob Storage containers inside the Storage Account.

---

### Step 9: Create New Blob Container  

Click:

- **+ Container**

#### Explanation  
This starts the process of creating a new Blob Storage container.

---

### Step 10: Configure Container Details  

Configure the following settings:

| Setting | Value |
|---|---|
| Container Name | `devops-blob-7217` |
| Anonymous Access Level | Container (anonymous read access for containers and blobs) |

#### Important  
Do **not** select Private.

The task specifically requires anonymous read access for both containers and blobs.

#### Explanation  
The **Container** access level allows anonymous users to list blobs within the container and read blob data without authentication.

---

### Step 11: Create Container  

Click:

- **Create**

Wait for the container creation process to complete.

#### Explanation  
Azure creates the Blob container with public read access enabled.

---

### Step 12: Verify Container Configuration  

Open the container and verify the following:

| Property | Expected Value |
|---|---|
| Container Name | `devops-blob-7217` |
| Public Access Level | Container |

#### Explanation  
This confirms that the Blob container was created successfully with anonymous read access enabled.

---

## Best Practices  

- Enable public access only when business requirements demand it  
- Use private containers for sensitive or confidential data  
- Follow consistent naming conventions for storage resources  
- Monitor anonymous access usage and storage activity regularly  
- Apply least-privilege access wherever possible  
- Use lifecycle management policies to optimize storage costs  

## Key Learnings  

- Azure Blob Storage is used to store unstructured data at scale  
- Storage Accounts act as containers for Azure storage services  
- Anonymous Blob Access must be enabled at the Storage Account level before public containers can be created  
- Container-level public access allows anonymous read access to containers and blobs  
- Storage Account names must be globally unique across Azure  
- Proper access configuration balances usability and security in cloud storage  
