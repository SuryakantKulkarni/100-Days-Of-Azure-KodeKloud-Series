# Day 48 – VM and ACR Integration for Storage

---

## Task Overview

The Nautilus DevOps team needs to set up an Azure Virtual Machine (VM) to interact with an Azure Blob Storage container for storing and retrieving data. The team must create a private storage account, configure Blob Storage, and test the functionality.

**Task:**

**1) Azure Virtual Machine Setup:**

- Create a VM named `xfusion-vm` in the **East US** region.
- Authentication: Use `SSH public key` authentication. (Please select `use existing public key` option, create public-key locally and paste contents of `~/.ssh/id_rsa.pub`).
- Install Docker and Azure CLI on the VM.
- Pull the Docker image from the ACR and run it on the VM, ensuring it listens on port 80.
  
**2) Azure Container Registry (ACR) Setup:**

- Create an ACR named `xfusionacr11650` in the **East US** region.
- The repository name should be `xfusion/python-app`.
- Build the Docker image using the Dockerfile already given under `/root/pyapp`.
- Push the Docker image to the ACR with the tag `latest`.

**3) Create a Storage Account and Blob Container:**

- Create a storage account named `xfusionstor11650` in the **East US** region with **Locally-redundant storage (LRS)**.
- Create a Blob container named `xfusion-config`.
- Upload a file named `config.json` (available under `/root/`) to the container.

**4) Validation:**

- Confirm that the application can be accessed on the browser.

`Notes:`
- Create all resources in the `East US` region.
- Use the Azure CLI or Azure Portal for resource creation.
- The **Dockerfile** is already given under `/root/pyapp`. The image tag must be `latest`.
- The repository name should be `xfusion/python-app`.

---

# Step-by-Step Implementation

### Step 1: Get the Resource Group

```bash
RG=$(az group list --query "[0].name" -o tsv)

echo $RG
```

#### Explanation

Retrieves the default Resource Group where all Azure resources will be created.

---

### Step 2: Generate SSH Key

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""
```

If prompted:

```text
Overwrite (y/n)?
```

Type:

```text
n
```

Display the public key:

```bash 
cat ~/.ssh/id_rsa.pub
```

#### Explanation

Generates (or reuses) an SSH key pair for authenticating to the virtual machine.

---

### Step 3: Create the Virtual Machine

```bash
az vm create \
--resource-group $RG \
--name xfusion-vm \
--location eastus \
--image Ubuntu2204 \
--size Standard_B1s \
--admin-username azureuser \
--authentication-type ssh \
--ssh-key-values ~/.ssh/id_rsa.pub \
--storage-sku Standard_LRS
```

#### Explanation

Creates an Ubuntu virtual machine using SSH public key authentication.

---

### Step 4: Retrieve the VM Public IP

```bash
IP=$(az vm show \
--resource-group $RG \
--name xfusion-vm \
--show-details \
--query publicIps \
-o tsv)

echo $IP
```

#### Explanation

Stores the VM's public IP address for SSH access.

---

### Step 5: Open Required Ports

Open SSH:

```bash
az vm open-port \
--resource-group $RG \
--name xfusion-vm \
--port 22
```

If HTTP is not already open:

```bash
az network nsg rule create \
--resource-group $RG \
--nsg-name xfusion-vmNSG \
--name allow-http \
--priority 1001 \
--direction Inbound \
--access Allow \
--protocol Tcp \
--destination-port-ranges 80
```

#### Explanation

Allows SSH and HTTP traffic to reach the VM.

---

### Step 6: Connect to the VM

```bash
ssh azureuser@$IP
```

#### Explanation

Logs into the virtual machine.

---

### Step 7: Install Docker

```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker azureuser
newgrp docker
```

Verify:

```bash
docker --version
```

#### Explanation

Installs Docker and configures it for the current user.

---

### Step 8: Install Azure CLI

```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

Verify:

```bash
az version
```

Exit the VM:

```bash
exit
```

#### Explanation

Installs Azure CLI to interact with Azure services directly from the VM.

---

### Step 9: Create Azure Container Registry

```bash
az acr create \
--resource-group $RG \
--name xfusionacr11650 \
--location eastus \
--sku Basic
```

#### Explanation

Creates an Azure Container Registry using the Basic pricing tier.

---

### Step 10: Login to ACR

```bash
az acr login \
--name xfusionacr11650
```

#### Explanation

Authenticates Docker with the Azure Container Registry.

---

### Step 11: Build the Docker Image

```bash
cd /root/pyapp
```

```bash
docker build \
-t xfusion/python-app:latest .
```

#### Explanation

Builds the Docker image using the Dockerfile located in `/root/pyapp`.

---

### Step 12: Tag the Docker Image

```bash
docker tag \
xfusion/python-app:latest \
xfusionacr11650.azurecr.io/xfusion/python-app:latest
```

#### Explanation

Tags the image with the Azure Container Registry repository name.

---

### Step 13: Push the Docker Image

```bash
docker push \
xfusionacr11650.azurecr.io/xfusion/python-app:latest
```

Verify:

```bash
az acr repository list \
--name xfusionacr11650 \
-o table
```

Expected:

```text
xfusion/python-app
```

#### Explanation

Pushes the Docker image into Azure Container Registry.

---

### Step 14: Create the Storage Account

