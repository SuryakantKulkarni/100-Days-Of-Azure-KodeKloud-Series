# Day 10 – Attach Public IP to Azure Virtual Machine

---

## Task Overview  

The Nautilus DevOps team has already set up a virtual machine and allocated a public IP address. The final task is to attach this public IP to the VM's network interface card (NIC).

An existing VM named `datacenter-vm-pip` and a public IP address named `datacenter-pip` already exist.

- Attach the public IP `datacenter-pip` to the network interface of the VM `datacenter-vm-pip`.
  
Make sure the VM is properly assigned the public IP.

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
| `datacenter-vm-pip` |

#### Explanation  
This opens the configuration and management page of the target Virtual Machine.

### Step 4: Verify VM Status  

On the VM overview page, verify the VM status:

| Property | Expected Value |
|---|---|
| Status | Running |

#### Important  
If the VM is still initializing, wait until the status changes to **Running**.

#### Explanation  
The Virtual Machine should be fully initialized before modifying networking configurations.

### Step 5: Open Networking Section  

From the left-side menu inside the Virtual Machine page, open:

- **Networking**

or

- **Network settings**

depending on Azure Portal version.

#### Explanation  
The Networking section is used to manage NICs, IP configurations, NSGs, and Public IP assignments.

### Step 6: Open Network Interface Card (NIC)  

Under the Networking section, click the attached NIC name.

Example:

| NIC Name |
|---|
| `datacenter-vm-pipVMNic` |

#### Explanation  
This opens the configuration page of the Network Interface Card connected to the Virtual Machine.

### Step 7: Open IP Configurations  

Inside the NIC page, open:

- **IP configurations**

from the left-side menu.

#### Explanation  
The IP configurations section is used to manage private and public IP assignments for the NIC.

### Step 8: Open Existing IP Configuration  

Under IP configurations, click the existing configuration.

Example:

| IP Configuration |
|---|
| `ipconfig1` |

#### Explanation  
This opens the IP configuration settings associated with the NIC.

### Step 9: Attach Public IP Address  

Under the **Public IP address** dropdown, select:

| Public IP |
|---|
| `datacenter-pip` |

#### Explanation  
This associates the existing Public IP Address with the Virtual Machine’s Network Interface Card.

### Step 10: Save Configuration  

Click:

- **Save**

Wait for the update process to complete successfully.

#### Explanation  
Azure updates the NIC configuration and attaches the selected Public IP Address.

### Step 11: Verify Public IP Attachment  

After the configuration is saved, verify the following:

| Property | Expected Value |
|---|---|
| Public IP Address | `datacenter-pip` |

#### Explanation  
This confirms that the Public IP Address has been successfully attached to the NIC.

### Step 12: Verify From VM Overview Page  

Go back to:

| Virtual Machine |
|---|
| `datacenter-vm-pip` |

Verify the following on the Overview page:

| Property | Expected Value |
|---|---|
| Public IP Address | Visible |

#### Explanation  
This confirms that the Virtual Machine is properly assigned the Public IP Address and can communicate externally.

---

## Best Practices  

- Use Static Public IPs for production Virtual Machines  
- Attach Public IPs only when external connectivity is required  
- Restrict inbound access using NSG rules for better security  
- Use meaningful naming conventions for networking resources  
- Monitor Public IP usage and associated VM connectivity regularly  
- Avoid exposing unnecessary services directly to the internet  

## Key Learnings  

- Public IP Addresses enable Azure Virtual Machines to communicate over the internet  
- Public IPs are associated through Network Interface Card configurations  
- NIC IP configurations manage both private and public IP assignments  
- Azure allows attaching existing Public IP resources to Virtual Machines  
- Proper Public IP management improves connectivity and security  
- VM networking components can be managed centrally through Azure Portal  
