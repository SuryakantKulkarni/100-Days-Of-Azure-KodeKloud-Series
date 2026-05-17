# Day 03 – Create VM using Azure CLI

---

## Task Overview  

The Nautilus DevOps team is in the process of migrating some of their workloads to Azure. One of the tasks involves creating a new Virtual Machine (VM) using the Azure CLI. The team does not have access to the Azure portal but can manage Azure resources via the `azure-client` host (the landing host for this lab). 

- Create a new Azure Virtual Machine named `datacenter-vm` using the Azure CLI. 

- Use the `Ubuntu2204` image and set the VM size to `Standard_B2s`. 

- Make sure the admin username is set to `azureuser` and SSH keys are generated for secure access. 

- Use `Standard_LRS` storage account, disk size must be `30GB` and ensure the VM `datacenter-vm` is in the `running` state after creation.

---

## Step-by-Step Implementation  

### Step 1: Connect to azure-client Host  

Open the lab terminal and connect to the Azure client machine:

```bash
ssh azure-client
```

Verify the hostname:

```bash
hostname
```

Expected output:

```bash
azure-client
```

#### Explanation  
The `azure-client` machine already has Azure CLI installed and configured for managing Azure resources.

### Step 2: Verify Azure CLI Installation  

Run:

```bash
az version
```

#### Explanation  
This command verifies that Azure CLI is installed and available on the system.

### Step 3: Verify Azure Login  

Run:

```bash
az account show
```

#### Explanation  
This command checks whether the Azure account is already authenticated.

### Step 4: Check Available Resource Groups  

Run:

```bash
az group list --output table
```

Example output:

| Name | Location |
|---|---|
| datacenter-rg | southcentralus |

#### Explanation  
This command lists all available Resource Groups in the Azure subscription.

### Step 5: Create the Virtual Machine  

Replace `<RESOURCE_GROUP>` with the actual Resource Group name from the previous step.

Run:

```bash
az vm create \
  --resource-group <RESOURCE_GROUP> \
  --name datacenter-vm \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --admin-username azureuser \
  --generate-ssh-keys \
  --storage-sku Standard_LRS \
  --os-disk-size-gb 30
```

Example:

```bash
az vm create \
  --resource-group datacenter-rg \
  --name datacenter-vm \
  --image Ubuntu2204 \
  --size Standard_B2s \
  --admin-username azureuser \
  --generate-ssh-keys \
  --storage-sku Standard_LRS \
  --os-disk-size-gb 30
```

#### Explanation  
This command creates a new Azure Virtual Machine using Ubuntu 22.04 image with Standard_B2s VM size, Standard_LRS storage, and a 30 GB OS disk.

### Step 6: Wait for Deployment Completion  

Wait for the deployment process to complete successfully.

Expected output:

```json
"powerState": "VM running"
```

#### Explanation  
This confirms that the Virtual Machine was created successfully and is currently running.

### Step 7: Verify VM Running State  

Run:

```bash
az vm show \
  --resource-group <RESOURCE_GROUP> \
  --name datacenter-vm \
  --show-details \
  --query powerState \
  --output tsv
```

Expected output:

```bash
VM running
```

#### Explanation  
This command checks the current power state of the Virtual Machine.

### Step 8: Get Public IP Address  

Run:

```bash
az vm list-ip-addresses \
  --resource-group <RESOURCE_GROUP> \
  --name datacenter-vm \
  --output table
```

#### Explanation  
This command retrieves the public IP address assigned to the Virtual Machine.

### Step 9: Connect to the Virtual Machine using SSH  

Run:

```bash
ssh azureuser@<PUBLIC-IP>
```

Example:

```bash
ssh azureuser@20.xx.xx.xx
```

#### Explanation  
This command establishes a secure SSH connection to the Azure Virtual Machine.

### Step 10: Accept SSH Fingerprint  

When prompted, type: **yes**

#### Explanation  
This adds the server fingerprint to the known hosts list for secure future connections.

### Step 11: Verify Successful Login  

Expected output:

```bash
Welcome to Ubuntu 22.04 LTS
```

#### Explanation  
This confirms successful SSH access to the Azure Virtual Machine.

---

## Best Practices  

- Use Azure CLI for faster and automated infrastructure provisioning  
- Always verify Azure account and Resource Group before deployment  
- Use SSH key authentication instead of passwords for secure VM access  
- Choose VM sizes according to workload and cost requirements  
- Restrict inbound access using only required ports like SSH (22)  
- Verify VM status and connectivity after deployment completion  

## Key Learnings  

- Azure CLI enables complete VM management without using the Azure Portal  
- Azure Virtual Machines can be provisioned quickly using a single CLI command  
- SSH keys provide secure authentication for Linux Virtual Machines  
- Standard_LRS provides cost-effective storage for general workloads  
- VM power state can be verified using Azure CLI commands  
- Public IP addresses are required for remote SSH access to Azure VMs 
