# Day 11 – Change Azure Virtual Machine Size Using Console

---

## Task Overview  

The Nautilus DevOps team is migrating a portion of its infrastructure to Azure. During the migration, they have created several virtual machines (VMs) in different regions. The team identified one VM that is experiencing increased workload demands and requires additional compute resources to maintain optimal performance. 

- Change the VM size from `Standard_B1s` to `Standard_B2s` for the virtual machine named `xfusion-vm`.

- Ensure the VM is in the `running` state after the size change is complete.

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

### Step 4: Verify Current VM Status  

On the VM overview page, verify the following:

| Property | Expected Value |
|---|---|
| VM Status | Running |

#### Important  
If the VM is still initializing, wait until the status changes to **Running**.

#### Explanation  
The Virtual Machine should be fully initialized before performing resize operations.

### Step 5: Open VM Size Settings  

From the left-side menu inside the Virtual Machine page, open:

- **Size**

under the **Settings** section.

#### Explanation  
The Size section is used to change the compute configuration and hardware profile of the Virtual Machine.

### Step 6: Select New VM Size  

Azure displays all available VM sizes.

Search for:

| VM Size |
|---|
| `Standard_B2s` |

Locate the row containing `Standard_B2s` and click:

- **Resize**

#### Explanation  
This updates the Virtual Machine configuration from Standard_B1s to Standard_B2s to provide additional CPU and memory resources.

### Step 7: Wait for Resize Operation  

Wait for the resize process to complete successfully.

During this process:

- The Virtual Machine may restart automatically
- The operation may take several minutes

#### Explanation  
Azure applies the new hardware configuration and updates the Virtual Machine resources.

### Step 8: Verify Updated VM Size  

Go back to:

- **Overview**

Verify the following:

| Property | Expected Value |
|---|---|
| VM Size | `Standard_B2s` |

#### Explanation  
This confirms that the Virtual Machine size was updated successfully.

### Step 9: Verify VM Running State  

Also verify:

| Property | Expected Value |
|---|---|
| VM Status | Running |

If the VM is stopped, click:

- **Start**

Wait until the VM status changes to:

| Expected VM State |
|---|
| Running |

#### Explanation  
The Virtual Machine must remain in a running state after the resize operation is completed.

### Step 10: Final Verification  

Verify the following:

| Verification | Expected Value |
|---|---|
| VM Size | `Standard_B2s` |
| VM State | Running |

#### Explanation  
This confirms that the resize operation completed successfully and the Virtual Machine is operational.

---

## Best Practices  

- Choose VM sizes according to workload and performance requirements  
- Monitor CPU and memory utilization before resizing Virtual Machines  
- Resize VMs during maintenance windows for production environments  
- Stop unnecessary services before performing infrastructure changes  
- Use cost-optimized VM sizes to balance performance and budget  
- Verify VM health and connectivity after resizing operations  

## Key Learnings  

- Azure Virtual Machines can be resized without recreating the VM  
- VM resizing changes the compute resources allocated to the VM  
- The Size section is used to modify Azure VM hardware profiles  
- Some resize operations may automatically restart the Virtual Machine  
- VM status should always be verified after infrastructure changes  
- Proper VM sizing improves application performance and scalability  
