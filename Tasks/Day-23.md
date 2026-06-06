# Day 23 – Automating User Data Configuration Using the CLI

---

## Task Overview  

The Nautilus DevOps Team is working on setting up a new virtual machine (VM) to host a web server for a critical application. The team lead has requested you to create an Azure VM that will serve as a web server using Nginx. This VM will be part of the initial infrastructure setup for the Nautilus project. Ensuring that the server is correctly configured and accessible from the internet is crucial for the upcoming deployment phase.

As a member of the Nautilus DevOps Team, your task is to create a VM using Azure CLI with the following specifications:

**Instance Name:** The VM must be named `devops-vm`.

**Image:** Use any available Ubuntu image to create this VM.

**Custom Script Extension/User Data:** Configure the VM to run a custom script during its launch. This script should:

  - Install the Nginx package.
  - Start the Nginx service.

**Network Security Group (NSG):** Ensure that the VM allows HTTP traffic on port 80 from the internet.

**Instructions:**

- Use Azure CLI commands to set up the VM in the specified configuration.

- Ensure the VM is accessible from the internet on port `80`.

- The Nginx service should be running after setup.

Use the Azure CLI commands to complete the task.

`Notes:`

- Create the resources only in the East US region.

- You may use the default resource group or create a new one if needed.

---

## Step-by-Step Implementation  

### Step 1: Verify Azure Login  

Run:

```bash
az account show
```

#### Explanation  
This command verifies that Azure CLI is authenticated and connected to a valid Azure subscription.

---

### Step 2: Identify the Resource Group  

Run:

```bash
az group list --output table
```

Example output:

```bash
kml-rg-12345
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
The Resource Group is required to create and manage Azure resources.

---

### Step 3: Create Cloud-Init Script  

Create a startup script file:

```bash
cat > nginx.sh << 'EOF'
#!/bin/bash
apt-get update -y
apt-get install -y nginx
systemctl enable nginx
systemctl start nginx
EOF
```

Verify:

```bash
cat nginx.sh
```

Expected output:

```bash
#!/bin/bash
apt-get update -y
apt-get install -y nginx
systemctl enable nginx
systemctl start nginx
```

#### Explanation  
This Cloud-Init script automatically installs and starts the Nginx web server during VM deployment.

---

### Step 4: Generate SSH Key Pair  

Check if an SSH key already exists:

```bash
ls ~/.ssh/id_rsa.pub
```

If no key exists, generate one:

```bash
ssh-keygen -t rsa -b 2048 -f ~/.ssh/id_rsa -N ""
```

#### Explanation  
The SSH key pair will be used for secure authentication to the Virtual Machine.

---

### Step 5: Create the Virtual Machine  

Run:

```bash
az vm create \
  --resource-group $RG \
  --name devops-vm \
  --location eastus \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --admin-username azureuser \
  --ssh-key-values ~/.ssh/id_rsa.pub \
  --custom-data nginx.sh \
  --storage-sku Standard_LRS \
  --public-ip-sku Standard
```

#### Important  

The parameter below is required to comply with lab storage policies:

```bash
--storage-sku Standard_LRS
```

#### Explanation  
This command creates the Ubuntu Virtual Machine and executes the Cloud-Init script during first boot.

---

### Step 6: Allow HTTP Traffic  

Run:

```bash
az vm open-port \
  --resource-group $RG \
  --name devops-vm \
  --port 80
```

#### Explanation  
This creates an NSG rule that allows inbound HTTP traffic from the internet.

---

### Step 7: Verify SSH Access Rule  

Run:

```bash
az vm open-port \
  --resource-group $RG \
  --name devops-vm \
  --port 22
```

#### Explanation  
This ensures SSH access is available for remote administration.

---

### Step 8: Retrieve Public IP Address  

Run:

```bash
az vm show \
  --resource-group $RG \
  --name devops-vm \
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
  --name devops-vm \
  -d \
  --query publicIps \
  -o tsv)

echo $IP
```

#### Explanation  
The Public IP is required to access the VM and verify the Nginx web server.

---

### Step 9: Allow Cloud-Init to Complete  

Wait a few minutes:

```bash
sleep 180
```

#### Explanation  
Cloud-Init requires time to update packages, install Nginx, and start the service.

---

### Step 10: Verify Nginx Service via SSH  

Connect to the VM:

```bash
ssh azureuser@$IP
```

Check Nginx status:

```bash
sudo systemctl status nginx
```

Expected output:

```text
active (running)
```

Exit the VM:

```bash
exit
```

#### Explanation  
This confirms that Nginx was installed successfully and is running.

---

### Step 11: Verify Nginx Listening on Port 80  

SSH into the VM and run:

```bash
sudo ss -tulpn | grep :80
```

Expected output:

```bash
LISTEN 0 511 0.0.0.0:80
```

#### Explanation  
This confirms that Nginx is actively listening for HTTP requests.

---

### Step 12: Verify Web Server Response  

From the Azure Client machine, run:

```bash
curl http://$IP
```

Expected output contains:

```html
Welcome to nginx!
```

#### Explanation  
This verifies that the web server is accessible from the internet and serving content correctly.

---

## Best Practices  

- Automate server provisioning using Cloud-Init scripts  
- Use Infrastructure as Code principles for repeatable deployments  
- Restrict network access to only required ports  
- Validate service health after automated deployments  
- Use SSH key authentication instead of passwords  
- Test application accessibility immediately after provisioning  

## Key Learnings  

- Azure CLI can fully automate Virtual Machine deployments  
- Cloud-Init enables automatic software installation during VM startup  
- Nginx can be provisioned without manual server configuration  
- NSG rules control inbound traffic to Azure Virtual Machines  
- Standard_LRS storage helps satisfy cloud governance policies  
- Automation improves deployment consistency and operational efficiency  
