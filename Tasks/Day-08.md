# Day 08 – Attach Managed Disk to Azure Virtual Machine

---

## Task Overview  

The Nautilus devops team is migrating services to Azure. They are breaking down tasks to ensure better control and optimization. You are tasked with attaching an existing data disk to a virtual machine (VM).

An existing VM named `devops-vm` and a managed disk named `devops-disk` already exist in the `southcentralus` region.

- Attach the disk `devops-disk` to the VM `devops-vm` as a data disk.

- Ensure the disk is attached to the VM `devops-vm`.

Make sure that the virtual machine initialization has been completed before submitting this task.

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
The Virtual Machines service is used to create, manage, and configure Azure Virtual Machines.

### Step 3: Open the Existing Virtual Machine  

From the Virtual Machines list, open:

| Virtual Machine |
|---|
| `devops-vm` |

#### Explanation  
This opens the configuration and management page of the target Virtual Machine.

### Step 4: Verify VM Initialization Status  

On the VM overview page, verify the VM status:

| Property | Expected Value |
|---|---|
| Status | Running |

#### Important  
If the VM status shows **Creating**, **Updating**, or **Starting**, wait until the status changes to **Running**.

#### Explanation  
The Virtual Machine must complete initialization before attaching additional resources.

### Step 5: Open Disks Section  

From the left-side menu inside the Virtual Machine page, open:

- **Disks**

#### Explanation  
The Disks section is used to manage OS disks and additional data disks attached to the Virtual Machine.

### Step 6: Attach Existing Managed Disk  

Under the **Data disks** section, click:

- **Attach existing disks**

#### Explanation  
This option allows attaching an already existing managed disk to the Virtual Machine.

### Step 7: Select Managed Disk  

Under **Disk name**, select:

| Disk Name |
|---|
| `devops-disk` |

#### Explanation  
This selects the existing managed disk that will be attached to the Virtual Machine as a data disk.

### Step 8: Save Configuration  

Click:

- **Save**

Wait for the operation to complete successfully.

#### Explanation  
Azure attaches the selected managed disk to the Virtual Machine and updates the VM configuration.

### Step 9: Verify Disk Attachment  

Inside:

- **VM → Disks**

Verify the following details under **Data disks**:

| Property | Expected Value |
|---|---|
| Disk Name | `devops-disk` |
| Status | Attached |

#### Explanation  
This confirms that the managed disk was successfully attached to the Virtual Machine.

---

## Best Practices  

- Verify VM status before attaching or modifying disks  
- Use managed disks for better scalability and reliability  
- Attach separate data disks instead of storing application data on OS disks  
- Use meaningful naming conventions for Azure resources  
- Monitor disk performance and storage utilization regularly  
- Keep backup and recovery strategies for important data disks  

## Key Learnings  

- Managed disks provide persistent and scalable storage for Azure VMs  
- Data disks can be attached to existing Virtual Machines without recreating them  
- Azure allows dynamic storage expansion using managed disks  
- The Disks section is used to manage VM storage resources  
- Linux block devices can be verified using the `lsblk` command  
- Proper disk management improves infrastructure scalability and maintainability  
