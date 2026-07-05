# Day 45 – Azure Kubernetes Service (AKS) Setup and Management

---

## Task Overview

The Nautilus DevOps team is tasked with preparing an AKS cluster to deploy a Kubernetes-based application. The team has the following requirements: 

- Create an AKS cluster named `nautilus-aks`.
- The Kubernetes version must be `1.33.0`.
- The AKS cluster endpoint access must be private.
- Ensure the cluster is created in the `Central US` region.
- Edit the `agentpool` Node pools (delete all other node pool if exists) and configure the cluster with the following properties:

  - Node size: `D2s v3`.
  - Minimum node count: `1`.
  - Maximum node count: `2`.

- Disable the `Container Insights` for now and disable all kind of monitoring as well.

The AKS cluster must be configured with high availability and private endpoint access. Verify that the cluster meets the requirements and is ready for workloads.  

`Notes:` 
- Create the resources only in the `Central US` region.
- Ensure that the Kubernetes version is `1.33.0`.

---

# Step-by-Step Implementation

### Step 1: Get the Resource Group

```bash
RG=$(az group list --query "[0].name" -o tsv)

echo $RG
```

#### Explanation

Retrieves the default Resource Group where the AKS cluster will be created.

---

### Step 2: Verify Available Kubernetes Versions

```bash
az aks get-versions \
--location centralus \
-o table
```

#### Explanation

Lists the Kubernetes versions available in the **Central US** region. Ensure version **1.33.0** is available.

---

## Method 1 – Azure Portal (Recommended)

### Step 3: Create the AKS Cluster

Navigate to:

```text
Azure Portal
→ Kubernetes Services
→ Create
→ Azure Kubernetes Service
```

---

### Basics

Configure the following:

| Field              | Value           |
| ------------------ | --------------- |
| Resource Group     | Existing Lab RG |
| Cluster Preset     | Dev/Test        |
| Cluster Name       | `nautilus-aks`  |
| Region             | Central US      |
| Kubernetes Version | **1.33.0**      |
| Automatic Upgrade  | Disabled        |

Click:

```text
Next : Node Pools
```

#### Explanation

Creates the AKS cluster in the required region using Kubernetes version **1.33.0**.

---

### Step 4: Configure the Node Pool

Configure the default **agentpool**:

| Field                     | Value           |
| ------------------------- | --------------- |
| Node Pool Name            | `agentpool`     |
| Mode                      | System          |
| VM Size                   | Standard_D2s_v3 |
| Enable Cluster Autoscaler | Yes             |
| Minimum Node Count        | 1               |
| Maximum Node Count        | 2               |

If any additional node pools exist, **delete them** so only **agentpool** remains.

Click:

```text
Next : Access
```

#### Explanation

Creates a single system node pool with autoscaling between **1 and 2 nodes**.

---

### Step 5: Configure Access

Leave all settings as default.

Click:

```text
Next : Networking
```

---

### Step 6: Configure Networking

Configure:

| Field             | Value               |
| ----------------- | ------------------- |
| API Server Access | **Private Cluster** |
| Private DNS Zone  | System Managed      |

Click:

```text
Next : Integrations
```

#### Explanation

Creates a **private AKS cluster**, ensuring the Kubernetes API server is not publicly accessible.

---

### Step 7: Disable Monitoring

Disable all monitoring services:

* ❌ Container Insights
* ❌ Azure Monitor
* ❌ Managed Prometheus
* ❌ Microsoft Defender

Click:

```text
Review + Create
```

Then click:

```text
Create
```

Wait until the deployment completes.

#### Explanation

Disables monitoring services as required by the lab.

---

### Step 8: Verify the Cluster

Open:

```text
Azure Portal
→ Kubernetes Services
→ nautilus-aks
```

Verify:

| Property           | Expected   |
| ------------------ | ---------- |
| Status             | Running    |
| Region             | Central US |
| Kubernetes Version | 1.33.0     |
| Private Cluster    | Enabled    |

---

### Step 9: Verify the Node Pool

Navigate to:

```text
Node Pools
```

Verify:

