# Day 06 – Create a Subnet in Azure Virtual Network

---

## Task Overview  

The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the Azure cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. 

For this task, create a Virtual Network (VNet) named `datacenter-vnet` and one subnet named `datacenter-subnet` within the VNet in the `southcentralus` region. Make sure the `IPv4 address range` is `10.0.0.0/16`.

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
| Virtual Network Name | `datacenter-vnet` |
| Region | South Central US |

#### Important  
Select **South Central US** only. Do not select Central US or East US.

#### Explanation  
These settings define the Virtual Network name and deployment region.

### Step 5: Configure IP Addresses  

Click:

- **Next : IP Addresses**

Under **IPv4 address space**, remove any default CIDR block if present and add:

| Setting | Value |
|---|---|
| IPv4 CIDR | `10.0.0.0/16` |

#### Explanation  
The IPv4 CIDR block defines the private IP address range available inside the Virtual Network.

### Step 6: Configure Subnet  

Under the subnet section, Azure may automatically create a default subnet.

Click the existing subnet:

- **default**

Update the subnet configuration:

| Setting | Value |
|---|---|
| Subnet Name | `datacenter-subnet` |
| Subnet CIDR | `10.0.0.0/24` |

Click:

- **Save**

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
Azure validates all configuration settings before provisioning the Virtual Network and subnet.

### Step 10: Verify Virtual Network  

After deployment completes, click:

- **Go to resource**

Verify the following details:

| Property | Expected Value |
|---|---|
| VNet Name | `datacenter-vnet` |
| Region | South Central US |
| Address Space | `10.0.0.0/16` |

#### Explanation  
This confirms that the Virtual Network was created successfully with the required configuration.

### Step 11: Verify Subnet  

Inside the Virtual Network page:

- Open **Subnets** from the left menu

Verify the subnet details:

| Property | Expected Value |
|---|---|
| Subnet Name | `datacenter-subnet` |
| Address Range | `10.0.0.0/24` |

#### Explanation  
This confirms that the subnet was created successfully inside the Virtual Network.

---

## Best Practices  

- Plan subnet CIDR ranges carefully to avoid overlapping IP addresses  
- Use separate subnets for different application or environment tiers  
- Keep Virtual Network naming conventions clear and consistent  
- Organize Azure resources using Resource Groups  
- Apply NSG rules to secure subnet traffic  
- Design network architecture with scalability in mind  

## Key Learnings  

- Azure VNets provide isolated private networking for cloud resources  
- Subnets help organize and segment network traffic efficiently  
- CIDR blocks define the IP address ranges available inside VNets and subnets  
- Azure automatically creates default subnet configurations during deployment  
- Proper subnet planning improves security and resource management  
- Virtual Networks are foundational components of Azure infrastructure  
