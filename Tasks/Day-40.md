# Day 40 – Managing Secrets with Azure Key Vault

---

## Task Overview

The Nautilus DevOps team is focusing on improving their data security by using Azure Key Vault. Your task is to create a Key Vault with an RSA key and manage the encryption and decryption of a pre-existing sensitive file using this key. 

Specific Requirements: 

**1) Create a Key Vault:** 

- Name the Key Vault `nautilus-20400`.
- Create the vault in the **East US** region.
- Use the **Standard** pricing tier.
- Set **Soft Delete retention** to **7 days**.
- Use the **Vault access policy** permission model.
- Configure an access policy that allows **Get, List, Encrypt, and Decrypt** permissions for the lab identity.

**2) Create an RSA Key:** 

- Create a key named `nautilus-key` within the Key Vault.
- Key type: **RSA**.
- RSA key size: **4096**.
- Leave all other settings as default.

**3) Encrypt the Sensitive Data:** 

- Use the key to encrypt the provided `SensitiveData.txt` file (located in `/root/`) on the `azure-client` host.
- Use the `RSA-OAEP` algorithm.
- Base64 encode the plaintext before encryption.
- Save the encrypted version as `EncryptedData.bin` in the `/root/` directory.

> Note: If you encounter a permissions error, retrieve the Service Principal ID using: **az account show --query user.name -o tsv** and grant the required Key Vault permissions.

**4) Verify Decryption:** 

- Decrypt `EncryptedData.bin`.
- Base64 decode the decrypted output.
- Save the result as `DecryptedData.txt` in `/root/`.
- Ensure the decrypted file matches the original `SensitiveData.txt`.

Ensure that the Key Vault and key are correctly configured. The validation script will test your configuration by decrypting the `EncryptedData.bin` file using the key you created.  

`Notes:` 

- Create the resources only in the **East US** region.
- Network restrictions or private endpoints are NOT required for this task.

---

## Step-by-Step Implementation

### Step 1: Get Resource Group and User Information

On **azure-client**:

```bash
RG=$(az group list --query "[0].name" -o tsv)

echo $RG

az account show --query user.name -o tsv
```

#### Explanation

Retrieves the resource group and current lab identity that will be granted Key Vault permissions.

---

### Step 2: Create Azure Key Vault

Navigate to:

```text
Azure Portal
→ Key Vaults
→ Create
```

Configure:

| Field          | Value            |
| -------------- | ---------------- |
| Resource Group | Existing RG      |
| Key Vault Name | `nautilus-20400` |
| Region         | `East US`        |
| Pricing Tier   | Standard         |

Click:

```text
Next
```

#### Explanation

Creates the Azure Key Vault that will store encryption keys.

---

### Step 3: Configure Access Model

Under **Access Configuration**:

| Setting          | Value               |
| ---------------- | ------------------- |
| Permission Model | Vault Access Policy |

Leave all other settings as default.

#### Explanation

The lab specifically requires the Vault Access Policy permission model instead of Azure RBAC.

---

### Step 4: Configure Soft Delete

Under **Recovery**:

| Setting               | Value  |
| --------------------- | ------ |
| Soft Delete Retention | 7 Days |

Click:

```text
Review + Create
→ Create
```

Wait until deployment succeeds.

#### Explanation

Configures deleted secrets and keys to remain recoverable for 7 days.

---

### Step 5: Configure Access Policy

Open:

```text
nautilus-20400
→ Access Policies
→ Create
```

Select Key Permissions:

```text
Get
List
Encrypt
Decrypt
```

Select the lab user identity and create the policy.

#### Explanation

Allows the lab identity to perform encryption and decryption operations using the Key Vault key.

---

### Step 6: Create RSA Key

Navigate to:

```text
nautilus-20400
→ Keys
→ Generate/Import
```

Configure:

| Field    | Value          |
| -------- | -------------- |
| Name     | `nautilus-key` |
| Key Type | RSA            |
| Key Size | 4096           |

Click:

```text
Create
```

#### Explanation

Creates a 4096-bit RSA key used for encryption and decryption.

---

### Step 7: Verify Key Creation

```bash
az keyvault key show \
  --vault-name nautilus-20400 \
  --name nautilus-key \
  -o table
```

#### Explanation

