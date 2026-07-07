# Day 49 – VM Setup with Web Storage Integration

---

## Task Overview

The Nautilus DevOps team is tasked with setting up an environment to host a static web application. The application will serve static content from an Azure Storage Account, and a Virtual Machine (VM) will be configured to fetch and display this content using Nginx. The Azure Storage Account is used as a secure, centralized location for storing the index.html file. The team intentionally keeps this file outside the main source code repository, since that repository contains additional internal application code that should not be exposed to or accessed by the VM. By placing only the required static file in the Storage Account, the team can distribute this asset safely and independently of the full codebase.

The VM should securely download the index.html blob directly from the designated container (e.g., using Azure CLI, SAS URL, or REST API) and place it in Nginx’s web root directory so that it is served locally by Nginx. The Storage Account is not mounted, and the Static Website feature is not used. The VM retrieves the file during deployment and may re-fetch it whenever updates are needed. The resources must follow best practices for security, performance, and accessibility.

Task Details:

**1) Create a Virtual Network (VNet) and Subnet:**

- Create a VNet named `xfusion-vnet` in the East US region.
- Create a subnet named `xfusion-subnet` within the VNet for the VM.

**2) Create an Azure Storage Account:**

- Create a storage account named `xfusionstor5079` in the **East US** region with **Locally-redundant storage (LRS)**.
- Create a Blob container named `xfusion-container` in the storage account.
- Upload the `index.html` file located at `/root` on the **client host** to the container `xfusion-container`.
- Ensure the **Storage Account is private and not publicly accessible** by disabling public access for the storage account.

**3) Create a Virtual Machine (VM):**

- Create a VM named `xfusion-vm` in the **East US** region.
- Use the `xfusion-vnet` and subnet `xfusion-subnet` for the VM.
- Authentication: Use `SSH public key` authentication. (Please select `use existing public key` option, create public-key locally and paste contents of `~/.ssh/id_rsa.pub`)
- Install **Nginx** on the VM.
- Download the `index.html` file using a command such as:

```bash
sudo az storage blob download --account-name xfusionstor5079 --account-key xxxxx --container-name xfusion-container --name index.html --file /var/www/html/index.html
```

- Ensure Nginx is configured to serve the file from `/var/www/html/index.html`.

**4) Verify Setup:**

- Verify that the Nginx web server on the client host serves the `index.html` file correctly when accessing the VM's public IP address.

`Note:` Follow best practices for security, accessibility, and performance while configuring resources.

---

## Step-by-Step Implementation

### Step 1: Get the Resource Group

Retrieve the default Resource Group created for the lab.

```bash
RG=$(az group list --query "[0].name" -o tsv)

echo $RG
```

#### Explanation

This command stores the existing Resource Group name in the `RG` variable, which is used throughout the lab.

---

### Step 2: Generate an SSH Key Pair

If an SSH key pair does not already exist, generate one.

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""
```

Display the public key:

```bash
cat ~/.ssh/id_rsa.pub
```

#### Explanation

The public key is required while creating the VM using **SSH Public Key Authentication**.

---

### Step 3: Create a Virtual Network (Azure Portal)

Navigate to:

```text
Azure Portal
→ Virtual Networks
→ Create
```

Configure:

| Setting | Value        |
| ------- | ------------ |
| Name    | xfusion-vnet |
| Region  | East US      |

Under **IP Addresses**, delete the default subnet and create:

| Setting       | Value          |
| ------------- | -------------- |
| Subnet Name   | xfusion-subnet |
| Address Range | 10.0.0.0/24    |

Click **Review + Create** → **Create**.

#### Explanation

The Virtual Network provides network isolation, while the dedicated subnet hosts the VM.

---

### Step 4: Create the Storage Account (Azure Portal)

Navigate to:

```text
Azure Portal
→ Storage Accounts
→ Create
```

Configure:

| Setting         | Value                           |
| --------------- | ------------------------------- |
| Storage Account | xfusionstor5079                 |
| Region          | East US                         |
| Performance     | Standard                        |
| Redundancy      | Locally-redundant Storage (LRS) |

On the **Advanced** tab:

```text
Allow Blob Anonymous Access = Disabled
```

Leave all other settings as default and create the storage account.

#### Explanation

Disabling anonymous access ensures the storage account remains private while still allowing authenticated access using the account key.

---

### Step 5: Create a Private Blob Container

Open the Storage Account.

Navigate to:

```text
Data Storage
→ Containers
→ + Container
```

Configure:

| Setting          | Value                         |
| ---------------- | ----------------------------- |
| Name             | xfusion-container             |
| Anonymous Access | Private (No anonymous access) |

Create the container.

#### Explanation

The Blob container stores the static website file securely without exposing it publicly.

---

### Step 6: Upload index.html

Retrieve the Storage Account key.

```bash
KEY=$(az storage account keys list \
-g $RG \
-n xfusionstor5079 \
--query "[0].value" \
-o tsv)
```

Upload the file.

```bash
az storage blob upload \
--account-name xfusionstor5079 \
--account-key "$KEY" \
--container-name xfusion-container \
--name index.html \
--file /root/index.html
```

Verify the upload.

```bash
az storage blob list \
--account-name xfusionstor5079 \
--account-key "$KEY" \
--container-name xfusion-container \
-o table
```

#### Explanation

The uploaded `index.html` will later be downloaded by the VM and served using Nginx.

---

### Step 7: Create the Virtual Machine (Azure Portal)

Navigate to:

```text
Azure Portal
→ Virtual Machines
→ Create
```

Configure:

| Setting | Value                   |
| ------- | ----------------------- |
| VM Name | xfusion-vm              |
| Region  | East US                 |
| Image   | Ubuntu Server 22.04 LTS |
| Size    | Standard_B1s            |

Authentication:

| Setting             | Value                   |
| ------------------- | ----------------------- |
| Authentication Type | SSH Public Key          |
| Username            | azureuser               |
| SSH Key Source      | Use Existing Public Key |

Paste the contents of:

```bash
cat ~/.ssh/id_rsa.pub
```

Networking:

| Setting         | Value          |
| --------------- | -------------- |
| Virtual Network | xfusion-vnet   |
| Subnet          | xfusion-subnet |

Disks:

```text
OS Disk = Standard HDD (Standard_LRS)
```

Create the VM.

#### Explanation

The VM hosts the Nginx web server and securely retrieves the static webpage from Azure Blob Storage.

---

### Step 8: Verify or Open HTTP Port

If port 80 is not already allowed, create an inbound rule.

```bash
az network nsg rule create \
-g $RG \
--nsg-name xfusion-vm-nsg \
-n allow-http \
--priority 1001 \
--direction Inbound \
--access Allow \
--protocol Tcp \
--destination-port-ranges 80
```

If an HTTP rule already exists, skip this step.

#### Explanation

Allowing TCP port 80 enables external users to access the Nginx web server.

---

### Step 9: Retrieve the VM Public IP

```bash
IP=$(az vm show \
-g $RG \
-n xfusion-vm \
-d \
--query publicIps \
-o tsv)

