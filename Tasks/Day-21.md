# Day 21 – Assigning Public IP to Virtual Machines

---

## Task Overview  

The Nautilus DevOps Team has received a new request from the Development Team to set up a new Azure Virtual Machine (VM). This VM will be used to host a new application that requires a stable public IP address. To ensure that the VM has a consistent public IP, a Static Public IP address needs to be associated with it. The VM will be named `xfusion-vm`, and the Static Public IP will be named `xfusion-pip`. This setup will help the Development Team to have a reliable and consistent access point for their application. 

- Create an Azure VM named `xfusion-vm` using any available Ubuntu image, with the VM size `Standard_B1s`. 

- Generate an SSH public key on the `azure-client` host and associate it with the VM for SSH access. 

Associate a Static Public IP address named `xfusion-pip` with this VM. 

Ensure the VM is accessible via SSH using the generated public key.

---

## Step-by-Step Implementation  

### Step 1: Connect to Azure Client Host  

Verify that you are on the Azure client machine:

```bash
hostname
```

Expected output:

```bash
azure-client
```

#### Explanation  
The Azure client host is used to generate SSH keys and manage Azure resources.

---

### Step 2: Generate SSH Key Pair  

Run:

```bash
ssh-keygen -t rsa -b 2048 -f ~/.ssh/id_rsa -N ""
```

If prompted:

```text
Overwrite (y/n)?
```

Enter:

```text
y
```

#### Explanation  
This command generates an RSA SSH key pair that will be used to securely access the Virtual Machine.

---

### Step 3: Display the Public Key  

Run:

```bash
cat ~/.ssh/id_rsa.pub
```

Example output:

```bash
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQ...
```

Copy the complete public key.

#### Explanation  
The public key will be associated with the Azure Virtual Machine for SSH authentication.

---

### Step 4: Create Static Public IP Address  

In Azure Portal, search for:

| Search |
|---|
| Public IP addresses |

Click:

- **+ Create**

Configure the following settings:

| Setting | Value |
|---|---|
| Name | `xfusion-pip` |
| SKU | Standard |
| Assignment | Static |
| IP Version | IPv4 |

Click:

- **Review + Create**
- **Create**

#### Explanation  
A Static Public IP ensures the VM always retains the same public IP address.

---

### Step 5: Navigate to Virtual Machines  

Search for:

| Search |
|---|
| Virtual Machines |

Click:

- **+ Create**
- **Azure Virtual Machine**

#### Explanation  
This starts the Virtual Machine deployment process.

---

### Step 6: Configure Basic VM Settings  

Under **Basics**, configure:

| Setting | Value |
|---|---|
| Resource Group | Existing Lab Resource Group |
| Virtual Machine Name | `xfusion-vm` |
| Region | Same as Resource Group |
| Availability Options | No infrastructure redundancy required |
| Image | Ubuntu Server 22.04 LTS (or any Ubuntu image) |
| Size | `Standard_B1s` |

#### Explanation  
These settings define the Virtual Machine name, operating system, and compute resources.

---

### Step 7: Configure Authentication  

Under **Administrator Account**, configure:

| Setting | Value |
|---|---|
| Authentication Type | SSH Public Key |
| Username | `azureuser` |
| SSH Public Key Source | Use Existing Public Key |
| Public Key | Paste copied SSH public key |

Under **Inbound Port Rules**, select:

| Setting | Value |
|---|---|
| Public Inbound Ports | Allow Selected Ports |
| Select Inbound Ports | SSH (22) |

#### Explanation  
This allows secure SSH access using the generated key pair.

---

### Step 8: Configure Storage Settings  

Click:

- **Next : Disks**

Configure:

| Setting | Value |
|---|---|
| OS Disk Type | Standard HDD (LRS) |

#### Important  
Use **Standard HDD (LRS)** to comply with lab deployment policies.

#### Explanation  
This defines the storage type used for the Virtual Machine operating system disk.

---

### Step 9: Configure Networking  

Click:

- **Next : Networking**

Under **Public IP**, select:

| Setting | Value |
|---|---|
| Public IP | `xfusion-pip` |

Verify:

| Setting | Value |
|---|---|
| SSH (22) | Enabled |

#### Explanation  
This associates the Static Public IP Address with the Virtual Machine network interface.

---

### Step 10: Review and Create  

Click:

- **Review + Create**

After validation succeeds, click:

- **Create**

Wait for deployment to complete successfully.

#### Explanation  
Azure validates all configurations and provisions the Virtual Machine.

---

### Step 11: Verify Virtual Machine Configuration  

After deployment completes, click:

- **Go to Resource**

Verify:

| Property | Expected Value |
|---|---|
| VM Name | `xfusion-vm` |
| Size | `Standard_B1s` |
| Public IP | `xfusion-pip` |
| Status | Running |

#### Explanation  
This confirms that the Virtual Machine and Public IP have been created successfully.

---

### Step 12: Verify SSH Connectivity  

Copy the Public IP Address from the VM Overview page.

Run:

```bash
ssh azureuser@<PUBLIC-IP>
```

Example:

```bash
ssh azureuser@20.xx.xx.xx
```

If prompted:

```text
Are you sure you want to continue connecting?
```

Type:

```text
yes
```

Expected output:

```bash
azureuser@xfusion-vm:~$
```

#### Explanation  
This confirms successful SSH authentication using the generated SSH key.

---

## Best Practices  

- Use SSH key authentication instead of passwords for secure access  
- Allocate Static Public IPs for workloads requiring consistent connectivity  
- Use least privilege access for VM administration  
- Choose VM sizes based on workload and cost requirements  
- Restrict inbound traffic using NSG rules whenever possible  
- Verify SSH connectivity immediately after deployment  

## Key Learnings  

- Azure Virtual Machines can be secured using SSH key authentication  
- Static Public IP addresses provide consistent external connectivity  
- Azure Public IP resources can be associated during VM deployment  
- SSH keys offer stronger security than password-based authentication  
- VM networking and storage settings can be customized during deployment  
- Proper VM provisioning ensures secure and reliable cloud infrastructure  
