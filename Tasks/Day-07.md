# Day 07 – Create a Public IP Address for Azure VM

---

## Task Overview  

The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the Azure cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.

For this task, allocate a `Public IP` address, name it as `datacenter-pip`. 

---

## Step-by-Step Implementation  

### Step 1: Login to Azure Portal  

Open the Azure Portal:

https://portal.azure.com

Login using your Azure account credentials.

#### Explanation  
The Azure Portal is Microsoft Azure’s web-based management interface used to create and manage cloud resources.

### Step 2: Navigate to Public IP Addresses  

Use the top search bar and search for:

| Search |
|---|
| Public IP addresses |

Open the **Public IP addresses** service from the results.

#### Explanation  
The Public IP Addresses service is used to allocate and manage public-facing IP addresses for Azure resources.

### Step 3: Start Creating Public IP Address  

Click:

- **+ Create**

#### Explanation  
This starts the process of creating a new Public IP Address resource.

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
| Region | South Central US |
| Name | `datacenter-pip` |
| SKU | Standard |
| IP Version | IPv4 |
| Assignment | Static |

#### Explanation  
These settings define the Public IP configuration including IP version and allocation method.

### Step 5: Review and Create  

Click:

- **Review + Create**

After validation passes successfully, click:

- **Create**

Wait for deployment to complete.

#### Explanation  
Azure validates all configuration settings before provisioning the Public IP resource.

### Step 6: Verify Public IP Address  

After deployment completes, click:

- **Go to resource**

Verify the following details:

| Property | Expected Value |
|---|---|
| Name | `datacenter-pip` |
| IP Version | IPv4 |
| Assignment | Static |

#### Explanation  
This confirms that the Public IP Address was created successfully with the required configuration.

---

## Best Practices  

- Use Static Public IPs for production workloads requiring consistent connectivity  
- Allocate Public IPs only when internet access is necessary  
- Restrict inbound traffic using NSG rules for better security  
- Use meaningful naming conventions for networking resources  
- Monitor Public IP usage and associated resources regularly  
- Avoid exposing unnecessary services directly to the internet  

## Key Learnings  

- Public IP Addresses allow Azure resources to communicate over the internet  
- Azure supports both Dynamic and Static Public IP allocation methods  
- IPv4 is commonly used for external network communication  
- Public IPs can be associated with VMs, Load Balancers, and Gateways  
- Standard SKU provides enhanced security and availability features  
- Proper Public IP management improves network security and connectivity  