echo $IP
```

#### Explanation

The public IP is used to connect to the VM and later verify the hosted website.

---

### Step 10: Connect to the VM

```bash
ssh azureuser@$IP
```

#### Explanation

This establishes an SSH session with the Ubuntu virtual machine.

---

### Step 11: Install Nginx

```bash
sudo apt update

sudo apt install nginx -y

sudo systemctl enable nginx

sudo systemctl start nginx
```

Verify:

```bash
systemctl status nginx
```

#### Explanation

Nginx will serve the downloaded `index.html` file from its default web root.

---

### Step 12: Install Azure CLI

```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

Verify:

```bash
az version
```

Login:

```bash
az login
```

#### Explanation

Azure CLI enables the VM to authenticate and download blobs from Azure Storage.

---

### Step 13: Retrieve the Storage Account Key

```bash
RG=$(az group list --query "[0].name" -o tsv)

KEY=$(az storage account keys list \
-g $RG \
-n xfusionstor5079 \
--query "[0].value" \
-o tsv)
```

#### Explanation

The Storage Account key authenticates the blob download operation.

---

### Step 14: Download index.html from Blob Storage

```bash
sudo az storage blob download \
--account-name xfusionstor5079 \
--account-key "$KEY" \
--container-name xfusion-container \
--name index.html \
--file /var/www/html/index.html
```

#### Explanation

The VM securely downloads the static webpage into Nginx's default document root.

---

### Step 15: Restart Nginx

```bash
sudo systemctl restart nginx
```

#### Explanation

Restarting Nginx ensures the newly downloaded webpage is served immediately.

---

### Step 16: Verify the Deployment

Verify locally:

```bash
curl http://localhost
```

Exit the VM:

```bash
exit
```

Verify externally:

```bash
curl http://$IP
```

or open:

```text
http://<VM_PUBLIC_IP>
```

#### Explanation

The browser or `curl` should display the contents of the uploaded `index.html`, confirming successful integration between Azure Blob Storage and the VM.

---

## Best Practices

* Keep Blob containers private and disable anonymous access.
* Use SSH key authentication instead of passwords.
* Store static assets separately from application source code.
* Retrieve Blob content securely using Azure CLI and Storage Account keys.
* Use Standard HDD disks to comply with Azure Policy restrictions.
* Verify Nginx status after configuration changes.
* Limit inbound NSG rules to only required ports such as SSH and HTTP.

## Key Learnings

* Created and configured an Azure Virtual Network and subnet.
* Provisioned a secure Azure Storage Account with private Blob Storage.
* Uploaded files to Azure Blob Storage using Azure CLI.
* Created an Ubuntu Virtual Machine using SSH public key authentication.
* Installed and configured Nginx on an Azure VM.
* Installed Azure CLI inside the VM.
* Securely downloaded a blob from Azure Storage to the VM.
* Hosted a static website locally using Nginx without enabling Azure Static Website hosting.
* Verified end-to-end connectivity between Azure Blob Storage and the Virtual Machine.
