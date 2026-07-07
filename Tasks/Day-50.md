# Day 50 – VM Setup and Configuration for Azure Application Gateway

---

## Task Overview

The Nautilus DevOps team needs to set up an Azure Application Gateway to manage traffic for a backend pool of virtual machines. The gateway will serve as a load balancer, distributing traffic across the VMs.

**Task:**

**1) Azure Virtual Network and Subnet:**

- Create a Virtual Network (VNet) named `datacenter-vnet` in the **East US** region.
- Create a Subnet named `datacenter-subnet` within the VNet for the VMs.
- Create a Subnet named `datacenter-apgw-subnet` within the VNet for the Application Gateway.

**2) Azure Virtual Machines:**

- Create two VMs named `datacenter-vm1` and `datacenter-vm2` in the **East US** region.
- Install **Nginx** on both VMs.
- Configure `index.html` on VM1 to display **"Welcome to KKE Labs:Version 1"**.
- Configure `index.html` on VM2 to display **"Welcome to KKE Labs:Version 2"**.
  
**3) Azure Application Gateway:**

- Create an Application Gateway named `datacenter-apgw` in the **East US** region.
- Assign the `datacenter-apgw-subnet` to the Application Gateway.
- Create a frontend IP configuration named `datacenter-apgw-ip`.
- Add the VMs `datacenter-vm1` and `datacenter-vm2` to the backend pool.
- Configure a basic routing rule to distribute traffic between the VMs.

**4) Validation:**

- Verify that the Application Gateway distributes traffic to both VMs.
- Ensure that accessing the Application Gateway URL displays either **"Welcome to KKE Labs:Version 1"** or **"Welcome to KKE Labs:Version 2"** depending on the load balancing.

`Notes:`
- Create all resources in the `East US` region.
- Use the Azure Portal or Azure CLI for resource creation.
- Ensure proper routing and traffic distribution through the Application Gateway.

---

# Step-by-Step Implementation

### Step 1: Get the Resource Group

Retrieve the default Resource Group used in the lab.

```bash
RG=$(az group list --query "[0].name" -o tsv)

echo $RG
```

#### Explanation

The Resource Group is stored in the `RG` variable and is used throughout the deployment.

---

### Step 2: Generate an SSH Key Pair

Generate an SSH key if one does not already exist.

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""
```

Display the public key.

```bash
cat ~/.ssh/id_rsa.pub
```

#### Explanation

The public key is required while creating both virtual machines using **SSH Public Key Authentication**.

---

### Step 3: Create the Virtual Network (Azure Portal)

Navigate to:

```text
Azure Portal
→ Virtual Networks
→ Create
```

Configure the following:

| Setting | Value           |
| ------- | --------------- |
| Name    | datacenter-vnet |
| Region  | East US         |

Click **Next: IP Addresses**.

Delete the default subnet and create the following subnets.

| Subnet Name            | Address Range |
| ---------------------- | ------------- |
| datacenter-subnet      | 10.0.0.0/24   |
| datacenter-apgw-subnet | 10.0.1.0/24   |

Click **Review + Create** → **Create**.

#### Explanation

The VM subnet hosts the backend Virtual Machines, while the dedicated Application Gateway subnet is required for Azure Application Gateway deployment.

---

### Step 4: Create Virtual Machine 1 (Azure Portal)

Navigate to:

```text
Azure Portal
→ Virtual Machines
→ Create
```

Configure:

| Setting | Value                   |
| ------- | ----------------------- |
| VM Name | datacenter-vm1          |
| Region  | East US                 |
| Image   | Ubuntu Server 22.04 LTS |
| Size    | Standard_B1s            |

Authentication:

| Setting             | Value                   |
| ------------------- | ----------------------- |
| Authentication Type | SSH Public Key          |
| Username            | azureuser               |
| SSH Key Source      | Use Existing Public Key |

Paste the output of:

```bash
cat ~/.ssh/id_rsa.pub
```

Disk:

```text
Standard HDD (Standard_LRS)
```

Networking:

| Setting         | Value             |
| --------------- | ----------------- |
| Virtual Network | datacenter-vnet   |
| Subnet          | datacenter-subnet |

Allow SSH during creation.

Create the VM.

#### Explanation

This VM will serve **Version 1** of the web application.

---

### Step 5: Create Virtual Machine 2

Repeat the previous step.

Only change:

| Setting | Value          |
| ------- | -------------- |
| VM Name | datacenter-vm2 |

#### Explanation

This VM serves **Version 2** of the application.

---

### Step 6: Allow HTTP Traffic

Open port **80** on both VMs.

```bash
az vm open-port \
-g $RG \
-n datacenter-vm1 \
--port 80
```

```bash
az vm open-port \
-g $RG \
-n datacenter-vm2 \
--port 80
```

#### Explanation

This allows the Application Gateway to communicate with the backend web servers.

---

### Step 7: Retrieve Public IP Addresses

VM1:

```bash
IP1=$(az vm show \
-g $RG \
-n datacenter-vm1 \
-d \
--query publicIps \
-o tsv)

echo $IP1
```

VM2:

```bash
IP2=$(az vm show \
-g $RG \
-n datacenter-vm2 \
-d \
--query publicIps \
-o tsv)

