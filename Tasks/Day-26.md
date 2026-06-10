# Day 26 – Deploying Virtual Machines in a Public Virtual Network

---

## Task Overview  

The Nautilus DevOps Team has received a request from the Networking Team to set up a new public VNet to support a set of public-facing services. This VNet will host various resources that need to be accessible over the internet. As part of this setup, you need to ensure the VNet has public subnets with automatic public IP assignment for resources. Additionally, a new VM will be launched within this VNet to host public applications that require SSH access. This setup will enable the Networking Team to deploy and manage public-facing applications. 

Create a public VNet named `xfusion-pub-vnet`, and a subnet named `xfusion-pub-subnet` under the same, make sure public IP is being auto-assigned to resources under this subnet. Further, create a VM named `xfusion-pub-vm` under this VNet. Make sure SSH port `22` is open for this instance and accessible over the internet. Use the Azure portal to complete the task and ensure that SSH access is configured correctly.

---

# Common Errors Encountered During Implementation

## Error 1: Subnet Address Overlap

### Error Message

```text
Address prefix 10.0.0.0/24 overlaps with existing subnet
```

### Cause

Azure automatically creates a default subnet using:

```text
10.0.0.0/24
```

Creating another subnet with the same range causes an overlap conflict.

### Solution

Use:

| Setting | Value |
|---|---|
| Subnet Name | `xfusion-pub-subnet` |
| Address Range | `10.0.1.0/24` |

This avoids subnet overlap issues.

---

## Error 2: VM Deployment Failed Due to Policy

### Error Message

```text
RequestDisallowedByPolicy
This storage configuration is not allowed
```

### Cause

Azure attempted to create the VM using:

```text
Premium SSD LRS
```

KodeKloud lab policies block Premium SSD storage.

### Solution

During VM creation select:

| Setting | Value |
|---|---|
| OS Disk Type | Standard HDD (LRS) |

Never use:

```text
Premium SSD LRS
```

---

## Error 3: SSH Authentication Failed

### Error Message

```bash
ssh azureuser@PUBLIC_IP

Permission denied (publickey)
```

### Cause

The VM was created using:

```text
Generate New Key Pair
```

Azure generated a key that was not available on the `azure-client` machine.

### Solution

Generate the SSH key on `azure-client` before creating the VM:

```bash
ssh-keygen -t rsa -b 2048 -f ~/.ssh/id_rsa -N ""
```

Display the public key:

```bash
cat ~/.ssh/id_rsa.pub
```

During VM creation select:

| Setting | Value |
|---|---|
| Authentication Type | SSH Public Key |
| SSH Public Key Source | Use Existing Public Key |

Paste the output from:

```bash
cat ~/.ssh/id_rsa.pub
```

---

# Step-by-Step Implementation

### Step 1: Generate SSH Key on Azure Client

Connect to the `azure-client` host and verify whether an SSH key already exists:

```bash
ls ~/.ssh
```

If the key does not exist, create one:

```bash
ssh-keygen -t rsa -b 2048 -f ~/.ssh/id_rsa -N ""
```

Display the public key:

```bash
cat ~/.ssh/id_rsa.pub
```

Keep this key ready for VM creation.

#### Explanation

The SSH public key will be used for secure, password-less authentication to the Virtual Machine.

---

### Step 2: Create the Virtual Network

In Azure Portal navigate to:

```text
Virtual Networks
→ Create
```

Configure:

| Setting | Value |
|---|---|
| Name | `xfusion-pub-vnet` |
| Region | East US (or Resource Group Region) |

Click:

```text
Next : IP Addresses
```

#### Explanation

The Virtual Network provides network connectivity for Azure resources.

---

### Step 3: Configure Address Space

Under Address Space configure:

```text
10.0.0.0/16
```

#### Explanation

This provides sufficient IP space for future resources and subnets.

---

### Step 4: Create Public Subnet

Click:

```text
Add Subnet
```

Configure:

| Setting | Value |
|---|---|
| Subnet Name | `xfusion-pub-subnet` |
| Starting Address | `10.0.1.0` |
| Subnet Size | `/24` |

Click:

```text
Add
```

Then:

```text
Review + Create
→ Create
```

