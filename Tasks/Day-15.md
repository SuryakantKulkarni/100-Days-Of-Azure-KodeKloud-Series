# Day 15 – Create and Configure Network Security Group (NSG) in Azure

---

## Task Overview  

The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the Azure cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process.

For this task, create a network security group (NSG) with the following requirements:

- Name of the NSG should be `xfusion-nsg`.

- Add an inbound security rule named `Allow-HTTP` for `HTTP` service on port `80`, with the source CIDR range of `0.0.0.0/0`.

- Add another inbound security rule named `Allow-SSH` for `SSH` service on port `22`, with the source CIDR range of `0.0.0.0/0`.

---

## Step-by-Step Implementation  

### Step 1: Login to Azure Portal  

Open the Azure Portal:

https://portal.azure.com

Login using your Azure account credentials.

#### Explanation  
The Azure Portal is Microsoft Azure’s web-based management interface used to create and manage cloud resources.

### Step 2: Navigate to Network Security Groups  

Use the top search bar and search for:

| Search |
|---|
| Network security groups |

Open the **Network security groups** service from the results.

#### Explanation  
Network Security Groups (NSGs) are used to filter and control inbound and outbound network traffic for Azure resources.

### Step 3: Start Creating Network Security Group  

Click:

- **+ Create**

#### Explanation  
This starts the process of creating a new Network Security Group resource.

### Step 4: Configure Basic Details  

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
| NSG Name | `xfusion-nsg` |
| Region | South Central US |

#### Explanation  
These settings define the Network Security Group name and deployment region.

### Step 5: Review and Create  

Click:

- **Review + Create**

After validation passes successfully, click:

- **Create**

Wait for deployment to complete.

#### Explanation  
Azure validates all configuration settings before provisioning the Network Security Group.

### Step 6: Open the NSG  

After deployment completes, click:

- **Go to resource**

Verify that you are inside:

| Network Security Group |
|---|
| `xfusion-nsg` |

#### Explanation  
This opens the management page for the newly created NSG.

### Step 7: Add HTTP Inbound Security Rule  

From the left-side menu, open:

- **Inbound security rules**

Click:

- **+ Add**

Configure the following settings:

| Field | Value |
|---|---|
| Source | IP Addresses |
| Source IP Addresses/CIDR Ranges | `0.0.0.0/0` |
| Source Port Ranges | `*` |
| Destination | Any |
| Service | Custom |
| Destination Port Ranges | `80` |
| Protocol | TCP |
| Action | Allow |
| Priority | `1000` |
| Name | `Allow-HTTP` |

Click:

- **Add**

#### Explanation  
This rule allows inbound HTTP traffic from any IP address on port 80.

### Step 8: Add SSH Inbound Security Rule  

Again click:

- **+ Add**

Configure the following settings:

| Field | Value |
|---|---|
| Source | IP Addresses |
| Source IP Addresses/CIDR Ranges | `0.0.0.0/0` |
| Source Port Ranges | `*` |
| Destination | Any |
| Service | SSH |
| Destination Port Ranges | `22` |
| Protocol | TCP |
| Action | Allow |
| Priority | `1010` |
| Name | `Allow-SSH` |

Click:

- **Add**

#### Explanation  
This rule allows inbound SSH traffic from any IP address on port 22.

### Step 9: Verify Inbound Security Rules  

Under **Inbound security rules**, verify the following entries:

| Rule Name | Port | Source |
|---|---|---|
| Allow-HTTP | 80 | 0.0.0.0/0 |
| Allow-SSH | 22 | 0.0.0.0/0 |

#### Explanation  
This confirms that both required inbound rules have been configured successfully.

---

## Best Practices  

- Follow the principle of least privilege when creating NSG rules  
- Allow only required ports and protocols for applications  
- Use descriptive names for security rules for easier management  
- Assign unique priorities to avoid rule conflicts  
- Regularly review and audit NSG configurations  
- Restrict SSH access to trusted IP ranges in production environments  

## Key Learnings  

- Network Security Groups control inbound and outbound traffic in Azure  
- NSG rules are processed based on priority values  
- HTTP traffic uses TCP port 80 for web communication  
- SSH traffic uses TCP port 22 for secure remote access  
- NSGs improve network security by filtering traffic at the resource level  
- Proper NSG configuration is essential for secure Azure deployments  
