# Day 01 – Create SSH Key Pair for Azure Virtual Machine

---

## Task Overview  

The Nautilus DevOps team is strategizing the migration of a portion of their infrastructure to the Azure cloud. Recognizing the scale of this undertaking, they have opted to approach the migration in incremental steps rather than as a single massive transition. To achieve this, they have segmented large tasks into smaller, more manageable units. This granular approach enables the team to execute the migration in gradual phases, ensuring smoother implementation and minimizing disruption to ongoing operations. By breaking down the migration into smaller tasks, the Nautilus DevOps team can systematically progress through each stage, allowing for better control, risk mitigation, and optimization of resources throughout the migration process. 

For this task, create an SSH key pair with the following requirements: 

- The name of the SSH key pair should be `datacenter-kp`.

- The key pair `type` must be `rsa`.

---

## Step-by-Step Implementation  

### Step 1: Login to Azure Portal  

Open the Azure Portal:

https://portal.azure.com

Login using your Azure account credentials.

#### Explanation  
The Azure Portal is Microsoft Azure’s web-based management interface used to create and manage cloud resources.

### Step 2: Navigate to SSH Keys  

Use the top search bar and search for:

| Search |
|---|
| SSH keys |

Open the **SSH keys** service from the results.

#### Explanation  
Azure SSH Keys service is used to create and manage SSH public keys for secure Linux Virtual Machine authentication.

### Step 3: Start Creating SSH Key  

Click the **+ Create** button located at the top-left corner.

#### Explanation  
This starts the process of creating a new SSH key resource in Azure.

---

### Step 4: Configure Project Details  

Under **Project Details**, configure the following settings:

| Setting | Value |
|---|---|
| Subscription | Default Azure Subscription |
| Resource Group | `datacenter-rg` |

#### Explanation  
A Resource Group is a logical container used to organize and manage related Azure resources together.

### Step 5: Configure SSH Key Details  

Under **Instance Details**, configure the following:

| Setting | Value |
|---|---|
| Region | East US |
| Name | `datacenter-kp` |
| SSH Public Key Source | Generate new key pair |
| Key Pair Type | RSA |

#### Explanation  
Azure automatically generates both public and private SSH keys. RSA is a widely used encryption algorithm for secure SSH authentication.

### Step 6: Review and Create  

Click:

- **Review + create**
- **Create**

after validation passes successfully.

#### Explanation  
Azure validates all configuration settings before provisioning the SSH key resource.

### Step 7: Download the Private Key  

A popup window will appear.

Click:

- **Download private key and create resource**

Save the downloaded file securely:

| File |
|---|
| `datacenter-kp.pem` |

#### Important  
The private key file can only be downloaded once.

#### Explanation  
The `.pem` file is required to securely connect to Linux Virtual Machines.

### Step 8: Verify SSH Key  

Go back to the **SSH keys** service and verify the following details:

| Property | Expected Value |
|---|---|
| Name | `datacenter-kp` |
| Key Type | `RSA` |

#### Explanation  
This confirms that the SSH key was created successfully with the required configuration.

---

# Method 2: Using Azure CLI  

### Step 1: Login to Azure  

Run the following command:

```bash
az login
```

Follow the authentication instructions displayed on the screen.

#### Explanation  
This command authenticates Azure CLI with your Azure account.

### Step 2: Check Available Resource Groups  

Run:

```bash
az group list --output table
```

#### Explanation  
This command lists all available Resource Groups in your Azure subscription.

### Step 3: Set Resource Group Variable  

Run:

```bash
RG_NAME="datacenter-rg"
```

#### Explanation  
This variable stores the Resource Group name for easier command execution.

### Step 4: Create SSH Key Pair  

Run the following command:

```bash
az sshkey create \
  --name "datacenter-kp" \
  --resource-group $RG_NAME \
  --location "eastus"
```

#### Explanation  
This command creates an SSH key resource named `datacenter-kp` inside the specified Resource Group. Azure creates RSA keys by default.

### Step 5: Verify SSH Key  

Run:

```bash
az sshkey list --resource-group $RG_NAME --output table
```

#### Explanation  
This command lists all SSH key resources available inside the specified Resource Group. Verify that `datacenter-kp` exists successfully.

### Step 6: Secure the Private Key  

For Linux or macOS systems, run:

```bash
chmod 400 datacenter-kp.pem
```

#### Explanation  
This command restricts file permissions so only the owner can access the private key. SSH clients reject keys with insecure permissions.

---

# Best Practices

- Store private keys securely
- Never upload `.pem` files to GitHub
- Use SSH keys instead of passwords
- Rotate keys periodically
- Apply least privilege access policies

## Key Learnings

* Azure supports centralized SSH key management
* RSA keys are widely used for secure authentication
* SSH keys improve infrastructure security
* Azure CLI helps automate cloud operations
* Proper key management is critical in production environments
