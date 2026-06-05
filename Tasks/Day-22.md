# Day 22 – Configuring Instances with User Data

---

## Task Overview  

The Nautilus DevOps Team is working on setting up a new virtual machine (VM) to host a web server for a critical application. The team lead has requested you to create an Azure VM that will serve as a web server using Nginx. This VM will be part of the initial infrastructure setup for the Nautilus project. Ensuring that the server is correctly configured and accessible from the internet is crucial for the upcoming deployment phase.

As a member of the Nautilus DevOps Team, your task is to create a VM with the following specifications:

- **Instance Name:** The VM must be named `datacenter-vm`.

- **Image:** Use any available Ubuntu image to create this VM.

- **Custom Script Extension/User Data:** Configure the VM to run a custom script during its launch. This script should:

  - Install the Nginx package.

  - Start the Nginx service.
    
- **Network Security Group (NSG):** Ensure that the VM allows HTTP traffic on port `80` from the internet.

---

## Step-by-Step Implementation  

### Step 1: Login to Azure Portal  

Open the Azure Portal:

https://portal.azure.com

Login using your Azure account credentials.

#### Explanation  
The Azure Portal provides a centralized interface for provisioning and managing Azure resources.

---

### Step 2: Start Creating a Virtual Machine  

Search for:

| Search |
|---|
| Virtual Machines |

Click:

- **+ Create**
- **Azure Virtual Machine**

#### Explanation  
This opens the Virtual Machine deployment wizard.

---

### Step 3: Configure Basic VM Settings  

Under **Basics**, configure the following:

| Setting | Value |
|---|---|
| Resource Group | Existing Lab Resource Group |
| Virtual Machine Name | `datacenter-vm` |
| Region | Same as Resource Group |
| Availability Options | No Infrastructure Redundancy Required |
| Image | Ubuntu Server 22.04 LTS (or any Ubuntu image) |
| Size | Standard_B1s |

#### Explanation  
These settings define the VM name, operating system, and compute configuration.

---

### Step 4: Configure Authentication  

Under **Administrator Account**, configure:

| Setting | Value |
|---|---|
| Authentication Type | SSH Public Key |
| Username | `azureuser` |
| SSH Key Source | Generate New Key Pair |

Under **Inbound Port Rules**, configure:

| Setting | Value |
|---|---|
| Public Inbound Ports | Allow Selected Ports |
| Select Inbound Ports | SSH (22), HTTP (80) |

#### Explanation  
This enables secure SSH access and allows web traffic to reach the Nginx server.

---

### Step 5: Configure Storage Settings  

Click:

- **Next : Disks**

Configure:

| Setting | Value |
|---|---|
| OS Disk Type | Standard HDD (LRS) |

#### Explanation  
Using Standard HDD (LRS) helps comply with common lab environment policies and minimizes deployment issues.

---

### Step 6: Configure Networking  

Click:

- **Next : Networking**

Verify that the Network Security Group allows:

| Port | Protocol | Action |
|---|---|---|
| 22 | TCP | Allow |
| 80 | TCP | Allow |

If HTTP is not present, create a new inbound rule for port 80.

#### Explanation  
Port 22 enables SSH access, while Port 80 allows users to access the Nginx web server through a browser.

---

### Step 7: Configure User Data / Custom Script  

Click:

- **Next : Advanced**

Locate:

| Setting |
|---|
| Custom Data and Cloud-init |

Paste the following script:

```bash
#!/bin/bash

apt-get update -y
apt-get install nginx -y
systemctl enable nginx
systemctl start nginx
```

#### Explanation  
This startup script automatically installs Nginx, enables it to start during boot, and immediately starts the service after VM creation.

---

### Step 8: Review and Create  

Click:

- **Review + Create**

After validation succeeds, click:

- **Create**

Wait for deployment to complete.

#### Explanation  
Azure validates all VM settings and provisions the Virtual Machine with the startup script.

---

### Step 9: Verify Virtual Machine Deployment  

After deployment completes, click:

- **Go to Resource**

Verify:

| Property | Expected Value |
|---|---|
| VM Name | `datacenter-vm` |
| Status | Running |
| Public IP | Assigned |

#### Explanation  
This confirms that the Virtual Machine was successfully deployed.

---

### Step 10: Verify Nginx Web Server  

Copy the VM Public IP Address.

Open a web browser and access:

```text
http://<PUBLIC-IP>
```

Example:

```text
http://20.xx.xx.xx
```

Expected page:

```text
Welcome to nginx!
```

#### Explanation  
The default Nginx welcome page confirms that the startup script executed successfully and the web server is running.

---

### Step 11: Verify Nginx Service via SSH (Optional)  

Connect to the VM:

```bash
ssh azureuser@<PUBLIC-IP>
```

Check service status:

```bash
sudo systemctl status nginx
```

Expected output:

```text
active (running)
```

#### Explanation  
This verifies that Nginx is installed, enabled, and actively running on the server.

---

## Best Practices  

- Automate server provisioning using cloud-init or startup scripts  
- Use Infrastructure as Code whenever possible for consistency  
- Allow only required inbound ports through NSGs  
- Validate service status after deployment completes  
- Keep operating system packages updated regularly  
- Use startup scripts to standardize environment configuration  

## Key Learnings  

- User Data (cloud-init) enables automated VM configuration during deployment  
- Azure Virtual Machines can execute startup scripts automatically  
- Nginx can be installed and configured without manual intervention  
- Network Security Groups control inbound web traffic access  
- Cloud automation reduces manual configuration effort and errors  
- Startup scripts improve deployment consistency across environments  
