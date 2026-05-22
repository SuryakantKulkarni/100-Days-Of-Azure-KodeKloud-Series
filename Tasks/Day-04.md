# Day 04 – Create a Virtual Network (VNet) in Azure

---

## Task Overview  

The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the Azure cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations.

Create a Virtual Network (VNet) named `xfusion-vnet` in the `southcentralus` region with any `IPv4` CIDR block.

---

## Step-by-Step Implementation  

### Step 1: Login to Azure Portal  

Open the Azure Portal:

https://portal.azure.com

Login using your Azure account credentials.

#### Explanation  
The Azure Portal is Microsoft Azure’s web-based management interface used to create and manage cloud resources.

### Step 2: Navigate to Virtual Networks  

Use the top search bar and search for:

| Search |
|---|
| Virtual networks |

Open the **Virtual networks** service from the results.

#### Explanation  
The Virtual Networks service is used to create and manage private networks for Azure resources.

### Step 3: Start Creating Virtual Network  

Click:

- **+ Create**

#### Explanation  
This starts the process of creating a new Azure Virtual Network.

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
| Name | `xfusion-vnet` |
| Region | South Central US |

#### Explanation  
These settings define the Virtual Network name and deployment region.

### Step 5: Configure IP Addresses  

Click:

- **Next : IP Addresses**

Under **IPv4 address space**, configure:

| Setting | Value |
|---|---|
| IPv4 CIDR | `10.0.0.0/16` |

#### Explanation  
The IPv4 CIDR block defines the private IP address range available inside the Virtual Network.

### Step 6: Verify Default Subnet  

Azure automatically creates a default subnet.

Example configuration:

| Setting | Value |
|---|---|
| Subnet Name | default |
| Subnet CIDR | 10.0.0.0/24 |

Keep the default configuration unchanged.

#### Explanation  
Subnets divide the Virtual Network into smaller network segments for better organization and traffic management.

### Step 7: Configure Security Tab  

Click:

- **Next : Security**

Keep all settings as default.

#### Explanation  
Default security settings are sufficient for this task.

### Step 8: Configure Tags Tab  

Click:

- **Next : Tags**

No changes are required.

#### Explanation  
Tags are optional metadata labels used for resource organization and billing management.

### Step 9: Review and Create  

Click:

- **Review + Create**

After validation passes successfully, click:

- **Create**

Wait for deployment to complete.

#### Explanation  
Azure validates all configuration settings before provisioning the Virtual Network.

### Step 10: Verify Virtual Network  

After deployment completes, click:

- **Go to resource**

Verify the following details:

| Property | Expected Value |
|---|---|
| Name | `xfusion-vnet` |
| Region | South Central US |
| Address Space | `10.0.0.0/16` |

#### Explanation  
This confirms that the Virtual Network was created successfully with the required configuration.

---

## Best Practices  

- Use properly planned CIDR ranges to avoid IP address conflicts  
- Organize Azure resources using Resource Groups  
- Use separate subnets for different application tiers  
- Keep network configurations simple and scalable  
- Apply NSG rules to secure subnet traffic  
- Use meaningful naming conventions for Azure resources  

## Key Learnings  

- Azure Virtual Networks provide isolated private networking for Azure resources  
- CIDR blocks define the IP address range available inside a VNet  
- Subnets help organize and segment network traffic efficiently  
- Azure automatically creates default networking components during deployment  
- VNets are foundational components of Azure cloud infrastructure  
- Proper network planning improves scalability, security, and manageability  
