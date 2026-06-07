# Day 24 – Securing Virtual Machine SSH Access

---

## Task Overview  

The Nautilus DevOps team needs to set up a new Virtual Machine (VM) on the Azure cloud that can be accessed securely from their landing host (`azure-client`). Follow the steps below to complete this task: 

- **Create an SSH Key:** On the `azure-client` host, check if an SSH key already exists. If it doesn’t exist, create a new SSH key on the `azure-client` host that will be used for password-less SSH access.

- **Create a Virtual Machine:** Use the Azure Portal or Azure CLI to create a new Virtual Machine named `nautilus-vm` in the `westus` region. Set the VM size to **Standard_B1s** and configure the VM with **SSH access** for the `azureuser` account using the newly created SSH key.

- **Configure SSH Access:** Ensure that the SSH key from the `azure-client` host is added to the `azureuser` account on `nautilus-vm`, enabling secure, password-less SSH access from the `azure-client` host.

- **Verify Connectivity:** Test the connection from `azure-client` to `nautilus-vm` using SSH to confirm that password-less access has been set up correctly.

Complete these tasks entirely within the Azure Portal or Azure CLI.

---

## Step-by-Step Implementation  

### Step 1: Verify Current Host  

Run:

```bash
hostname
```

Expected output:

```bash
azure-client
```

#### Explanation  
This confirms that you are working from the Azure client machine where the SSH key will be generated or reused.

---

### Step 2: Check Existing SSH Key Pair  

Run:

```bash
ls -la ~/.ssh/
```

Look for:

```bash
id_rsa
id_rsa.pub
```

#### Explanation  
The private key (`id_rsa`) and public key (`id_rsa.pub`) are required for SSH key-based authentication.

---

### Step 3: Generate SSH Key Pair (If Required)  

If no SSH key exists, create a new one:

```bash
ssh-keygen -t rsa -b 2048 -f ~/.ssh/id_rsa -N ""
```

Verify the public key:

```bash
cat ~/.ssh/id_rsa.pub
```

Example output:

```bash
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQ...
```

#### Explanation  
This generates an RSA SSH key pair that will be associated with the Azure Virtual Machine.

---

### Step 4: Identify the Resource Group  

Run:

```bash
az group list --output table
```

Store the Resource Group name:

```bash
RG=$(az group list --query "[0].name" -o tsv)
```

Verify:

```bash
echo $RG
```

#### Explanation  
The Resource Group is required for creating and managing Azure resources.

---

### Step 5: Create the Virtual Machine  

Run:

```bash
az vm create \
  --resource-group $RG \
  --name nautilus-vm \
  --location westus \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --admin-username azureuser \
  --ssh-key-values ~/.ssh/id_rsa.pub \
  --storage-sku Standard_LRS
```

#### Important  

The parameter below helps avoid storage policy issues in lab environments:

```bash
--storage-sku Standard_LRS
```

#### Explanation  
This command creates the Virtual Machine and automatically configures the `azureuser` account with the SSH public key from the Azure client machine.

---

### Step 6: Open SSH Port  

Run:

```bash
az vm open-port \
  --resource-group $RG \
  --name nautilus-vm \
  --port 22
```

#### Explanation  
This creates an NSG rule allowing inbound SSH traffic to the Virtual Machine.

---

### Step 7: Retrieve Public IP Address  

Run:

```bash
az vm show \
  --resource-group $RG \
  --name nautilus-vm \
  -d \
  --query publicIps \
  -o tsv
```

Example output:

```bash
20.xx.xx.xx
```

Store the IP:

```bash
IP=$(az vm show \
  --resource-group $RG \
  --name nautilus-vm \
  -d \
  --query publicIps \
  -o tsv)

echo $IP
```

#### Explanation  
The Public IP Address is required to establish an SSH connection to the Virtual Machine.

---

### Step 8: Verify VM Initialization  

Run:

```bash
az vm get-instance-view \
  --resource-group $RG \
  --name nautilus-vm \
  --query "instanceView.statuses[].displayStatus" \
  -o table
```

Expected output:

```text
Provisioning succeeded
VM running
```

#### Explanation  
This confirms that the Virtual Machine has been deployed successfully and is ready for connections.

---

### Step 9: Verify Password-less SSH Access  

Run:

```bash
ssh azureuser@$IP
```

If prompted:

```text
Are you sure you want to continue connecting?
```

Type:

```text
yes
```

Expected login:

```bash
azureuser@nautilus-vm:~$
```

#### Explanation  
Successful login without entering a password confirms that SSH key-based authentication is working correctly.

---

### Step 10: Verify Hostname  

Inside the VM, run:

```bash
hostname
```

Expected output:

```bash
nautilus-vm
```

Exit the session:

```bash
exit
```

#### Explanation  
This verifies that you are connected to the correct Virtual Machine.

---

## Best Practices  

- Use SSH key authentication instead of passwords whenever possible  
- Protect private keys and never share them publicly  
- Restrict SSH access using NSG rules and trusted IP ranges  
- Use standardized usernames and access policies across environments  
- Regularly rotate SSH keys for enhanced security  
- Verify SSH connectivity immediately after VM deployment  

## Key Learnings  

- SSH key authentication provides stronger security than password-based login  
- Azure Virtual Machines support SSH key injection during deployment  
- NSG rules control inbound SSH access to Azure resources  
- Azure CLI can fully automate secure VM provisioning  
- Password-less authentication simplifies secure server administration  
- Proper SSH configuration is a fundamental DevOps security practice  
