# Day 28 – Troubleshooting Public Virtual Network Configurations

---

## Task Overview

The Nautilus DevOps Team attempted to deploy an Nginx server on an Azure VM in a public VNet named devops-vnet, but they were unable to complete the setup successfully, and the server remains inaccessible from the internet.

As a DevOps team member, complete the following tasks:

- **Verify VNet Configuration:** Ensure `devops-vnet` allows internet access.

- **Attach Public IP:** A public IP named `devops-pip` already exists. Attach this public IP to the VM `devops-vm` to make it accessible from the internet.

- **Ensure Accessibility:** Confirm the VM `devops-vm` is accessible on port `80`.

- **Install and Configure Nginx:** Install `Nginx` on `devops-vm` and ensure the service is enabled and running, listening on port `80`.
  
`Notes:`

- Create resources only in the `West US` region.

- Ensure the Network Security Group (NSG) is attached to the VM's NIC or subnet and configured to allow HTTP traffic on port `80`.

---

## Step-by-Step Implementation

### Step 1: Verify Existing Resources

Login to the Azure CLI environment and verify that the required resources already exist.

```bash
az vm list --output table
az network public-ip list --output table
az network vnet list --output table
```

#### Explanation

These commands confirm that the VM, Public IP, and Virtual Network required for the task are already deployed.

Expected resources:

| Resource | Name |
|----------|------|
| VM | `devops-vm` |
| Public IP | `devops-pip` |
| VNet | `devops-vnet` |

---

### Step 2: Identify the Resource Group

Retrieve the resource group assigned to the lab environment.

```bash
RG=$(az group list --query "[0].name" -o tsv)

echo $RG
```

#### Explanation

The resource group is required for all subsequent Azure CLI operations.

---

### Step 3: Verify VNet Configuration

Check the existing Virtual Network configuration.

```bash
az network vnet show \
  --resource-group $RG \
  --name devops-vnet \
  --output table
```

Verify the subnet configuration.

```bash
az network vnet subnet list \
  --resource-group $RG \
  --vnet-name devops-vnet \
  --output table
```

#### Explanation

This confirms that the VNet and subnet are properly configured and available for internet connectivity.

---

### Step 4: Attach the Existing Public IP

Identify the VM Network Interface Card (NIC).

```bash
az network nic list --output table
```

Expected output:

```text
devops-vmVMNic
```

Retrieve the IP configuration name.

```bash
az network nic show \
  --resource-group $RG \
  --name devops-vmVMNic \
  --query "ipConfigurations[].name" \
  -o table
```

Attach the existing Public IP.

```bash
az network nic ip-config update \
  --resource-group $RG \
  --nic-name devops-vmVMNic \
  --name ipconfigdevops-vm \
  --public-ip-address devops-pip
```

#### Explanation

The Public IP enables internet access to the Virtual Machine.

---

### Step 5: Verify Public IP Assignment

Check whether the Public IP is associated successfully.

```bash
az vm show \
  --resource-group $RG \
  --name devops-vm \
  -d \
  --query "{name:name,publicIp:publicIps}" \
  -o table
```

#### Explanation

The VM should display the Public IP assigned through `devops-pip`.

---

### Step 6: Verify Network Security Group

List available NSGs.

```bash
az network nsg list --output table
```

Identify the NSG attached to the VM.

```bash
az network nsg rule list \
  --resource-group $RG \
  --nsg-name devops-vmNSG \
  --output table
```

#### Explanation

The NSG controls inbound and outbound traffic to the Virtual Machine.

---

### Step 7: Allow HTTP Traffic on Port 80

If an HTTP rule does not already exist, create one.

```bash
az network nsg rule create \
  --resource-group $RG \
  --nsg-name devops-vmNSG \
  --name Allow-HTTP \
  --priority 300 \
  --access Allow \
  --direction Inbound \
  --protocol Tcp \
  --source-address-prefixes '*' \
  --source-port-ranges '*' \
  --destination-port-ranges 80
```

#### Explanation

This rule allows web traffic from the internet to reach the Nginx server.

---

### Step 8: Verify Route Table Configuration

List route tables.

```bash
az network route-table list --output table
```

Check routes.

```bash
az network route-table route list \
  --resource-group $RG \
  --route-table-name devops-rtb \
  --output table
```

#### Explanation

A misconfigured route table can prevent outbound internet connectivity, causing package installations to fail.

---

### Step 9: Remove Blocking Route

If a route similar to the following exists:

```text
0.0.0.0/0  →  None
```

Delete it.

```bash
az network route-table route delete \
  --resource-group $RG \
  --route-table-name devops-rtb \
  --name Block-Internet
```

#### Explanation

This removes the route preventing internet access.

---

### Step 10: Create Internet Route

Create a route that allows internet connectivity.

```bash
az network route-table route create \
  --resource-group $RG \
  --route-table-name devops-rtb \
  --name Internet-Route \
  --address-prefix 0.0.0.0/0 \
  --next-hop-type Internet
```

#### Explanation

This route enables outbound traffic required for package downloads and updates.

---

### Step 11: Verify Internet Connectivity

Run a connectivity test from the VM.

```bash
az vm run-command invoke \
  --resource-group $RG \
  --name devops-vm \
  --command-id RunShellScript \
  --scripts "ping -c 3 8.8.8.8"
```

#### Explanation

Successful replies confirm internet access is working.

---

### Step 12: Install Nginx

Install Nginx directly from Azure CLI.

```bash
az vm run-command invoke \
  --resource-group $RG \
  --name devops-vm \
  --command-id RunShellScript \
  --scripts "
apt update
apt install nginx -y
systemctl enable nginx
systemctl restart nginx
"
```

#### Explanation

This installs the Nginx package and ensures the service starts automatically.

---

### Step 13: Verify Nginx Service

Check the service status.

```bash
az vm run-command invoke \
  --resource-group $RG \
  --name devops-vm \
  --command-id RunShellScript \
  --scripts "systemctl status nginx --no-pager"
```

Expected output:

```text
active (running)
```

#### Explanation

This confirms that Nginx is running successfully.

---

### Step 14: Verify Web Server Accessibility

Retrieve the Public IP.

```bash
az vm show \
  --resource-group $RG \
  --name devops-vm \
  -d \
  --query publicIps \
  -o tsv
```

Test the website.

```bash
curl http://<PUBLIC_IP>
```

Expected output:

```html
Welcome to nginx!
```

#### Explanation

This confirms that the web server is publicly accessible.

---

## Best Practices

- Always verify Public IP association before troubleshooting connectivity issues.
- Configure NSG rules following the principle of least privilege.
- Validate route table entries whenever internet access problems occur.
- Use Azure VM Run Command for remote troubleshooting when SSH is unavailable.
- Enable critical services to start automatically after reboots.
- Perform end-to-end connectivity testing after infrastructure changes.

## Key Learnings

- Public IPs must be associated with a VM NIC for internet accessibility.
- NSGs act as virtual firewalls controlling inbound and outbound traffic.
- Route tables can override network connectivity even when NSG rules are correct.
- Azure VM Run Command simplifies VM administration without SSH access.
- Nginx installation depends on outbound internet connectivity.
- Proper validation ensures applications are accessible from external networks.
