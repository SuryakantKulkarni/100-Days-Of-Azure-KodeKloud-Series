# Day 02 – Create an Azure Virtual Machine

---

## Task Overview  
The Nautilus DevOps team is planning to migrate a portion of their infrastructure to the Azure cloud incrementally. As part of this migration, you are tasked with creating an Azure Virtual Machine (VM). 

The requirements are: 

- Use the existing resource group.

- The VM name must be `xfusion-vm`, it should be in `southcentralus` region.

- Use the `Ubuntu 24.04 LTS` image for the VM.

- The VM size must be `Standard_B1s`.

- Attach a default Network Security Group (NSG) that allows inbound SSH (port 22).

- Attach a 30 GB storage disk of type `Standard HDD`.

- The rest of the configurations should remain as default.

After completing these steps, make sure you can SSH into the virtual machine.

---

# Method 1: Using Azure Portal

## Step 1: Login to Azure Portal  

Open the Azure Portal:

https://portal.azure.com

Login using your Azure account credentials.

#### Explanation  
The Azure Portal is Microsoft Azure’s web-based management interface used to create and manage cloud resources.

## Step 2: Navigate to Virtual Machines  

Use the top search bar and search for:

| Search |
|---|
| Virtual machines |

Open the **Virtual machines** service from the results.

#### Explanation  
The Virtual Machines service is used to create, manage, monitor, and access Azure Virtual Machines.

## Step 3: Start Creating Virtual Machine  

Click:

- **+ Create**
- **Azure virtual machine**

#### Explanation  
This starts the process of provisioning a new Azure Virtual Machine.

## Step 4: Configure Basics Tab  

Under **Project Details**, configure the following settings:

| Setting | Value |
|---|---|
| Subscription | Default Azure Subscription |
| Resource Group | Existing Resource Group |

#### Explanation  
The Resource Group acts as a logical container used to organize Azure resources together.

Under **Instance Details**, configure:

| Setting | Value |
|---|---|
| Virtual Machine Name | `xfusion-vm` |
| Region | South Central US |
| Availability Options | Default |
| Image | Ubuntu Server 24.04 LTS |
| VM Architecture | x64 |
| Size | Standard_B1s |

To select VM size:

- Click **See all sizes**
- Search for `Standard_B1s`
- Click **Select**

#### Explanation  
These settings define the VM name, operating system, region, and compute configuration.

## Step 5: Configure Administrator Account  

Under **Administrator Account**, configure:

| Setting | Value |
|---|---|
| Authentication Type | SSH public key |
| Username | `azureuser` |
| SSH Public Key Source | Generate new key pair |
| Key Pair Name | `datacenter-kp` |

#### Explanation  
SSH key authentication provides secure passwordless access to Linux Virtual Machines.

## Step 6: Configure Inbound Ports  

Under **Public inbound ports**, select:

| Setting | Value |
|---|---|
| Public inbound ports | Allow selected ports |
| Select inbound ports | SSH (22) |

#### Explanation  
This automatically creates a Network Security Group (NSG) rule allowing SSH access on port 22.

## Step 7: Configure Disks  

Click:

- **Next : Disks**

Configure the following:

| Setting | Value |
|---|---|
| OS Disk Type | Standard HDD (LRS) |
| Disk Size | 30 GiB |

#### Explanation  
The OS disk stores the operating system files and VM data. Standard HDD provides cost-effective storage.

## Step 8: Configure Networking  

Click:

- **Next : Networking**

Keep all networking configurations as default.

Ensure the following settings are configured:

| Setting | Value |
|---|---|
| NIC Network Security Group | Basic |
| Public Inbound Ports | Allow selected ports |
| SSH Port | Enabled |

#### Explanation  
Azure automatically creates required networking resources including Virtual Network, Subnet, Public IP, and NSG.

## Step 9: Review and Create  

Click:

- **Review + Create**

After validation passes successfully, click:

- **Create**

#### Explanation  
Azure validates all configurations before provisioning the Virtual Machine.

## Step 10: Download Private Key  

If Azure generates a new SSH key pair, download the private key file securely.

Example:

| File |
|---|
| `datacenter-kp.pem` |

#### Important  
The private key file can only be downloaded once.

#### Explanation  
The `.pem` file is required for securely connecting to the Virtual Machine using SSH.

## Step 11: Connect to Virtual Machine  

After deployment completes:

- Open the Virtual Machine
- Click **Connect**
- Select **SSH**

Azure displays the SSH connection command.

Example:

```bash
ssh -i datacenter-kp.pem azureuser@<public-ip>
```

#### Explanation  
This command establishes a secure SSH connection to the Azure Virtual Machine.

## Step 12: Configure Key Permission  

Before connecting from Linux or macOS systems, run:

```bash
chmod 400 datacenter-kp.pem
```

Then connect using:

```bash
ssh -i datacenter-kp.pem azureuser@<public-ip>
```

#### Explanation  
The `chmod 400` command secures the private key by restricting file permissions.

## Expected Output  

After successful login, the following message appears:

```bash
Welcome to Ubuntu 24.04 LTS
```

Task completed successfully ✅

---

# Method 2: Using Azure CLI  

## Step 1: Login to Azure  

Run:

```bash
az login
```

#### Explanation  
This command authenticates Azure CLI with your Azure account.

## Step 2: Create Virtual Machine  

Run:

```bash
az vm create \
  --resource-group <resource-group-name> \
  --name xfusion-vm \
  --image Ubuntu2404 \
  --size Standard_B1s \
  --location southcentralus \
  --admin-username azureuser \
  --generate-ssh-keys \
  --storage-sku Standard_LRS
```

#### Explanation  
This command creates an Azure Virtual Machine with Ubuntu 24.04 LTS image, Standard_B1s size, and Standard HDD storage.

## Step 3: Open SSH Port  

Run:

```bash
az vm open-port \
  --resource-group <resource-group-name> \
  --name xfusion-vm \
  --port 22
```

#### Explanation  
This command creates an NSG rule allowing inbound SSH traffic on port 22.

## Step 4: Get Public IP Address  

Run:

```bash
az vm list-ip-addresses \
  --resource-group <resource-group-name> \
  --name xfusion-vm \
  --output table
```

#### Explanation  
This command retrieves the public IP address assigned to the Virtual Machine.

## Step 5: Connect Using SSH  

Run:

```bash
ssh azureuser@<public-ip>
```

#### Explanation  
This command securely connects to the Azure Virtual Machine using SSH.

---

## Best Practices  

- Use SSH key authentication instead of passwords for better security  
- Store `.pem` private key files securely and never share them publicly  
- Allow only required inbound ports in NSG rules  
- Use properly organized Resource Groups for infrastructure management  
- Choose VM sizes based on workload requirements to optimize costs  
- Regularly monitor and maintain Virtual Machine resources  

## Key Learnings  

- Azure Virtual Machines provide scalable cloud infrastructure  
- SSH keys enable secure access to Linux Virtual Machines  
- NSGs help control and secure network traffic  
- Azure automatically creates required networking resources  
- Azure CLI simplifies infrastructure automation and management  
- Proper VM configuration improves security and performance  
