# Day 09 – Attach Network Interface Card (NIC) to Azure Virtual Machine

---

## Task Overview  

The nautilus DevOps team is migrating services to Azure. They are breaking down tasks to ensure better control and optimization. You are tasked with attaching an existing network interface (NIC) to a virtual machine (VM).

An existing VM named `nautilus-vm` and a network interface named `nautilus-nic` already exist in the `centralus` region.

- Attach the network interface `nautilus-nic` to the VM `nautilus-vm`.

- Ensure the NIC's status is `attached` before submitting the task.

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
| `nautilus-vm` |

#### Explanation  
This opens the configuration and management page of the target Virtual Machine.

### Step 4: Verify VM Initialization Status  

On the VM overview page, verify the VM status:

| Property | Expected Value |
|---|---|
| VM Status | Running |

#### Important  
If the VM status shows **Creating**, **Updating**, or **Starting**, wait until the status changes to **Running**.

#### Explanation  
The Virtual Machine must complete initialization before attaching additional resources.

### Step 5: Stop the Virtual Machine  

At the top of the VM page, click:

- **Stop**

Confirm if prompted.

Wait until the VM status changes to:

| Expected VM State |
|---|
| `Stopped (deallocated)` |

#### Explanation  
Azure allows attaching additional NICs only when the Virtual Machine is fully stopped and deallocated.

### Step 6: Open Networking Section  

From the left-side menu inside the Virtual Machine page, open:

- **Networking**

or

- **Network settings**

depending on Azure Portal version.

#### Explanation  
The Networking section is used to manage NICs, IP configurations, and network-related settings for the Virtual Machine.

### Step 7: Attach Existing Network Interface  

Click:

- **Attach network interface**

A side panel opens.

Under available NICs, select:

| Network Interface |
|---|
| `nautilus-nic` |

Then click:

- **OK**
- **Attach**

depending on Azure Portal version.

#### Explanation  
This attaches the existing Network Interface Card to the Virtual Machine.

### Step 8: Verify NIC Attachment  

Inside the Networking section, verify the attached NICs.

Expected configuration:

| Network Interface |
|---|
| Existing Primary NIC |
| `nautilus-nic` |

#### Explanation  
This confirms that the NIC was successfully attached to the Virtual Machine.

### Step 9: Start the Virtual Machine  

Go back to the VM overview page and click:

- **Start**

Wait until the VM status changes to:

| Expected VM State |
|---|
| Running |

#### Explanation  
The Virtual Machine must be restarted after NIC attachment to resume operations.

### Step 10: Final Verification  

Return to:

- **Networking**
- **Network settings**

Verify the following:

| Property | Expected Value |
|---|---|
| NIC Name | `nautilus-nic` |
| Status | Attached |

#### Explanation  
This confirms that the Network Interface Card is properly attached and associated with the Virtual Machine.

---

## Best Practices  

- Stop and deallocate the VM before attaching additional NICs  
- Use separate NICs for different network traffic requirements  
- Apply NSG rules to secure network interfaces  
- Use meaningful naming conventions for Azure networking resources  
- Monitor network performance and connectivity regularly  
- Keep VM networking configurations properly documented  

## Key Learnings  

- Azure NICs provide network connectivity for Virtual Machines  
- Additional NICs can be attached to Azure VMs for advanced networking setups  
- Virtual Machines must be deallocated before attaching secondary NICs  
- The Networking section is used to manage VM network interfaces  
- NICs can have independent IP configurations and security rules  
- Proper NIC management improves scalability and network organization  
