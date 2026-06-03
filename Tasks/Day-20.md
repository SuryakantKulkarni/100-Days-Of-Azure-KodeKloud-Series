# Day 20 – Deploy Azure Resources Using ARM Template

---

## Task Overview  

You are tasked with modifying an ARM template for deploying a virtual network. The current template is located in the `/root/arm-templates` directory under the filename `vnet-deployment-template.json`. You need to make the following changes to the template:

- Change the name and `displayName` tag of the virtual network to `arm-vnet-devops`.

- Update the `addressPrefixes` to `192.168.0.0/16`.

- Add one more tag named `Environment` with value `KKE-devops`.

After making these changes, you need to deploy the ARM template using the Azure CLI.

Use the following command to find out the resource group to use: 
```bash
az group list --query '[].name' --output table | grep 'kml'
```
---

## Step-by-Step Implementation  

### Step 1: Verify Current Host  

Run:

```bash
hostname
```

Expected output:

```bash
azure-client
```

#### Explanation  
This confirms that you are connected to the Azure client machine where Azure CLI and ARM templates are available.

---

### Step 2: Identify the Resource Group  

Run:

```bash
az group list --query '[].name' --output table | grep 'kml'
```

Example output:

```bash
kml-rg-12345
```

#### Explanation  
This command retrieves the Resource Group assigned to the lab environment.

---

### Step 3: Navigate to ARM Template Directory  

Run:

```bash
cd /root/arm-templates
```

Verify the template exists:

```bash
ls
```

Expected output:

```bash
vnet-deployment-template.json
```

#### Explanation  
The ARM template file that needs modification is stored in this directory.

---

### Step 4: Review Existing Template  

Run:

```bash
cat vnet-deployment-template.json
```

#### Explanation  
Reviewing the template helps identify the sections that require modification.

---

### Step 5: Open the ARM Template for Editing  

Run:

```bash
vi vnet-deployment-template.json
```

or

```bash
nano vnet-deployment-template.json
```

#### Explanation  
This opens the ARM template for editing.

---

### Step 6: Update Virtual Network Name  

Locate the Virtual Network resource definition and modify:

```json
"name": "arm-vnet-devops"
```

#### Explanation  
This updates the Virtual Network name as required by the task.

---

### Step 7: Update displayName Tag  

Locate the tags section and update:

```json
"displayName": "arm-vnet-devops"
```

#### Explanation  
This updates the displayName tag to match the required Virtual Network name.

---

### Step 8: Update Address Prefix  

Locate the address space configuration:

```json
"addressPrefixes": [
    "192.168.0.0/16"
]
```

#### Explanation  
This defines the IPv4 address range for the Virtual Network.

---

### Step 9: Add Environment Tag  

Inside the tags section, add:

```json
"Environment": "KKE-devops"
```

Final tags section should look similar to:

```json
"tags": {
    "displayName": "arm-vnet-devops",
    "Environment": "KKE-devops"
}
```

#### Explanation  
This adds an additional metadata tag required for the deployment.

---

### Step 10: Save the Template  

If using Nano:

```text
CTRL + O
Enter
CTRL + X
```

#### Explanation  
This saves the modifications and exits the editor.

---

### Step 11: Validate Template Syntax  

Run:

```bash
cat vnet-deployment-template.json | python -m json.tool > /dev/null
```

#### Explanation  
This validates the JSON syntax before deployment. No output indicates valid JSON formatting.

---

### Step 12: Deploy the ARM Template  

Replace `<RESOURCE_GROUP>` with the Resource Group identified earlier.

Run:

```bash
az deployment group create \
  --resource-group <RESOURCE_GROUP> \
  --template-file /root/arm-templates/vnet-deployment-template.json
```

Example:

```bash
az deployment group create \
  --resource-group kml-rg-12345 \
  --template-file /root/arm-templates/vnet-deployment-template.json
```

#### Explanation  
This command deploys the ARM template and provisions the Virtual Network in Azure.

---

### Step 13: Verify Deployment Status  

Run:

```bash
az deployment group list \
  --resource-group <RESOURCE_GROUP> \
  --output table
```

Expected output:

```bash
Succeeded
```

#### Explanation  
This confirms that the ARM template deployment completed successfully.

---

### Step 14: Verify Virtual Network Deployment  

Run:

```bash
az network vnet list \
  --resource-group <RESOURCE_GROUP> \
  --output table
```

Expected output:

```bash
arm-vnet-devops
```

#### Explanation  
This verifies that the Virtual Network was created using the updated ARM template.

---

### Step 15: Verify Tags and Address Space  

Run:

```bash
az network vnet show \
  --resource-group <RESOURCE_GROUP> \
  --name arm-vnet-devops
```

Verify:

| Property | Expected Value |
|---|---|
| Name | arm-vnet-devops |
| displayName | arm-vnet-devops |
| Environment | KKE-devops |
| Address Prefix | 192.168.0.0/16 |

#### Explanation  
This confirms that all required template modifications were deployed successfully.

---

## Best Practices  

- Validate ARM templates before deployment to avoid runtime failures  
- Use meaningful resource names and tags for easier management  
- Store ARM templates in version control repositories  
- Follow infrastructure-as-code practices for repeatable deployments  
- Test template changes in non-production environments first  
- Use tags consistently for governance, billing, and automation  

## Key Learnings  

- ARM Templates enable Infrastructure as Code (IaC) in Azure  
- Azure CLI can deploy ARM templates directly from local files  
- Virtual Network configurations can be managed through templates  
- Tags help organize and manage Azure resources effectively  
- JSON validation helps prevent deployment errors  
- Infrastructure automation improves consistency and operational efficiency  
