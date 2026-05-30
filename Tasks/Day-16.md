# Day 16 – Create a Private Azure Blob Storage Container

---

## Task Overview  

As part of the data migration process, the Nautilus DevOps team is actively creating several storage containers on Azure. They plan to utilize private Blob containers to store the relevant data. Given the ongoing migration of other infrastructure to Azure, it is logical to consolidate data storage within the Azure environment as well.

Create a new storage account named `xfusionst6988` and a `private` Blob container named `xfusion-blob-28309` within the storage account. 

---

## Step-by-Step Implementation  

### Step 1: Login to Azure Portal  

Open the Azure Portal:

https://portal.azure.com

Login using your Azure account credentials.

#### Explanation  
The Azure Portal is Microsoft Azure’s web-based management interface used to create and manage cloud resources.

### Step 2: Navigate to Storage Accounts  

Use the top search bar and search for:

| Search |
|---|
| Storage accounts |

Open the **Storage accounts** service from the results.

#### Explanation  
Storage Accounts provide a unique namespace in Azure for storing blobs, files, queues, and tables.

### Step 3: Start Creating Storage Account  

Click:

- **+ Create**

#### Explanation  
This starts the process of creating a new Azure Storage Account.

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

Under **Instance Details**, configure:

| Setting | Value |
|---|---|
| Storage Account Name | `xfusionst6988` |
| Region | Same as lab resources |
| Performance | Standard |
| Redundancy | Locally-redundant storage (LRS) |

#### Important  
Storage Account names:
- Must be lowercase
- Must be globally unique
- Cannot contain special characters or spaces

#### Explanation  
These settings define the Storage Account configuration and storage redundancy level.

### Step 5: Review and Create Storage Account  

Click:

- **Review + Create**

After validation passes successfully, click:

- **Create**

Wait for deployment to complete.

#### Explanation  
Azure validates all configuration settings before provisioning the Storage Account.

### Step 6: Open Storage Account  

After deployment completes, click:

- **Go to resource**

Verify that you are inside:

| Storage Account |
|---|
| `xfusionst6988` |

#### Explanation  
This opens the management page for the newly created Storage Account.

### Step 7: Navigate to Containers  

From the left-side menu, open:

- **Data Storage**
- **Containers**

#### Explanation  
The Containers section is used to create and manage Blob Storage containers inside the Storage Account.

### Step 8: Create New Blob Container  

Click:

- **+ Container**

#### Explanation  
This starts the process of creating a new Blob Storage container.

### Step 9: Configure Container Details  

Configure the following settings:

| Setting | Value |
|---|---|
| Container Name | `xfusion-blob-28309` |
| Anonymous Access Level | Private (no anonymous access) |

#### Important  
Select **Private (no anonymous access)** because the task specifically requires a private Blob container.

#### Explanation  
Private containers prevent anonymous users from accessing stored blob data.

### Step 10: Create Container  

Click:

- **Create**

Wait for the container creation process to complete.

#### Explanation  
Azure creates the Blob container inside the Storage Account with the configured access settings.

### Step 11: Verify Container Configuration  

Under the **Containers** section, verify the following:

| Property | Expected Value |
|---|---|
| Container Name | `xfusion-blob-28309` |
| Access Level | Private |

#### Explanation  
This confirms that the Blob container was created successfully with private access enabled.

---

## Best Practices  

- Use private containers for sensitive application and infrastructure data  
- Follow consistent naming conventions for storage resources  
- Choose appropriate redundancy options based on business requirements  
- Enable access controls and identity-based authentication whenever possible  
- Organize data using separate containers for different workloads  
- Regularly monitor storage usage and access activity
  
## Key Learnings  

- Azure Storage Accounts provide scalable cloud storage services  
- Blob Containers are used to store unstructured data such as files and backups  
- Private containers restrict anonymous access to stored data  
- Storage Account names must be globally unique across Azure  
- LRS provides locally redundant storage within a single Azure region  
- Proper storage configuration improves security and data management  
