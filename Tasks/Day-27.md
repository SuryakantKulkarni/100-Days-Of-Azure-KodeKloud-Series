# Day 27 – Deploying Virtual Machines in a Private Virtual Network

---

## Task Overview  

The Nautilus DevOps team is expanding their Azure infrastructure and requires the setup of a private Virtual Network (VNet) along with a subnet. This VNet and subnet configuration will ensure that resources deployed within them remain isolated from external networks and can only communicate within the VNet. Additionally, the team needs to provision a Virtual Machine (VM) under the newly created private VNet. This VM should be accessible over SSH from within the VNet only, allowing for secure communication and resource management within the Azure environment. 

The name of the VNet must be `nautilus-priv-vnet`, create a subnet named `nautilus-priv-subnet` under the same. Further, create a Virtual Machine named `nautilus-priv-vm` under this VNet. Additionally, create a Network Security Group (NSG) named `nautilus-priv-nsg`, and ensure that the NSG rules for the VM allow access only from within the VNet's CIDR block. Ensure all resources are created in the `Central US region.

`Notes:` 

- Create the resources only in the `Central US` region.
- Use the VNet CIDR `10.0.0.0/16` for `nautilus-priv-vnet` in `Central US`.
- Set up an explicit NSG inbound SSH rule on `nautilus-priv-nsg` with the following parameters:
  - Source: `10.0.0.0/16`
  - Destination: `10.0.0.0/16`
  - TCP Port: `22`
  - Action: Allow`.

---

# Step-by-Step Implementation

### Step 1: Create the Virtual Network

Navigate to:

```text
Virtual Networks
→ Create
```

Configure:

| Setting | Value |
|---|---|
| Name | `nautilus-priv-vnet` |
| Region | `Central US` |
| Resource Group | Existing Lab Resource Group |

Click:

```text
Next : IP Addresses
```

#### Explanation

The Virtual Network provides an isolated private network for Azure resources.

---

### Step 2: Configure Address Space

Use:

```text
10.0.0.0/16
```

#### Explanation

This address space allows multiple internal subnets while maintaining network isolation.

---

### Step 3: Create the Private Subnet

Click:

```text
Add Subnet
```

Configure:

| Setting | Value |
|---|---|
| Subnet Name | `nautilus-priv-subnet` |
| Starting Address | `10.0.1.0` |
| Size | `/24` |

Result:

```text
10.0.1.0/24
```

Click:

```text
Add
Review + Create
Create
```

#### Explanation

This subnet will host the private Virtual Machine.

---

### Step 4: Create the Network Security Group

Navigate to:

```text
Network Security Groups
→ Create
```

Configure:

| Setting | Value |
|---|---|
| Name | `nautilus-priv-nsg` |
| Region | `Central US` |
| Resource Group | Same Resource Group |

Click:

```text
Review + Create
Create
```

#### Explanation

The NSG controls network traffic entering and leaving resources.

---

### Step 5: Create Custom SSH Rule

Open:

```text
nautilus-priv-nsg
→ Inbound Security Rules
→ Add
```

Configure:

| Field | Value |
|---|---|
| Source | IP Addresses |
| Source IP Addresses/CIDR | `10.0.0.0/16` |
| Source Port Ranges | * |
| Destination | IP Addresses |
| Destination IP Addresses/CIDR | `10.0.0.0/16` |
| Destination Port Ranges | `22` |
| Protocol | TCP |
| Action | Allow |
| Priority | `300` |
| Name | `Allow-VNet-SSH` |

Click:

```text
Add
```

#### Explanation

This rule permits SSH traffic only from systems inside the Virtual Network.

---

### Step 6: Generate SSH Key on Azure Client

Run:

```bash
ssh-keygen -t rsa -b 2048 -f ~/.ssh/id_rsa -N ""
```

Display the public key:

```bash
cat ~/.ssh/id_rsa.pub
```

#### Explanation

The SSH key enables secure authentication without passwords.

---

### Step 7: Create the Virtual Machine

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
| VM Name | `nautilus-priv-vm` |
| Region | `Central US` |
| Image | Ubuntu Server 22.04 LTS or 24.04 LTS |
| Size | Standard_B1s |
| Username | azureuser |
| Authentication Type | SSH Public Key |

Select:

```text
Use Existing Public Key
```

Paste:

```bash
cat ~/.ssh/id_rsa.pub
```

#### Public Inbound Ports

Select:

```text
None
```

#### Explanation

The VM should not be directly accessible from the internet.

---

### Step 8: Configure Disk Settings

Click:

```text
Next : Disks
```

Select:

| Setting | Value |
|---|---|
| OS Disk Type | Standard HDD (LRS) |

#### Explanation

This avoids storage policy violations in the lab environment.

---

### Step 9: Configure Networking

Click:

```text
Next : Networking
```

Configure:

| Setting | Value |
|---|---|
| Virtual Network | `nautilus-priv-vnet` |
| Subnet | `nautilus-priv-subnet` |
| Public IP | None |
| NIC Network Security Group | Advanced |
| Existing NSG | `nautilus-priv-nsg` |
| Public Inbound Ports | None |

#### Explanation

This keeps the VM private and applies the custom NSG.

---

### Step 10: Review and Create

Verify:

| Component | Expected Value |
|---|---|
| VNet | nautilus-priv-vnet |
| Subnet | nautilus-priv-subnet |
| Public IP | None |
| NSG | nautilus-priv-nsg |
| Disk Type | Standard HDD (LRS) |

Click:

```text
Review + Create
```

After validation succeeds:

```text
Create
```

#### Explanation

Azure validates the configuration and deploys the resources.

---

# Verification Steps

### Step 11: Verify VM Status

Run:

```bash
RG=$(az group list --query "[0].name" -o tsv)

az vm get-instance-view \
  --resource-group $RG \
  --name nautilus-priv-vm \
  --query "instanceView.statuses[].displayStatus" \
  -o table
```

Expected:

```text
Provisioning succeeded
VM running
```

---

### Step 12: Verify No Public IP

Run:

```bash
az vm show \
  --resource-group $RG \
  --name nautilus-priv-vm \
  -d \
  --query publicIps
```

Expected:

```text
null
```

or

```text
(empty)
```

#### Explanation

This confirms that the VM is completely private.

---

### Step 13: Verify NSG Rule

Run:

```bash
az network nsg rule list \
  --resource-group $RG \
  --nsg-name nautilus-priv-nsg \
  -o table
```

Verify:

```text
Allow-VNet-SSH
TCP
22
Allow
10.0.0.0/16
```

#### Explanation

This confirms that SSH access is restricted to the VNet.

---

## Best Practices

- Keep private workloads isolated from the public internet
- Use NSGs to restrict access to only trusted networks
- Disable Public IPs for internal-only resources
- Follow least-privilege network access principles
- Use SSH key authentication instead of passwords
- Validate NSG rules after deployment

## Key Learnings

- Azure VNets provide secure network isolation
- Private subnets prevent direct internet exposure
- NSGs act as virtual firewalls for Azure resources
- SSH access can be restricted to specific CIDR ranges
- Public IPs should be avoided for internal workloads
- Proper network segmentation improves cloud security
