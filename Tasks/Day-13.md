# Day 13 – SSH into an Azure Virtual Machine

---

## Task Overview  

The Nautilus DevOps team is working on setting up secure SSH access for their virtual machines in Azure. One of the requirements is to add the SSH public key of the root user from the Azure client host (landing host) to the `nautilus-vm` Azure VM's `authorized_keys` file. This ensures secure and password-less SSH access to the VM.

**Task Details:**

**1) VM Details:**

- The VM is named `nautilus-vm` and is running in the `westus` region. The default SSH user is `azureuser` — use this user to connect to the VM.

- You need to add the root user's SSH public key from the Azure client host to the `authorized_keys` file of the VM's root user.

- The SSH public key of the root user on the Azure client host is located at `/root/.ssh/id_rsa.pub`.
  
**2) Public Key Addition:**

- Copy the public key located at `/root/.ssh/id_rsa.pub` on the Azure client host to the `authorized_keys` file of the root user on `nautilus-vm`.

- Ensure that the proper permissions for the `.ssh` folder and `authorized_keys` file are set on the VM.
  
**3) Verification:**

- After adding the public key, make sure that you are able to SSH into the `nautilus-vm` VM as the `root` user from the Azure client host without needing a password.

**Important Notes:**

- Ensure that the VM is up and running before attempting to SSH.

- You may need to adjust the firewall or security group rules for the VM to allow SSH access. 

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
This confirms that you are currently working on the Azure client host where the root SSH public key exists.

### Step 2: Get Virtual Machine Public IP Address  

Run:

```bash
az vm list-ip-addresses --name nautilus-vm --output table
```

Example output:

```bash
20.xx.xx.xx
```

#### Explanation  
This command retrieves the public IP address assigned to the Azure Virtual Machine.

### Step 3: Connect to VM Using SSH  

Run:

```bash
ssh azureuser@<PUBLIC-IP>
```

Example:

```bash
ssh azureuser@20.xx.xx.xx
```

If prompted, type:

```bash
yes
```

#### Explanation  
This establishes an SSH connection to the Azure Virtual Machine using the default Azure user.

### Step 4: Switch to Root User  

Inside the VM, run:

```bash
sudo -i
```

Expected prompt:

```bash
root@nautilus-vm:~#
```

#### Explanation  
This switches the current session to the root user for system-level configuration.

### Step 5: Create Root SSH Directory  

Run:

```bash
mkdir -p /root/.ssh
```

#### Explanation  
This command creates the `.ssh` directory for the root user if it does not already exist.

### Step 6: Set SSH Directory Permissions  

Run:

```bash
chmod 700 /root/.ssh
```

#### Explanation  
This sets secure permissions on the `.ssh` directory so only the root user can access it.

### Step 7: Open New Terminal Session  

Open a new terminal tab without closing the existing SSH session.

Verify again:

```bash
hostname
```

Expected output:

```bash
azure-client
```

#### Explanation  
A second terminal session is required to copy the SSH public key from the Azure client host.

### Step 8: View Root Public Key  

Run:

```bash
sudo cat /root/.ssh/id_rsa.pub
```

Example output:

```bash
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQ...
```

Copy the complete SSH public key.

#### Explanation  
This displays the root user's SSH public key that will be added to the Virtual Machine.

### Step 9: Return to VM Root Session  

Go back to the first terminal where you are connected to the VM as the root user.

#### Explanation  
The SSH public key will now be added to the root user's authorized keys file.

### Step 10: Add Public Key to authorized_keys  

Run:

```bash
nano /root/.ssh/authorized_keys
```

Paste the copied SSH public key into the file.

Example:

```bash
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQ...
```

#### Explanation  
The `authorized_keys` file stores all SSH public keys allowed for passwordless authentication.

### Step 11: Save the File  

Inside nano editor:

- Press `CTRL + O`
- Press `Enter`
- Press `CTRL + X`

#### Explanation  
This saves the `authorized_keys` file and exits the editor.

### Step 12: Set authorized_keys Permissions  

Run:

```bash
chmod 600 /root/.ssh/authorized_keys
```

#### Explanation  
This sets secure permissions on the `authorized_keys` file. SSH rejects insecure file permissions.

### Step 13: Enable Root SSH Login  

Open SSH configuration:

```bash
nano /etc/ssh/sshd_config
```

Find either:

```bash
#PermitRootLogin prohibit-password
```

or:

```bash
PermitRootLogin no
```

Change it to:

```bash
PermitRootLogin yes
```

#### Explanation  
This enables direct SSH login for the root user.

### Step 14: Restart SSH Service  

Run:

```bash
systemctl restart ssh
```

#### Explanation  
Restarting the SSH service applies the updated SSH configuration.

### Step 15: Exit VM Session  

Run:

```bash
exit
```

Then again:

```bash
exit
```

You should now return to:

```bash
azure-client
```

#### Explanation  
This exits both the root session and the SSH session connected to the Virtual Machine.

### Step 16: Verify Passwordless Root SSH Login  

Run:

```bash
ssh root@<PUBLIC-IP>
```

Example:

```bash
ssh root@20.xx.xx.xx
```

Expected output:

```bash
root@nautilus-vm:~#
```

#### Explanation  
This confirms that passwordless SSH access for the root user is working successfully.

---

## Best Practices  

- Use SSH key authentication instead of passwords for secure access  
- Restrict SSH access only to trusted users and IP addresses  
- Maintain proper permissions on `.ssh` directories and files  
- Disable password authentication for production servers whenever possible  
- Use NSG rules to restrict unnecessary inbound SSH access  
- Regularly rotate SSH keys for enhanced security  

## Key Learnings  

- SSH public keys enable secure passwordless authentication  
- The `authorized_keys` file controls SSH access permissions  
- Proper file permissions are critical for SSH authentication to work  
- Root SSH access can be controlled through `sshd_config` settings  
- Azure Virtual Machines can be accessed securely using SSH  
- Network Security Groups (NSGs) control inbound SSH connectivity  