Confirms that the key exists and is available for cryptographic operations.

---

### Step 8: Base64 Encode Sensitive File

```bash
base64 -w 0 /root/SensitiveData.txt > /root/plain.txt
```

Verify:

```bash
cat /root/plain.txt
```

#### Explanation

The task requires Base64 encoding before encryption because RSA encryption expects a text payload.

---

### Step 9: Encrypt the File

```bash
PLAINTEXT=$(cat /root/plain.txt)

az keyvault key encrypt \
  --vault-name nautilus-20400 \
  --name nautilus-key \
  --algorithm RSA-OAEP \
  --value "$PLAINTEXT" \
  --query result \
  -o tsv > /root/EncryptedData.bin
```

Verify:

```bash
cat /root/EncryptedData.bin
```

#### Explanation

Encrypts the Base64 encoded file using the RSA-OAEP algorithm and saves the encrypted output.

---

### Step 10: Permission Fix (If Needed)

If encryption fails with:

```text
Forbidden
Insufficient Permissions
```

Run:

```bash
OBJECT_ID=$(az ad signed-in-user show --query id -o tsv)

az keyvault set-policy \
  --name nautilus-20400 \
  --object-id $OBJECT_ID \
  --key-permissions get list encrypt decrypt
```

Wait 30–60 seconds and retry.

#### Explanation

Grants the required permissions directly to the lab identity.

---

### Step 11: Decrypt the File

```bash
ENCRYPTED=$(cat /root/EncryptedData.bin)

az keyvault key decrypt \
  --vault-name nautilus-20400 \
  --name nautilus-key \
  --algorithm RSA-OAEP \
  --value "$ENCRYPTED" \
  --query result \
  -o tsv > /root/decrypted.b64
```

#### Explanation

Uses the same RSA key to decrypt the encrypted payload.

---

### Step 12: Base64 Decode the Output

```bash
base64 -d /root/decrypted.b64 > /root/DecryptedData.txt
```

#### Explanation

Converts the decrypted Base64 string back into the original file content.

---

### Step 13: Verify File Integrity

```bash
diff /root/SensitiveData.txt /root/DecryptedData.txt
```

Expected:

```text
(no output)
```

#### Explanation

No output means both files are identical and encryption/decryption succeeded.

---

## Azure CLI Only Method

### Create Key Vault

```bash
az keyvault create \
  --resource-group $RG \
  --name nautilus-20400 \
  --location eastus \
  --sku standard \
  --enable-rbac-authorization false \
  --retention-days 7
```

### Create RSA Key

```bash
az keyvault key create \
  --vault-name nautilus-20400 \
  --name nautilus-key \
  --kty RSA \
  --size 4096
```

### Configure Access Policy

```bash
OBJECT_ID=$(az ad signed-in-user show --query id -o tsv)

az keyvault set-policy \
  --name nautilus-20400 \
  --object-id $OBJECT_ID \
  --key-permissions get list encrypt decrypt
```

---

## Verification Commands

### Verify Key Vault

```bash
az keyvault show \
  --name nautilus-20400 \
  --query "{Name:name,Location:location,SKU:properties.sku.name}" \
  -o table
```

Expected:

```text
nautilus-20400
eastus
standard
```

---

### Verify Key

```bash
az keyvault key list \
  --vault-name nautilus-20400 \
  -o table
```

Expected:

```text
nautilus-key
```

---

### Verify Encrypted File

```bash
ls -lh /root/EncryptedData.bin
```

---

### Verify Decrypted File

```bash
ls -lh /root/DecryptedData.txt
```

---

### Verify Contents Match

```bash
diff /root/SensitiveData.txt /root/DecryptedData.txt
```

Expected:

```text
(no output)
```

---

## Best Practices

* Use RSA-OAEP instead of RSA1_5 for stronger security.
* Apply the principle of least privilege when creating access policies.
* Enable Soft Delete to protect against accidental deletion.
* Store sensitive files only after encryption.
* Verify decrypted data integrity using `diff`.

## Key Learnings

* Creating Azure Key Vaults.
* Managing Vault Access Policies.
* Creating RSA encryption keys.
* Encrypting and decrypting data using Azure Key Vault.
* Base64 encoding and decoding workflows.
* Validating encrypted data integrity.
