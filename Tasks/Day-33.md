# Day 33 – Integrating Virtual Machines with Azure Load Balancer

---

## Task Overview

The Nautilus DevOps team is currently working on setting up a simple application on the Azure cloud. They aim to establish an Azure Load Balancer in front of a Virtual Machine (VM) where an Nginx server is currently running. While the Nginx server currently serves a sample page, the team plans to deploy the actual application later. 

- Set up an **Azure Load Balancer** named `nautilus-lb`.

- Configure the Load Balancer’s **frontend IP configuration** with the name `nautilus-lb-ip` and assign a public IP address with the same name `(nautilus-lb-ip)`. 

- Create a **backend pool** named `nautilus-backend-pool` and add the VM running Nginx to this pool. 

- Create a **health probe** named `nautilus-health-probe` on port `80` to check the VM's health. 

- Set up a **load balancer rule** named `nautilus-lb-rule` to route traffic on port `80` to the backend pool on port `80`. 

- Add an inbound rule to the existing NSG of the VM to allow HTTP traffic on port 80.

---

# Method 1: Using Azure Portal (GUI)

## Step 1: Open Azure Portal

Login to Azure Portal using your lab credentials.

Navigate to:

```text
Load Balancers
```

Click:

```text
+ Create
```

---

## Step 2: Create Public IP Address

Under the Basics tab:

| Field | Value |
|---------|---------|
| Resource Group | Existing Resource Group |
| Name | `nautilus-lb-ip` |
| Region | Same region as VM |
| SKU | Standard |
| Assignment | Static |

Click:

```text
Review + Create
→ Create
```

Wait for deployment to complete.

---

## Step 3: Create Load Balancer

Navigate to:

```text
Load Balancers
→ Create
```

Fill the details:

| Field | Value |
|---------|---------|
| Name | `nautilus-lb` |
| Region | Same region as VM |
| SKU | Standard |
| Type | Public |

Frontend IP Configuration:

| Field | Value |
|---------|---------|
| Name | `nautilus-lb-ip` |
| Public IP Address | `nautilus-lb-ip` |

Backend Pool:

| Field | Value |
|---------|---------|
| Name | `nautilus-backend-pool` |

Click:

```text
Review + Create
→ Create
```

---

## Step 4: Add VM to Backend Pool

Open:

```text
nautilus-lb
→ Backend Pools
```

Select:

```text
nautilus-backend-pool
```

Click:

```text
Add
```

Choose the existing VM NIC.

Save the configuration.

---

## Step 5: Create Health Probe

Open:

```text
nautilus-lb
→ Health Probes
→ Add
```

Configure:

| Field | Value |
|---------|---------|
| Name | `nautilus-health-probe` |
| Protocol | TCP |
| Port | 80 |

Click:

```text
Add
```

---

## Step 6: Create Load Balancing Rule

Open:

```text
Load Balancing Rules
→ Add
```

Fill:

| Field | Value |
|---------|---------|
| Name | `nautilus-lb-rule` |
| Frontend IP | `nautilus-lb-ip` |
| Backend Pool | `nautilus-backend-pool` |
| Protocol | TCP |
| Frontend Port | 80 |
| Backend Port | 80 |
| Health Probe | `nautilus-health-probe` |

Click:

```text
Add
```

---

## Step 7: Configure NSG Rule

Navigate to:

```text
Network Security Groups
```

Open the NSG attached to the VM.

Select:

```text
Inbound Security Rules
→ Add
```

Configure:

| Field | Value |
|---------|---------|
| Source | Any |
| Source Port Ranges | * |
| Destination | Any |
| Destination Port | 80 |
| Protocol | TCP |
| Action | Allow |
| Priority | 300 |
| Name | Allow-HTTP |

Click:

```text
Add
```

---

## Step 8: Verify Nginx Service

SSH into the VM and verify:

```bash
sudo systemctl status nginx
```

Expected:

```text
active (running)
```

Verify locally:

```bash
curl localhost
```

Expected:

```html
Welcome to nginx!
```

---

## Step 9: Verify Load Balancer

Copy the Public IP assigned to:

```text
nautilus-lb-ip
```

Open:

```text
http://<LOAD_BALANCER_PUBLIC_IP>
```

Expected:

```html
Welcome to nginx!
```

Task completed successfully.

---

# Method 2: Using Azure CLI

## Step 1: Get Resource Group

```bash
RG=$(az group list --query "[0].name" -o tsv)

echo $RG
```

---

## Step 2: Create Public IP

```bash
az network public-ip create \
  --resource-group $RG \
  --name nautilus-lb-ip \
  --sku Standard \
  --allocation-method Static
```

---

## Step 3: Create Load Balancer

```bash
az network lb create \
  --resource-group $RG \
  --name nautilus-lb \
  --sku Standard \
  --public-ip-address nautilus-lb-ip \
  --frontend-ip-name nautilus-lb-ip \
  --backend-pool-name nautilus-backend-pool
```

---

## Step 4: Add VM NIC to Backend Pool

List NICs:

```bash
az network nic list -o table
```

Attach NIC:

```bash
az network nic ip-config address-pool add \
  --resource-group $RG \
  --nic-name nautilus-vmVMNic \
  --ip-config-name ipconfig1 \
  --lb-name nautilus-lb \
  --address-pool nautilus-backend-pool
```

---

## Step 5: Create Health Probe

```bash
az network lb probe create \
  --resource-group $RG \
  --lb-name nautilus-lb \
  --name nautilus-health-probe \
  --protocol Tcp \
  --port 80
```

---

## Step 6: Create Load Balancing Rule

```bash
az network lb rule create \
  --resource-group $RG \
  --lb-name nautilus-lb \
  --name nautilus-lb-rule \
  --protocol Tcp \
  --frontend-port 80 \
  --backend-port 80 \
  --frontend-ip-name nautilus-lb-ip \
  --backend-pool-name nautilus-backend-pool \
  --probe-name nautilus-health-probe
```

---

## Step 7: Create NSG Rule

```bash
az network nsg rule create \
  --resource-group $RG \
  --nsg-name nautilus-vmNSG \
  --name Allow-HTTP \
  --priority 300 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --destination-port-ranges 80
```

---

## Step 8: Verify Load Balancer Components

Verify backend pool:

```bash
az network lb address-pool list \
  --resource-group $RG \
  --lb-name nautilus-lb \
  -o table
```

Verify probe:

```bash
az network lb probe list \
  --resource-group $RG \
  --lb-name nautilus-lb \
  -o table
```

Verify rule:

```bash
az network lb rule list \
  --resource-group $RG \
  --lb-name nautilus-lb \
  -o table
```

---

## Step 9: Get Load Balancer Public IP

```bash
az network public-ip show \
  --resource-group $RG \
  --name nautilus-lb-ip \
  --query ipAddress \
  -o tsv
```

---

## Step 10: Test Load Balancer

```bash
curl http://<LOAD_BALANCER_PUBLIC_IP>
```

Expected:

```html
Welcome to nginx!
```

---

## Best Practices

- Use health probes to monitor backend service availability.
- Use Standard SKU Load Balancers for production workloads.
- Restrict NSG rules to only required ports.
- Verify backend pool configuration after deployment.
- Monitor Load Balancer metrics and backend health regularly.
- Test application access through the Load Balancer after configuration.

## Key Learnings

- Azure Load Balancer distributes traffic across backend resources.
- Frontend IP configurations provide external connectivity.
- Backend pools define the target resources for traffic distribution.
- Health probes determine backend availability.
- NSG rules directly affect application accessibility.
- Load Balancers improve availability and scalability of applications.