```bash
az storage account create \
--resource-group $RG \
--name xfusionstor11650 \
--location eastus \
--sku Standard_LRS \
--kind StorageV2
```

#### Explanation

Creates a Storage Account using Locally Redundant Storage (LRS).

---

### Step 15: Retrieve the Storage Account Key

```bash 
KEY=$(az storage account keys list \
--resource-group $RG \
--account-name xfusionstor11650 \
--query "[0].value" \
-o tsv)

echo $KEY
```

#### Explanation

Retrieves the access key required for Blob Storage operations.

---

### Step 16: Create the Blob Container

```bash
az storage container create \
--account-name xfusionstor11650 \
--account-key "$KEY" \
--name xfusion-config
```

#### Explanation

Creates the Blob container used to store the application configuration file.

---

### Step 17: Upload config.json

```bash
az storage blob upload \
--account-name xfusionstor11650 \
--account-key "$KEY" \
--container-name xfusion-config \
--name config.json \
--file /root/config.json
```

Verify:

```bash
az storage blob list \
--account-name xfusionstor11650 \
--account-key "$KEY" \
--container-name xfusion-config \
-o table
```

Expected:

```text
config.json
```

#### Explanation

Uploads the configuration file to Azure Blob Storage.

---

### Step 18: Login to the VM Again

```bash 
ssh azureuser@$IP
```

Login to Azure:

```bash
az login
```

#### Explanation

Authenticates the VM with Azure.

---

### Step 19: Login to ACR from the VM

```bash
az acr login \
--name xfusionacr11650
```

#### Explanation

Allows Docker on the VM to pull images from Azure Container Registry.

---

### Step 20: Pull the Docker Image

```bash
docker pull \
xfusionacr11650.azurecr.io/xfusion/python-app:latest
```

#### Explanation

Downloads the application image from Azure Container Registry.

---

### Step 21: Run the Docker Container

```bash 
docker run -d \
--name python-app \
-p 80:80 \
--restart unless-stopped \
xfusionacr11650.azurecr.io/xfusion/python-app:latest
```

#### Explanation

Starts the application container and maps port **80**.

---

### Step 22: Copy config.json to the VM

From the Azure client:

```bash
scp /root/config.json \
azureuser@$IP:/home/azureuser/
```

#### Explanation

Transfers the configuration file to the VM.

---

### Step 23: Copy config.json into the Running Container

SSH into the VM if required:

```bash
ssh azureuser@$IP
```

Copy the file:

```bash
docker cp \
/home/azureuser/config.json \
python-app:/app/config.json
```

Restart the container:

```bash
docker restart python-app
```

#### Explanation

Places the configuration file inside the running container where the application expects it.

---

### Step 24: Test the Application

On the VM:

```bash
curl http://localhost
```

Expected:

```text
Welcome to KKE Azure Labs: {'key': 'value', 'version': 1}
```

Test externally:

```bash
curl http://<VM_PUBLIC_IP>
```

Expected:

```text
Welcome to KKE Azure Labs: {'key': 'value', 'version': 1}
```

#### Explanation

Verifies that the application is accessible locally and externally.

---

## Azure Portal (GUI) Steps

### Create the Virtual Machine

Navigate to:

```text
Azure Portal
→ Virtual Machines
→ Create
```

Configure:

| Setting        | Value                   |
| -------------- | ----------------------- |
| VM Name        | xfusion-vm              |
| Region         | East US                 |
| Image          | Ubuntu Server 22.04 LTS |
| Size           | Standard_B1s            |
| Authentication | SSH Public Key          |
| Username       | azureuser               |
| OS Disk        | Standard HDD            |

Allow inbound ports:

* SSH (22)
* HTTP (80)

Deploy the VM.

---

### Create Azure Container Registry

Navigate to:

```text
Container Registries
→ Create
```

Configure:

| Setting | Value           |
| ------- | --------------- |
| Name    | xfusionacr11650 |
| Region  | East US         |
| SKU     | Basic           |

Deploy the registry.

---

### Create Storage Account

Navigate to:

```text
Storage Accounts
→ Create
```

Configure:

| Setting     | Value            |
| ----------- | ---------------- |
| Name        | xfusionstor11650 |
| Region      | East US          |
| Performance | Standard         |
| Redundancy  | LRS              |

Deploy the Storage Account.

---

### Create Blob Container

Navigate to:

```text
Storage Account
→ Containers
→ + Container
```

Configure:

| Setting       | Value          |
| ------------- | -------------- |
| Name          | xfusion-config |
| Public Access | Private        |

Create the container.

---

### Upload config.json

Open:

```text
xfusion-config
→ Upload
```

Select:

```text 
/root/config.json
```

Upload the file.

---

## Best Practices

* Use SSH key authentication instead of passwords.
* Store container images securely in Azure Container Registry.
* Keep Blob containers private unless public access is required.
* Store application configuration outside the image for easier updates.
* Verify container health after deployment.

## Key Learnings

* Creating Azure Virtual Machines.
* Installing Docker and Azure CLI on Linux.
* Building and pushing Docker images to Azure Container Registry.
* Creating Azure Storage Accounts and Blob Containers.
* Uploading files to Azure Blob Storage.
* Pulling container images from ACR.
* Running Docker containers on Azure Virtual Machines.
* Managing application configuration files inside containers.
