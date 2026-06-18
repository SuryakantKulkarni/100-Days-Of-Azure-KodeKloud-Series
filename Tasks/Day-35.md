# Day 35 – Configuring Virtual Network Peering

---

## Task Overview

The Nautilus DevOps team has been tasked with demonstrating the use of VNet Peering to enable communication between two VNets. One VNet will be a private VNet that contains a private Azure VM, while the other will be a public VNet containing a publicly accessible Azure VM.

**1) Existing Azure Resources:**

- Public VM: `nautilus-pub-vm` is already in the public VNet.

- Private VNet and VM: `nautilus-priv-vnet` and `nautilus-priv-vm` exist in the private VNet with its subnet: `nautilus-priv-subnet`.

**2) Create VNet Peering:**

- Create a VNet Peering between the Public VNet and Private VNet.

- VNet Peering Name: `nautilus-pub-to-priv-peering`.

**3) Test the Connection:**

- SSH into the public VM and verify that you can ping the private VM.

---

## Step-by-Step Implementation

### Step 1: Verify Existing Resources

On the `azure-client` host, verify the available virtual machines.

```bash
RG=$(az group list --query "[0].name" -o tsv)

az vm list -o table
```

#### Explanation

This command confirms that both the public and private virtual machines are available before configuring VNet peering.

---

### Step 2: Verify Virtual Networks

List the available VNets.

```bash
az network vnet list -o table
```

#### Explanation

This verifies that both `nautilus-pub-vnet` and `nautilus-priv-vnet` exist and are ready for peering.

---

### Step 3: Open Public VNet Peering Configuration

Navigate to:

```text
Azure Portal
→ Virtual Networks
→ nautilus-pub-vnet
→ Peerings
→ + Add
```

#### Explanation

VNet peering is initiated from the public VNet to establish communication with the private VNet.

---

### Step 4: Configure Local Peering Settings

Enter the following values:

| Field             | Value                          |
| ----------------- | ------------------------------ |
| Peering Link Name | `nautilus-pub-to-priv-peering` |

Enable:

```text
Allow 'nautilus-pub-vnet' to access the peered virtual network
```

#### Explanation

This creates the required peering connection from the public VNet to the private VNet.

---

### Step 5: Configure Remote Peering Settings

Select:

| Field                  | Value                 |
| ---------------------- | --------------------- |
| Remote Virtual Network | `nautilus-priv-vnet`  |
| Remote Peering Name    | `priv-to-pub-peering` |

Enable:

```text
Allow the peered virtual network to access 'nautilus-pub-vnet'
```

#### Explanation

This creates the reverse peering connection automatically and allows bidirectional communication.

---

### Step 6: Create the Peering

Click:

```text
Add
```

Wait until the peering status becomes:

```text
Connected
```

#### Explanation

The peering must show a Connected status before communication between VNets can occur.

---

### Step 7: Verify VNet Peering

Verify peering on the public VNet.

```bash
az network vnet peering list \
  --resource-group $RG \
  --vnet-name nautilus-pub-vnet \
  -o table
```

#### Explanation

This confirms that the peering named `nautilus-pub-to-priv-peering` is successfully connected.

---

### Step 8: Verify Reverse Peering

Check the private VNet peering.

```bash
az network vnet peering list \
  --resource-group $RG \
  --vnet-name nautilus-priv-vnet \
  -o table
```

#### Explanation

The reverse peering should also show a Connected status.

---

### Step 9: Retrieve VM IP Addresses

Get the public and private IP addresses of both VMs.

```bash
az vm list -d \
  --resource-group $RG \
  --query "[].{VM:name,PublicIP:publicIps,PrivateIP:privateIps}" \
  -o table
```

#### Explanation

The public IP of `nautilus-pub-vm` is required for SSH access, while the private IP of `nautilus-priv-vm` is required for connectivity testing.

---

### Step 10: SSH into the Public VM

Connect to the public VM.

```bash
ssh -i /root/.ssh/id_rsa azureuser@<PUBLIC_IP>
```

#### Explanation

The public VM acts as the source machine for testing connectivity to the private VM.

---

### Step 11: Test Connectivity to Private VM

Ping the private VM using its private IP address.

```bash
ping <PRIVATE_IP>
```

Example:

```bash
ping 10.1.1.4
```

#### Explanation

Successful ping responses confirm that communication between the two VNets is working through VNet Peering.

---

### Step 12: Stop the Ping Test

Press:

```text
Ctrl + C
```

Verify the output shows:

```text
0% packet loss
```

#### Explanation

Zero packet loss confirms stable network connectivity between both virtual networks.

---

## Best Practices

* Use VNet Peering instead of VPNs for communication within Azure whenever possible.
* Verify peering status before performing connectivity tests.
* Follow a consistent naming convention for peering connections.
* Restrict network access using NSGs where required.
* Monitor peering connections regularly for connectivity issues.
* Document network topology and peering relationships.

## Key Learnings

* VNet Peering enables low-latency communication between Azure VNets.
* Peered VNets communicate using private IP addresses.
* Bidirectional peering is required for full connectivity.
* Azure CLI can be used to validate peering configurations.
* Network troubleshooting should always include connectivity verification.
* VNet Peering simplifies secure communication between isolated environments.
