# Day 05 – Create a Virtual Network (IPv4) in Azure

---

## Task Overview  

The Nautilus DevOps team is strategically planning the migration of a portion of their infrastructure to the Azure cloud. Acknowledging the magnitude of this endeavor, they have chosen to tackle the migration incrementally rather than as a single, massive transition. Their approach involves creating Virtual Networks (VNets) as the initial step, as they will be provisioning various services under different VNets. 

Create a Virtual Network (VNet) named `datacenter-vnet` in the `eastus` region with `192.168.0.0/24` IPv4 CIDR.

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
| Name | `datacenter-vnet` |
| Region | East US |

#### Important  
Select **East US** only. Do not select **East US 2**.

#### Explanation  
These settings define the Virtual Network name and deployment region.

### Step 5: Configure IP Addresses  

Click:

- **Next : IP Addresses**

Under **IPv4 address space**, remove the default CIDR block if present and add:

| Setting | Value |
|---|---|
| IPv4 CIDR | `192.168.0.0/24` |

#### Explanation  
The IPv4 CIDR block defines the private IP address range available inside the Virtual Network.

### Step 6: Verify Default Subnet  

Azure automatically creates a default subnet.

Example configuration:

| Setting | Value |
|---|---|
| Subnet Name | default |
| Subnet CIDR | 192.168.0.0/26 |

Keep the default subnet configuration unchanged.

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
| Name | `datacenter-vnet` |
| Region | East US |
| Address Space | `192.168.0.0/24` |

#### Explanation  
This confirms that the Virtual Network was created successfully with the required configuration.

---

## Best Practices  

- Plan CIDR ranges carefully to avoid IP address conflicts  
- Use separate VNets for different environments such as Dev, Test, and Production  
- Organize Azure resources using Resource Groups  
- Keep subnet structures simple and scalable  
- Apply NSG rules to secure network traffic  
- Use meaningful naming conventions for Azure networking resources  

## Key Learnings  

- Azure VNets provide isolated private networking for Azure resources  
- CIDR blocks define the IP address range available inside a Virtual Network  
- Subnets help organize and segment network traffic efficiently  
- Azure automatically creates default networking components during deployment  
- VNets are foundational building blocks of Azure cloud infrastructure  
- Proper network planning improves security, scalability, and manageability  
