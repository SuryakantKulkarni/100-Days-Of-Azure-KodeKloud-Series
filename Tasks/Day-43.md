# Day 43 – Configuring Azure VM with Application Gateway

---

## Task Overview

The Nautilus Development Team needs to set up a new Azure Virtual Machine (VM) and configure it to run a web server. This VM should be part of an Azure Application Gateway (AGW) setup to ensure high availability and better traffic management. The task involves creating a VM, setting up an AGW, configuring a backend pool, and ensuring the web server is accessible via the AGW public IP. 

- **Create a Network Security Group (NSG):** Create an NSG named `nautilus-nsg` and add an inbound security rule `Allow-HTTP` to `Allow TCP` traffic on port `80`.

- **Create a Virtual Machine:** Create a VM named `nautilus-vm` using any available Ubuntu image. Configure the instance with the following settings:

  - **Size:** Choose a lightweight VM size (e.g., `Standard_B1s`).
  - **Authentication:** Use `SSH public key` authentication. (Please select `use existing public key` option, create public-key locally and paste contents of `~/.ssh/id_rsa.pub`)
  - **OS Disk:** Use a `Standard HDD`.
  - **Networking:** Under the Advanced section, attach an existing NSG (e.g., `nautilus-nsg`).

Additionally, configure the instance to run a user data script during launch that: 

  - Install the Nginx package.
  - Start the Nginx service.

- **Set up an Application Gateway:** Set up an Azure Application Gateway named `nautilus-agw` with the following:

  - Create and Associate it with a **Public IP Address** named `nautilus-agw-ip`.
  - Attach the **backend pool**:`nautilus-backendpool` to the **VM** `nautilus-vm`.
  - Select a **subnet** for the Application Gateway (you can create a new one if needed).
  
- **Configure HTTP Settings:** Create an HTTP setting named `nautilus-http-settings` on port `80`

- **Route Traffic:** Add a **listener** named `nautilus-listener` and a **routing rule** named `nautilus-routing-rule` to route traffic from the AGW frontend to the backend pool:

  - Listener: Frontend IP = public IP, Frontend port = 80, Protocol = HTTP
  - Routing rule: Connects `nautilus-listener` to `nautilus-backendpool` using `nautilus-http-settings`.

- **NSG Adjustments:** Make sure the **NSG** attached to the VM allows **inbound TCP traffic on port 80**, so the Nginx server running on `nautilus-vm` is accessible via the Application Gateway public IP.

`Note:` Wait for the Application Gateway resource to be fully deployed before proceeding with the next steps. Deployment may take several minutes to complete.

---

# Step-by-Step Implementation

### Step 1: Get the Resource Group

```bash
RG=$(az group list --query "[0].name" -o tsv)

echo $RG
```

#### Explanation

Retrieves the default Azure Resource Group for the lab.

---

### Step 2: Generate SSH Public Key

Check if an SSH key already exists:

```bash
ls ~/.ssh/id_rsa ~/.ssh/id_rsa.pub
```

If not present, generate one:

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""
```

Display the public key:

```bash
cat ~/.ssh/id_rsa.pub
```

#### Explanation

The generated public key will be used for password-less SSH authentication to the VM.

---

### Step 3: Create the Network Security Group (GUI)

Go to:

```text
Azure Portal
→ Network Security Groups
→ Create
```

Fill in:

| Field          | Value          |
| -------------- | -------------- |
| Name           | `nautilus-nsg` |
| Resource Group | Existing RG    |
| Region         | West US        |

Click:

```text
Review + Create
→ Create
```

#### Explanation

Creates an NSG to secure the Virtual Machine.

---

### Step 4: Add HTTP Inbound Rule

Open:

```text
nautilus-nsg
→ Inbound Security Rules
→ Add
```

Configure:

| Field            | Value      |
| ---------------- | ---------- |
| Source           | Any        |
| Source Port      | *          |
| Destination      | Any        |
| Destination Port | 80         |
| Protocol         | TCP        |
| Action           | Allow      |
| Priority         | 110        |
| Name             | Allow-HTTP |

Click:

```text
Add
```

#### Explanation

Allows HTTP traffic so the Application Gateway can access the Nginx web server.

---

### Step 5: Create the Virtual Machine

Go to:

```text
Azure Portal
→ Virtual Machines
→ Create
```

Configure:

| Field                | Value                     |
| -------------------- | ------------------------- |
| VM Name              | `nautilus-vm`             |
| Region               | West US                   |
| Image                | Ubuntu Server 22.04 LTS   |
| Size                 | Standard_B1s              |
| Authentication       | SSH Public Key            |
| Username             | azureuser                 |
| SSH Key              | Paste `~/.ssh/id_rsa.pub` |
| Public Inbound Ports | SSH (22)                  |

#### Disks Tab

| Field   | Value              |
| ------- | ------------------ |
| OS Disk | Standard HDD (LRS) |

#### Networking Tab

Attach the existing NSG:

```text
nautilus-nsg
```

---

### Step 6: Configure User Data (Cloud-Init)

Under:

```text
Advanced
→ Custom Data
```

Paste:

```bash
#cloud-config
package_update: true
packages:
  - nginx
