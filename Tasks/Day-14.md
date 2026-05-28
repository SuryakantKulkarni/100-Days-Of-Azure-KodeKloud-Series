# Day 14 – Create and Attach Managed Disks in Azure

---

## Task Overview  

The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the Azure cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.

Create a managed disk with the following requirements:

- Name of the disk should be `xfusion-disk`.

- Disk `type` must be `Standard_LRS`.

- Disk `size` must be `2 GiB`.

---

## Step-by-Step Implementation  

### Step 1: Login to Azure Portal  

Open the Azure Portal:

https://portal.azure.com

Login using your Azure account credentials.

#### Explanation  
The Azure Portal is Microsoft Azure’s web-based management interface used to create and manage cloud resources.

### Step 2: Navigate to Disks  

Use the top search bar and search for:

| Search |
|---|
| Disks |

Open the **Disks** service from the results.

#### Explanation  
The Disks service is used to create and manage Azure Managed Disks for Virtual Machines and other services.

### Step 3: Start Creating Managed Disk  

Click:

- **+ Create**

#### Explanation  
This starts the process of creating a new Managed Disk resource.

### Step 4: Configure Basics Tab  

Under **Project Details**, configure the following settings:

| Setting | Value |
|---|---|
| Subscription | Default Azure Subscription |
| Resource Group | Existing Resource Group |

Example:

| Resource Group |
|---|
| `datacenter-rg` |

#### Explanation  
A Resource Group is a logical container used to organize Azure resources together.

Under **Instance Details**, configure:

| Setting | Value |
|---|---|
| Disk Name | `xfusion-disk` |
| Region | South Central US |
| Availability Zone | Default |

#### Explanation  
These settings define the Managed Disk name, deployment region, and availability configuration.

### Step 5: Configure Source Type 

Under **Source type**, select:

| Setting | Value |
|---|---|
| Source Type | `None` |

#### Explanation  
Selecting `None` creates an empty managed disk without using snapshots or existing disks as a source.

### Step 6: Configure Disk Size  

Under **Size + performance**, configure:

| Setting | Value |
|---|---|
| Custom Disk Size | `2 GiB` |

#### Explanation  
This defines the storage capacity allocated to the Managed Disk.

### Step 7: Configure Disk Type  

Under **Storage type**, select:

| Setting | Value |
|---|---|
| Storage Type | Standard HDD (locally-redundant storage) |

This corresponds to:

| Azure SKU |
|---|
| `Standard_LRS` |

#### Explanation  
Standard_LRS provides cost-effective locally redundant storage suitable for general workloads.

### Step 8: Review and Create  

Click:

- **Review + Create**

After validation passes successfully, click:

- **Create**

Wait for deployment to complete.

#### Explanation  
Azure validates all configuration settings before provisioning the Managed Disk resource.

### Step 9: Verify Managed Disk  

After deployment completes, click:

- **Go to resource**

Verify the following details:

| Property | Expected Value |
|---|---|
| Name | `xfusion-disk` |
| Size | `2 GiB` |
| SKU | Standard HDD LRS |

#### Explanation  
This confirms that the Managed Disk was created successfully with the required configuration.

---

## Best Practices  

- Use managed disks for better scalability and reliability  
- Choose disk types according to workload performance requirements  
- Keep disk naming conventions clear and consistent  
- Use separate data disks instead of storing all data on OS disks  
- Monitor disk utilization and performance regularly  
- Use cost-optimized storage types for non-critical workloads  

## Key Learnings  

- Azure Managed Disks provide persistent block-level storage  
- Standard_LRS offers locally redundant storage for general workloads  
- Empty managed disks can be created using Source Type = None  
- Managed disks can later be attached to Azure Virtual Machines  
- Disk size determines the storage capacity available for workloads  
- Proper storage planning improves scalability and infrastructure management  