#### Explanation

This subnet will host publicly accessible resources including the Virtual Machine.

---

### Step 5: Create the Virtual Machine

Navigate to:

```text
Virtual Machines
→ Create
→ Azure Virtual Machine
```

#### Basics Tab

Configure:

| Setting | Value |
|---|---|
| VM Name | `xfusion-pub-vm` |
| Region | Same as VNet |
| Image | Ubuntu Server 22.04 LTS or 24.04 LTS |
| Size | Standard_B1s |
| Username | azureuser |

#### Authentication

Configure:

| Setting | Value |
|---|---|
| Authentication Type | SSH Public Key |
| SSH Public Key Source | Use Existing Public Key |

Paste the key copied from:

```bash
cat ~/.ssh/id_rsa.pub
```

#### Inbound Ports

Configure:

| Setting | Value |
|---|---|
| Public Inbound Ports | Allow Selected Ports |
| Selected Ports | SSH (22) |

#### Explanation

These settings create a Linux VM with secure SSH access.

---

### Step 6: Configure Disk Settings

Click:

```text
Next : Disks
```

Select:

| Setting | Value |
|---|---|
| OS Disk Type | Standard HDD (LRS) |

#### Important

Do not use:

```text
Premium SSD LRS
```

#### Explanation

This avoids Azure policy violations in the lab environment.

---

### Step 7: Configure Networking

Click:

```text
Next : Networking
```

Configure:

| Setting | Value |
|---|---|
| Virtual Network | `xfusion-pub-vnet` |
| Subnet | `xfusion-pub-subnet` |
| Public IP | Create New |
| NIC Network Security Group | Basic |
| Public Inbound Ports | Allow Selected Ports |
| Selected Ports | SSH (22) |

#### Explanation

This associates a Public IP Address and enables SSH access from the internet.

---

### Step 8: Review and Create

Verify:

| Setting | Value |
|---|---|
| VM Name | xfusion-pub-vm |
| Disk Type | Standard HDD (LRS) |
| Public IP | Enabled |
| SSH | Enabled |

Click:

```text
Review + Create
```

After validation succeeds:

```text
Create
```

#### Explanation

Azure validates the configuration and deploys the Virtual Machine.

---

# Verification Steps

### Step 9: Verify VM Status

Run:

```bash
RG=$(az group list --query "[0].name" -o tsv)

az vm get-instance-view \
  --resource-group $RG \
  --name xfusion-pub-vm \
  --query "instanceView.statuses[].displayStatus" \
  -o table
```

Expected output:

```text
Provisioning succeeded
VM running
```

---

### Step 10: Verify Public IP

Run:

```bash
az vm show \
  --resource-group $RG \
  --name xfusion-pub-vm \
  -d \
  --query "{name:name,publicIp:publicIps}" \
  -o table
```

Expected output:

```text
xfusion-pub-vm
<public-ip>
```

---

### Step 11: Verify NSG Rule

List NSGs:

```bash
az network nsg list -o table
```

Verify SSH rule:

```bash
az network nsg rule list \
  --resource-group $RG \
  --nsg-name <NSG_NAME> \
  -o table
```

Expected output:

```text
SSH
Allow
TCP
Inbound
22
```

---

### Step 12: Verify SSH Connectivity

Connect to the VM:

```bash
ssh azureuser@<PUBLIC-IP>
```

Expected output:

```bash
azureuser@xfusion-pub-vm:~$
```

#### Explanation

Successful login confirms that the Public IP, NSG rule, and SSH authentication are configured correctly.

---

## Best Practices

- Generate SSH keys on the client machine before VM creation
- Use Standard HDD (LRS) in restricted lab environments
- Plan subnet ranges carefully to avoid overlap issues
- Allow only required inbound ports through NSGs
- Verify Public IP assignment before testing connectivity
- Test SSH access immediately after deployment

## Key Learnings

- Azure Virtual Networks provide network isolation and connectivity
- Public subnets enable internet-facing workloads
- NSGs control inbound and outbound traffic rules
- SSH key authentication is more secure than passwords
- Azure policy restrictions can affect VM deployment options
- Proper network planning prevents subnet conflicts and deployment failures
