# Day 31 – Deploying and Managing a Web Application

---

## Task Overview

The Nautilus DevOps team is tasked with deploying a Python-based web application on Azure. You need to create a web app using the following specifications:

1) The **Web App** name should be `datacenter-webapp`.

2) It should be created in the **central US** region under the **default resource group**.

3) The **publish** option should be set to `Code`.

4) The **Runtime Stack** should be `Python` with `Linux` as the operating system.

5) Create a new **App Service Plan** named `datacenter-learn-python` with the SKU `Basic B1`.

6) **Application Insights** should be disabled.

7) Add **tags**:

    - Name: WebAppLearning
    - Environment: Dev

Make sure the web app is in `Running` state after creation.

---

# Method 1: Azure Portal (GUI)

### Step 1: Open App Services

In Azure Portal navigate to:

```text
App Services
→ Create
→ Web App
```

#### Explanation

This opens the Web App deployment wizard used to create Azure-hosted web applications.

---

### Step 2: Configure Basic Details

Under the **Basics** tab configure the following:

#### Project Details

| Field | Value |
|---|---|
| Resource Group | Default Resource Group |

#### Web App Details

| Field | Value |
|---|---|
| Name | `datacenter-webapp` |
| Publish | `Code` |
| Runtime Stack | `Python 3.x` |
| Operating System | `Linux` |
| Region | `Central US` |

#### Explanation

These settings define the application name, deployment method, runtime platform, and hosting region.

---

### Step 3: Create App Service Plan

Under App Service Plan click:

```text
Create New
```

Configure:

| Field | Value |
|---|---|
| Plan Name | `datacenter-learn-python` |

Click:

```text
Change Size
```

Select:

```text
Basic
→ B1
```

Click:

```text
Apply
```

#### Explanation

The App Service Plan defines the compute resources allocated to the Web App.

---

### Step 4: Configure Monitoring

Navigate to:

```text
Monitoring
```

Set:

```text
Application Insights = No
```

or

```text
Disable
```

#### Explanation

The task explicitly requires Application Insights to remain disabled.

---

### Step 5: Configure Tags

Navigate to:

```text
Tags
```

Add the following tags:

| Name | Value |
|---|---|
| Name | WebAppLearning |
| Environment | Dev |

#### Explanation

Tags help organize, identify, and manage Azure resources.

---

### Step 6: Review and Create

Verify:

| Setting | Value |
|---|---|
| Web App | datacenter-webapp |
| Region | Central US |
| Runtime | Python |
| OS | Linux |
| App Service Plan | datacenter-learn-python |
| SKU | Basic B1 |
| Application Insights | Disabled |

Click:

```text
Review + Create
```

After validation succeeds:

```text
Create
```

#### Explanation

Azure validates the configuration and deploys the Web App.

---

### Step 7: Verify Web App

After deployment completes:

```text
App Services
→ datacenter-webapp
```

Verify:

| Property | Expected Value |
|---|---|
| Status | Running |
| Runtime | Python |
| Operating System | Linux |
| Region | Central US |
| App Service Plan | datacenter-learn-python |

#### Explanation

The application must be running before submitting the task.

---

# Method 2: Azure CLI

### Step 1: Identify Resource Group

Run:

```bash
RG=$(az group list --query "[0].name" -o tsv)

echo $RG
```

#### Explanation

The Resource Group is required when creating Azure App Service resources.

---

### Step 2: Create App Service Plan

Run:

```bash
az appservice plan create \
  --resource-group $RG \
  --name datacenter-learn-python \
  --location centralus \
  --sku B1 \
  --is-linux
```

#### Explanation

This creates a Linux-based App Service Plan using the Basic B1 pricing tier.

---

### Step 3: Create Web App

Run:

```bash
az webapp create \
  --resource-group $RG \
  --plan datacenter-learn-python \
  --name datacenter-webapp \
  --runtime "PYTHON|3.11"
```

#### Explanation

This deploys a Python Web App using the previously created App Service Plan.

---

### Step 4: Add Required Tags

Run:

```bash
az tag update \
  --resource-id $(az webapp show \
  --resource-group $RG \
  --name datacenter-webapp \
  --query id -o tsv) \
  --operation merge \
  --tags Name=WebAppLearning Environment=Dev
```

#### Explanation

This applies the required tags to the Web App resource.

---

### Step 5: Ensure Web App is Running

Run:

```bash
az webapp start \
  --resource-group $RG \
  --name datacenter-webapp
```

#### Explanation

This guarantees the Web App is in the Running state.

---

# Verification Steps

### Step 6: Verify Web App

Run:

```bash
az webapp show \
  --resource-group $RG \
  --name datacenter-webapp \
  --query "{Name:name,State:state,Location:location}" \
  -o table
```

Expected output:

```text
Name               State    Location
-----------------  -------  ----------
datacenter-webapp  Running  Central US
```

---

### Step 7: Verify App Service Plan

Run:

```bash
az appservice plan show \
  --resource-group $RG \
  --name datacenter-learn-python \
  --query "{Plan:name,SKU:sku.name,Linux:reserved}" \
  -o table
```

Expected output:

```text
Plan                     SKU   Linux
-----------------------  ----  -----
datacenter-learn-python  B1    True
```

#### Explanation

A value of `True` confirms the App Service Plan is running on Linux.

---

### Step 8: Verify Tags

Run:

```bash
az webapp show \
  --resource-group $RG \
  --name datacenter-webapp \
  --query tags
```

Expected output:

```json
{
  "Environment": "Dev",
  "Name": "WebAppLearning"
}
```

---

### Step 9: Final Validation

Run:

```bash
az webapp show \
  --resource-group $RG \
  --name datacenter-webapp \
  --query "{Name:name,State:state,Location:location,Tags:tags}" \
  -o json
```

Expected output:

```json
{
  "Name": "datacenter-webapp",
  "State": "Running",
  "Location": "centralus",
  "Tags": {
    "Name": "WebAppLearning",
    "Environment": "Dev"
  }
}
```

---

## Best Practices

- Choose the appropriate App Service Plan based on workload requirements
- Use tags consistently for resource organization and cost tracking
- Disable unused monitoring services to reduce unnecessary resource usage
- Keep runtime versions updated for security and performance improvements
- Deploy applications through CI/CD pipelines for consistent releases
- Monitor application health and availability after deployment

## Key Learnings

- Azure App Service simplifies web application hosting and management
- App Service Plans determine compute resources and pricing
- Python applications can be deployed directly using the Code publishing model
- Linux App Services provide a flexible platform for modern web applications
- Tags improve governance, organization, and cost visibility
- Azure CLI enables automated deployment and management of Web Apps
