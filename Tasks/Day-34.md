# Day 34 – Enabling Internet Connectivity for Virtual Machines

---

## Task Overview

The Nautilus DevOps team has encountered an issue with an Azure VM named `nautilus-vm`. They are unable to install any packages on this VM due to connectivity issues. The team needs to identify the root cause of the problem and resolve it to restore normal operations. 

- Investigate the connectivity issue preventing package installation on the Azure VM `nautilus-vm`. 

- Implement a solution to resolve the connectivity issue and restore package installation capabilities on the VM.

`Note:` The SSH key required to access the Azure VM is already created and added to the VM's authorized keys. You can find the SSH key at `/root/.ssh/id_rsa` on the `azure-client` host.

---

## Step-by-Step Implementation

### Step 1: Verify SSH Key Availability

On the `azure-client` host, verify that the SSH private key exists.

```bash
ls -l /root/.ssh/id_rsa
```

#### Explanation

The task provides an SSH key for accessing the VM. This step confirms that the key is available before attempting to connect.

---

### Step 2: Identify the Resource Group

Retrieve the default resource group assigned to the lab environment.

```bash
RG=$(az group list --query "[0].name" -o tsv)

echo $RG
```

#### Explanation

The resource group contains all Azure resources required for this task and will be used in subsequent Azure CLI commands.

---

### Step 3: Get the Virtual Machine Public IP

Retrieve the public IP address associated with `nautilus-vm`.

```bash
az vm show \
  --resource-group $RG \
  --name nautilus-vm \
  -d \
  --query publicIps \
  -o tsv
```

#### Explanation

The public IP is required to establish an SSH connection to the virtual machine.

---

### Step 4: Connect to the Virtual Machine

Use the SSH key to connect to the VM.

```bash
ssh -i /root/.ssh/id_rsa azureuser@<PUBLIC_IP>
```

#### Explanation

This confirms that inbound SSH connectivity is working and allows further troubleshooting from inside the VM.

---

### Step 5: Verify Package Installation Issue

Attempt to update package repositories.

```bash
sudo apt update
```

#### Explanation

The command hangs while connecting to Ubuntu repositories, confirming the package installation issue reported by the team.

---

### Step 6: Check Routing Configuration

Verify the routing table on the VM.

```bash
ip route
```

#### Explanation

A valid default route confirms that the Linux networking configuration is not causing the issue.

---

### Step 7: Test Internet Connectivity

Test connectivity to a public IP address.

```bash
ping -c 4 8.8.8.8
```

#### Explanation

Failure to reach a public IP indicates that outbound internet traffic is blocked.

---

### Step 8: Verify DNS Resolution

Test DNS functionality.

```bash
ping -c 4 google.com
```

#### Explanation

Successful hostname resolution confirms that DNS is working correctly and is not the root cause.

---

### Step 9: Exit the Virtual Machine

Return to the `azure-client` host.

```bash
exit
```

#### Explanation

The remaining investigation needs to be performed from Azure CLI.

---

### Step 10: Check Attached Network Security Group

Identify the NSG associated with the VM NIC.

```bash
az network nic show \
  --resource-group $RG \
  --name nautilus-vmVMNic \
  --query networkSecurityGroup.id \
  -o tsv
```

#### Explanation

This identifies the security group responsible for controlling inbound and outbound traffic.

---

### Step 11: Review NSG Rules

List all configured security rules.

```bash
az network nsg rule list \
  --resource-group $RG \
  --nsg-name nautilus-nsg \
  -o table
```

#### Explanation

Reviewing NSG rules helps identify any configuration that may be blocking internet access.

---

### Step 12: Identify the Blocking Rule

Locate the outbound deny rule.

```text
Block-All-Outbound
```

#### Explanation

This rule denies all outbound traffic, preventing the VM from accessing package repositories and the internet.

---

### Step 13: Remove the Blocking Rule

Delete the outbound deny rule.

```bash
az network nsg rule delete \
  --resource-group $RG \
  --nsg-name nautilus-nsg \
  --name Block-All-Outbound
```

#### Explanation

Removing the rule restores outbound internet connectivity for the virtual machine.

---

### Step 14: Verify Connectivity

Reconnect to the VM.

```bash
ssh -i /root/.ssh/id_rsa azureuser@<PUBLIC_IP>
```

Test connectivity again.

```bash
ping -c 4 8.8.8.8
```

#### Explanation

Successful replies confirm that internet connectivity has been restored.

---

### Step 15: Verify Package Installation

Run the package update command again.

```bash
sudo apt update
```

#### Explanation

Successful repository synchronization confirms that package installation capabilities have been restored.

---

### Step 16: Optional Validation

Install a sample package.

```bash
sudo apt install nginx -y
```

#### Explanation

Installing a package provides additional confirmation that outbound connectivity is functioning correctly.

---

## Best Practices

- Verify connectivity from inside the VM before changing Azure resources.
- Review both inbound and outbound NSG rules during troubleshooting.
- Test DNS and internet connectivity separately.
- Use Azure CLI for faster diagnostics and validation.
- Apply least-privilege security rules whenever possible.
- Document troubleshooting steps and resolutions for future reference.

## Key Learnings

- NSGs control both inbound and outbound traffic in Azure.
- Outbound deny rules can prevent package installations.
- DNS resolution and internet connectivity are independent checks.
- Azure CLI is useful for network troubleshooting.
- Systematic investigation helps identify root causes quickly.
- Proper NSG configuration is critical for VM functionality.