runcmd:
  - systemctl enable nginx
  - systemctl restart nginx
```

#### Explanation

Automatically installs and starts Nginx during VM provisioning.

---

### Step 7: Verify the VM

Get the VM Public IP:

```bash
VMIP=$(az vm show -d \
-g $RG \
-n nautilus-vm \
--query publicIps -o tsv)

echo $VMIP
```

SSH into the VM:

```bash
ssh azureuser@$VMIP
```

Check Nginx:

```bash
sudo systemctl status nginx
```

If required:

```bash
sudo apt update
sudo apt install nginx -y
sudo systemctl enable nginx
sudo systemctl restart nginx
```

Test locally:

```bash
curl localhost
```

Expected:

```html
Welcome to nginx!
```

Exit:

```bash
exit
```

#### Explanation

Confirms that the web server is running correctly.

---

### Step 8: Create Application Gateway Public IP

Go to:

```text
Public IP Addresses
→ Create
```

Configure:

| Field  | Value             |
| ------ | ----------------- |
| Name   | `nautilus-agw-ip` |
| SKU    | Standard          |
| Region | West US           |

Create the resource.

---

### Step 9: Create the Application Gateway

Go to:

```text
Azure Portal
→ Application Gateways
→ Create
```

#### Basics

| Field          | Value          |
| -------------- | -------------- |
| Name           | `nautilus-agw` |
| Region         | West US        |
| Tier           | Basic          |
| Instance Count | 1              |

---

#### Frontends

Select the Public IP:

```text
nautilus-agw-ip
```

---

#### Networking

| Field           | Value                                 |
| --------------- | ------------------------------------- |
| Virtual Network | Existing VNet                         |
| Subnet          | appGatewaySubnet (create if required) |

---

#### Backend Pool

Create:

| Field       | Value                  |
| ----------- | ---------------------- |
| Name        | `nautilus-backendpool` |
| Target Type | Virtual Machine        |
| Target      | `nautilus-vm`          |

---

#### HTTP Settings

| Field    | Value                    |
| -------- | ------------------------ |
| Name     | `nautilus-http-settings` |
| Protocol | HTTP                     |
| Port     | 80                       |

---

#### Listener

| Field    | Value               |
| -------- | ------------------- |
| Name     | `nautilus-listener` |
| Frontend | Public IPv4         |
| Port     | 80                  |
| Protocol | HTTP                |

---

#### Routing Rule

| Field         | Value                    |
| ------------- | ------------------------ |
| Name          | `nautilus-routing-rule`  |
| Listener      | `nautilus-listener`      |
| Backend Pool  | `nautilus-backendpool`   |
| HTTP Settings | `nautilus-http-settings` |

Click:

```text
Review + Create
→ Create
```

Deployment may take **5–10 minutes**.

#### Explanation

The Application Gateway routes HTTP traffic from the public IP to the backend VM running Nginx.

---

## Azure CLI Verification

### Verify VM

```bash
az vm list -d \
-g $RG \
-o table
```

---

### Verify NSG Rules

```bash
az network nsg rule list \
-g $RG \
--nsg-name nautilus-nsg \
-o table
```

---

### Verify Application Gateway

```bash
az network application-gateway list \
-g $RG \
-o table
```

---

### Get AGW Public IP

```bash
AGWIP=$(az network public-ip show \
-g $RG \
-n nautilus-agw-ip \
--query ipAddress -o tsv)

echo $AGWIP
```

---

### Test the Website

```bash
curl http://$AGWIP
```

Expected:

```html
Welcome to nginx!
```

---

## Common Errors & Fixes

### Error: SSH Connection Timeout

**Cause**

Port 22 not allowed.

**Fix**

Add an inbound SSH rule to the NSG.

---

### Error: Nginx Not Running

**Cause**

Cloud-init has not completed.

**Fix**

```bash
sudo apt update
sudo apt install nginx -y
sudo systemctl enable nginx
sudo systemctl restart nginx
```

---

### Error: Application Gateway Backend Unhealthy

**Cause**

Port 80 blocked or Nginx stopped.

**Fix**

* Verify NSG allows TCP port **80**.
* Ensure Nginx service is running.
* Check HTTP settings use **port 80**.

---

### Error: AGW Creation Fails

**Cause**

Incorrect subnet configuration.

**Fix**

Create a dedicated subnet named:

```text
appGatewaySubnet
```

within the same Virtual Network.

---

## Best Practices

* Use **Standard HDD (LRS)** for lab compatibility.
* Create a dedicated subnet for the Application Gateway.
* Use Cloud-Init for automated software installation.
* Allow only required inbound ports (22 and 80).
* Verify backend health before testing through the Application Gateway.

## Key Learnings

* Creating and configuring Azure Network Security Groups.
* Deploying Ubuntu Virtual Machines using SSH authentication.
* Automating VM configuration using Cloud-Init.
* Configuring Azure Application Gateway with backend pools.
* Routing HTTP traffic through an Application Gateway to a backend VM.
* Verifying web application availability through the AGW public IP.