echo $IP2
```

#### Explanation

These IP addresses are used to SSH into each Virtual Machine.

---

### Step 8: Configure VM1

Connect to VM1.

```bash
ssh azureuser@$IP1
```

Install Nginx.

```bash
sudo apt update

sudo apt install nginx -y
```

Create the web page.

```bash
echo "Welcome to KKE Labs:Version 1" | sudo tee /var/www/html/index.html
```

Restart Nginx.

```bash
sudo systemctl restart nginx
```

Verify.

```bash
curl localhost
```

Exit.

```bash
exit
```

#### Explanation

VM1 now serves **Version 1** through Nginx.

---

### Step 9: Configure VM2

Connect.

```bash
ssh azureuser@$IP2
```

Install Nginx.

```bash
sudo apt update

sudo apt install nginx -y
```

Create the web page.

```bash
echo "Welcome to KKE Labs:Version 2" | sudo tee /var/www/html/index.html
```

Restart Nginx.

```bash
sudo systemctl restart nginx
```

Verify.

```bash
curl localhost
```

Exit.

```bash
exit
```

#### Explanation

VM2 now serves **Version 2**.

---

### Step 10: Create the Application Gateway (Azure Portal)

Navigate to:

```text
Azure Portal
→ Application Gateways
→ Create
```

Configure:

| Setting        | Value           |
| -------------- | --------------- |
| Name           | datacenter-apgw |
| Region         | East US         |
| Tier           | Basic           |
| Instance Count | 1               |

#### Explanation

The Basic SKU satisfies the Azure Lab policy and provides Layer 7 HTTP load balancing.

---

### Step 11: Configure the Frontend

Create a new Public IP.

| Setting        | Value              |
| -------------- | ------------------ |
| Public IP Name | datacenter-apgw-ip |
| Assignment     | Static             |

Continue.

#### Explanation

The frontend Public IP receives incoming client requests.

---

### Step 12: Configure the Backend Pool

Create a Backend Pool.

| Setting           | Value       |
| ----------------- | ----------- |
| Backend Pool Name | backendpool |

Add:

* datacenter-vm1
* datacenter-vm2

#### Explanation

The backend pool contains both web servers that receive traffic from the Application Gateway.

---

### Step 13: Configure Backend HTTP Settings

Create a new backend setting.

| Setting         | Value        |
| --------------- | ------------ |
| Name            | http-setting |
| Protocol        | HTTP         |
| Backend Port    | 80           |
| Cookie Affinity | Disabled     |

Save.

#### Explanation

These settings define how the Application Gateway communicates with the backend web servers.

---

### Step 14: Configure the Listener

Create a Listener.

| Setting       | Value     |
| ------------- | --------- |
| Listener Name | listener1 |
| Protocol      | HTTP      |
| Port          | 80        |
| Listener Type | Basic     |

#### Explanation

The listener accepts incoming HTTP requests on port 80.

---

### Step 15: Configure the Routing Rule

Create a Routing Rule.

| Setting         | Value        |
| --------------- | ------------ |
| Rule Name       | basic-rule   |
| Priority        | 100          |
| Listener        | listener1    |
| Backend Pool    | backendpool  |
| Backend Setting | http-setting |

For networking:

| Setting         | Value                  |
| --------------- | ---------------------- |
| Virtual Network | datacenter-vnet        |
| Gateway Subnet  | datacenter-apgw-subnet |

Click **Review + Create** and deploy.

#### Explanation

The routing rule connects the frontend listener with the backend pool, enabling load-balanced traffic routing.

---

### Step 16: Verify Backend Health

```bash
az network application-gateway show-backend-health \
-g $RG \
-n datacenter-apgw
```

#### Explanation

Both backend Virtual Machines should report a **Healthy** status.

---

### Step 17: Verify the Deployment

Retrieve the Application Gateway Public IP.

```bash
az network public-ip list \
-g $RG \
-o table
```

Open:

```text
http://<ApplicationGatewayPublicIP>
```

or

```bash
curl http://<ApplicationGatewayPublicIP>
```

Refresh multiple times.

Expected output:

```text
Welcome to KKE Labs:Version 1
```

or

```text
Welcome to KKE Labs:Version 2
```

#### Explanation

Receiving both responses confirms that the Application Gateway is successfully distributing traffic between the two backend Virtual Machines.

---

## Best Practices

* Use a dedicated subnet for the Application Gateway.
* Use SSH public key authentication instead of passwords.
* Select the **Basic SKU** to comply with Azure Lab policy.
* Configure identical backend HTTP settings across all backend VMs.
* Verify backend health before testing the public endpoint.
* Keep Nginx running as a system service.
* Open only the required ports (22 and 80) in the Network Security Group.

## Key Learnings

* Created an Azure Virtual Network with dedicated subnets.
* Provisioned multiple Ubuntu Virtual Machines.
* Configured Nginx web servers with custom web pages.
* Created an Azure Application Gateway using the Basic SKU.
* Configured frontend IP, backend pool, HTTP settings, listener, and routing rule.
* Implemented Layer 7 load balancing across multiple backend servers.
* Verified backend health using Azure CLI.
* Validated Application Gateway traffic distribution using the public endpoint.
