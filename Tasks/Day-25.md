# Day 25 – Expanding and Managing Disk Storage

---

## Task Overview  

The Nautilus DevOps team needs to expand the storage capacity of an existing virtual machine and add an additional data disk to support increased workloads. This task requires resizing the existing VM disk and mounting a new data disk to the VM.

As a member of the team, perform the following steps:

1) Expand the existing VM `xfusion-vm` disk from `32Gi` to `64Gi`.

2) Also create a new standard HDD data disk named `xfusion-disk` of `64Gi` and mount the disk to VM `xfusion-vm` at location `/mnt/xfusion-disk`.

---

## Step-by-Step Implementation

### Step 1: Open the Virtual Machine

In Azure Portal, search for:

| Search |
|---|
| Virtual Machines |

Open:

| VM Name |
|---|
| `xfusion-vm` |

#### Explanation  
This is the Virtual Machine whose storage needs to be expanded.

---

## Part 1: Expand Existing OS Disk

### Step 2: Open Disk Configuration

From the left menu navigate to:

- **Settings**
- **Disks**

Click the attached OS disk.

#### Explanation  
The OS disk currently has a size of 32 GiB and needs to be expanded.

---

### Step 3: Resize the OS Disk

Open:

- **Size + Performance**

Change the disk size from:

```text
32 GiB
```

to:

```text
64 GiB
```

Keep:

| Setting | Value |
|---|---|
| Storage Type | Standard HDD (LRS) |

Click:

- **Save**

Wait until the update completes successfully.

#### Explanation  
Increasing the disk size provides additional storage capacity for the operating system and applications.

---

## Part 2: Create a New Managed Disk

### Step 4: Create the Data Disk

Search for:

| Search |
|---|
| Disks |

Click:

- **+ Create**

Configure:

| Setting | Value |
|---|---|
| Name | `xfusion-disk` |
| Source Type | None |
| Size | 64 GiB |
| Storage Type | Standard HDD (LRS) |

Click:

- **Review + Create**
- **Create**

#### Explanation  
This creates an empty managed disk that will be attached as a data disk.

---

## Part 3: Attach the Disk to the VM

### Step 5: Attach Existing Disk

Return to:

- **Virtual Machines**
- **xfusion-vm**
- **Disks**

Under **Data Disks**, click:

- **Attach Existing Disk**

Select:

| Disk |
|---|
| `xfusion-disk` |

Click:

- **Save**

Wait until the attachment process completes.

#### Explanation  
The managed disk is now connected to the Virtual Machine and available to the operating system.

---

## Part 4: Configure the Disk Inside Linux

### Step 6: Connect to the VM

From the `azure-client` host:

```bash
ssh azureuser@<VM-PUBLIC-IP>
```

#### Explanation  
SSH access is required to configure the newly attached disk inside the operating system.

---

### Step 7: Verify Available Disks

Run:

```bash
lsblk
```

Example output:

```bash
sda    64G
sdc    64G
```

#### Explanation  
The newly attached disk should appear as an unpartitioned device.

---

### Step 8: Create a Partition

Assuming the new disk is:

```bash
/dev/sdc
```

Run:

```bash
sudo fdisk /dev/sdc
```

Inside fdisk enter:

```text
n
p
1
Enter
Enter
w
```

#### Explanation  
This creates a primary partition on the new disk.

---

### Step 9: Format the Partition

Run:

```bash
sudo mkfs.ext4 /dev/sdc1
```

#### Explanation  
Formatting prepares the partition with an ext4 filesystem so it can be mounted and used.

---

### Step 10: Create the Mount Directory

Run:

```bash
sudo mkdir -p /mnt/xfusion-disk
```

#### Explanation  
This directory will serve as the mount point for the new data disk.

---

### Step 11: Mount the Disk

Run:

```bash
sudo mount /dev/sdc1 /mnt/xfusion-disk
```

#### Explanation  
This mounts the filesystem and makes it available for use.

---

### Step 12: Verify the Mount

Run:

```bash
df -h
```

Expected output:

```bash
/dev/sdc1   64G   ...   /mnt/xfusion-disk
```

#### Explanation  
This confirms that the disk is mounted successfully.

---

### Step 13: Retrieve Disk UUID

Run:

```bash
sudo blkid /dev/sdc1
```

Example output:

```bash
UUID="1edb09ef-f674-4104-8a1c-76afe88b4636"
```

#### Explanation  
The UUID is required to create a persistent mount entry.

---

### Step 14: Configure Persistent Mount

Open:

```bash
sudo nano /etc/fstab
```

Add the following entry at the end of the file:

```bash
UUID=1edb09ef-f674-4104-8a1c-76afe88b4636 /mnt/xfusion-disk ext4 defaults,nofail 0 2
```

Save and exit.

#### Explanation  
This ensures the disk is automatically mounted after every reboot.

---

### Step 15: Validate fstab Configuration

Run:

```bash
sudo mount -a
```

Expected result:

```text
No output
```

#### Explanation  
No errors indicate that the `/etc/fstab` configuration is valid.

---

### Step 16: Final Verification

Verify mounted disk:

```bash
df -h | grep xfusion-disk
```

Verify disk layout:

```bash
lsblk
```

Expected output:

```bash
sda   64G
sdc   64G
└─sdc1   /mnt/xfusion-disk
```

#### Explanation  
This confirms that the OS disk was expanded and the new data disk is mounted correctly.

---

## Best Practices

- Always validate disk changes before modifying production systems  
- Use separate data disks for application and workload storage  
- Configure persistent mounts using UUIDs instead of device names  
- Verify filesystem integrity after disk provisioning  
- Monitor disk utilization regularly to prevent storage shortages  
- Follow standardized storage naming conventions across environments  

## Key Learnings

- Azure managed disks can be resized without recreating resources  
- Additional storage can be attached to running Virtual Machines  
- Linux requires partitioning, formatting, and mounting new disks manually  
- UUID-based mounts provide reliable persistent storage configuration  
- Azure storage expansion supports growing application workloads  
- Proper disk management improves system scalability and reliability  
