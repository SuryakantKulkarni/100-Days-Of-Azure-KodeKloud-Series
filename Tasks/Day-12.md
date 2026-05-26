# Day 12 – Add and Manage Tags for Azure Virtual Machines

---

## Task Overview  

The Nautilus DevOps team is migrating a portion of their infrastructure to Azure. During the migration, they have created several virtual machines (VMs) in different regions. The team has identified one VM that is not tagged properly so they decided to tag it as needed.

Add the tag `Environment=dev` to the virtual machine named `xfusion-vm`.

---

## Step-by-Step Implementation  

### Step 1: Login to Azure Portal  

Open the Azure Portal:

https://portal.azure.com

Login using your Azure account credentials.

#### Explanation  
The Azure Portal is Microsoft Azure’s web-based management interface used to create and manage cloud resources.

### Step 2: Navigate to Virtual Machines  

Use the top search bar and search for:

| Search |
|---|
| Virtual machines |

Open the **Virtual machines** service from the results.

#### Explanation  
The Virtual Machines service is used to create, manage, monitor, and configure Azure Virtual Machines.

### Step 3: Open the Existing Virtual Machine  

From the Virtual Machines list, open:

| Virtual Machine |
|---|
| `xfusion-vm` |

#### Explanation  
This opens the configuration and management page of the target Virtual Machine.

### Step 4: Open Tags Section  

From the left-side menu inside the Virtual Machine page, open:

- **Tags**

under the **Settings** section.

#### Explanation  
The Tags section is used to add metadata labels to Azure resources for better organization and management.

### Step 5: Add the Required Tag  

Under the tag configuration fields, enter the following values:

| Field | Value |
|---|---|
| Name | `Environment` |
| Value | `dev` |

#### Important  
- Use exact spelling for the tag key  
- `Environment` must start with a capital `E`  
- `dev` must remain lowercase  

#### Explanation  
Azure tags are key-value pairs used for organizing, filtering, billing, automation, and environment identification.

### Step 6: Save the Tag Configuration  

Click:

- **Apply**

or

- **Save**

depending on Azure Portal version.

Wait for the update operation to complete successfully.

#### Explanation  
Azure updates the Virtual Machine metadata and applies the new tag configuration.

### Step 7: Verify Tag Assignment  

Verify the following tag details:

| Tag Name | Tag Value |
|---|---|
| `Environment` | `dev` |

#### Explanation  
This confirms that the tag was successfully applied to the Virtual Machine.

---

## Best Practices  

- Use standardized tagging conventions across all Azure resources  
- Apply tags consistently for environments such as Dev, Test, and Production  
- Use tags for cost management and billing analysis  
- Avoid unnecessary or duplicate tag values  
- Use meaningful tag names for easier resource identification  
- Regularly audit and maintain Azure resource tags  

## Key Learnings  

- Azure Tags are key-value pairs used for resource organization  
- Tags help simplify resource management and cost tracking  
- Tags can be applied directly from the Azure Portal  
- Consistent tagging improves automation and governance  
- Azure resources can be filtered and grouped using tags  
- Proper tagging strategies improve cloud infrastructure management  