| Property      | Expected        |
| ------------- | --------------- |
| Node Pool     | agentpool       |
| VM Size       | Standard_D2s_v3 |
| Autoscaling   | Enabled         |
| Minimum Nodes | 1               |
| Maximum Nodes | 2               |

---

### Step 10: Verify Monitoring

Navigate to:

```text
Monitoring
→ Insights
```

Verify:

```text
Container Insights
Not Enabled
```

---

# Method 2 – Azure CLI

### Step 11: Create the AKS Cluster

```bash
az aks create \
-g $RG \
-n nautilus-aks \
-l centralus \
--kubernetes-version 1.33.0 \
--node-vm-size Standard_D2s_v3 \
--node-count 1 \
--enable-cluster-autoscaler \
--min-count 1 \
--max-count 2 \
--enable-private-cluster \
--private-dns-zone system \
--network-plugin azure \
--enable-managed-identity \
--generate-ssh-keys
```

#### Explanation

Creates a private AKS cluster with the required Kubernetes version and autoscaling configuration.

---

### Step 12: Disable Container Insights

```bash
az aks disable-addons \
-g $RG \
-n nautilus-aks \
-a monitoring
```

#### Explanation

Disables Azure Monitor and Container Insights.

---

### Step 13: Verify the AKS Cluster

```bash
az aks show \
-g $RG \
-n nautilus-aks \
--query "{Name:name,Location:location,Version:kubernetesVersion,Private:apiServerAccessProfile.enablePrivateCluster}" \
-o table
```

Expected:

```text
Name           Location    Version    Private
-------------  ----------  ---------  -------
nautilus-aks   centralus   1.33.0     true
```

---

### Step 14: Verify the Node Pool

```bash
az aks show \
-g $RG \
-n nautilus-aks \
--query "agentPoolProfiles[].{Name:name,Mode:mode,VMSize:vmSize,AutoScaling:enableAutoScaling,Min:minCount,Max:maxCount,Count:count}" \
-o table
```

Expected:

```text
Name       Mode    VMSize             AutoScaling   Min   Max   Count
---------  ------  -----------------  ------------  ----  ----  -----
agentpool  System  Standard_D2s_v3    true          1     2     1
```

---

### Step 15: Verify Monitoring

```bash
az aks show \
-g $RG \
-n nautilus-aks \
--query "addonProfiles.omsagent.enabled"
```

Expected:

```text
false
```

or

```text
null
```

---

### Step 16: Verify Cluster Health

```bash
az aks show \
-g $RG \
-n nautilus-aks \
--query "{ProvisioningState:provisioningState,PowerState:powerState.code}" \
-o table
```

Expected:

```text
ProvisioningState    PowerState
------------------   ----------
Succeeded            Running
```

---

## Common Errors & Fixes

### Error: Kubernetes Version Not Available

**Cause**

Version **1.33.0** is unavailable in the selected region.

**Fix**

Run:

```bash
az aks get-versions \
-l centralus \
-o table
```

Confirm the version before creating the cluster.

---

### Error: Private Cluster Creation Failed

**Cause**

Private DNS Zone configuration issue.

**Fix**

Use:

```text
Private DNS Zone
→ System Managed
```

---

### Error: Monitoring Still Enabled

**Cause**

Container Insights enabled during creation.

**Fix**

Disable it:

```bash
az aks disable-addons \
-g $RG \
-n nautilus-aks \
-a monitoring
```

---

### Error: Additional Node Pools Exist

**Cause**

Extra node pools were created.

**Fix**

Delete them from:

```text
AKS Cluster
→ Node Pools
→ Delete
```

Ensure only **agentpool** remains.

---

## Best Practices

* Use **Private Cluster** for improved security.
* Enable autoscaling to optimize compute costs.
* Keep only one system node pool unless additional workloads require separate pools.
* Disable monitoring if not required to reduce resource usage.
* Verify Kubernetes version availability before deployment.

## Key Learnings

* Creating Azure Kubernetes Service (AKS) clusters.
* Configuring private AKS API server access.
* Managing node pools and autoscaling.
* Disabling Azure Monitor and Container Insights.
* Verifying AKS cluster health using Azure CLI and Azure Portal.
* Preparing AKS clusters for production-ready workloads.
