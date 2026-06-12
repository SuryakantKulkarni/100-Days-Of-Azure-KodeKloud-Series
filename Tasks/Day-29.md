# Day 29 – Working with Azure Container Registry (ACR)

---

## Task Overview

The Nautilus DevOps team has been tasked with setting up a containerized application. They need to create a `Azure Container Registry (ACR)` to store their Docker images. Once the repository is created, they will build a Docker image from a Dockerfile located on the `azure-client` host and push this image to the ACR repository. This process is essential for maintaining and deploying containerized applications in a streamlined manner.

1) Create a ACR repository named `datacenteracr17663` under `East US`.

2) Pricing plan must be `Basic`.

3) Dockerfile already exists under `/root/pyapp` directory on `azure-client` host.

4) Build a Docker image using this Dockerfile and push the same to the newly created ACR repo. The image tag must be `latest` i.e `datacenteracr17663:latest`.

---

# Step-by-Step Implementation

### Step 1: Verify Azure Login

Login to Azure if not already authenticated.

```bash
az login
```

Verify account details:

```bash
az account show --output table
```

#### Explanation

This ensures that Azure CLI is authenticated and ready to create resources.

---

### Step 2: Identify the Resource Group

List available resource groups:

```bash
az group list --output table
```

Store the resource group name:

```bash
RG=$(az group list --query "[0].name" -o tsv)

echo $RG
```

#### Explanation

The Resource Group is required when creating Azure resources.

---

### Step 3: Create Azure Container Registry

Create the Container Registry:

```bash
az acr create \
  --resource-group $RG \
  --name datacenteracr17663 \
  --sku Basic \
  --location eastus
```

#### Explanation

This creates an Azure Container Registry named `datacenteracr17663` in the East US region using the Basic pricing tier.

---

### Step 4: Verify ACR Creation

Run:

```bash
az acr show \
  --name datacenteracr17663 \
  --output table
```

Expected output:

```text
Name                Location    Sku
------------------  ----------  -----
datacenteracr17663  eastus      Basic
```

#### Explanation

This confirms that the Container Registry was created successfully.

---

### Step 5: Enable Admin Access

Enable the admin user for registry authentication.

```bash
az acr update \
  --name datacenteracr17663 \
  --admin-enabled true
```

Verify:

```bash
az acr show \
  --name datacenteracr17663 \
  --query adminUserEnabled
```

Expected output:

```text
true
```

#### Explanation

Admin access allows Docker to authenticate and push images to the registry.

---

### Step 6: Login to Azure Container Registry

Authenticate Docker with ACR.

```bash
az acr login --name datacenteracr17663
```

Expected output:

```text
Login Succeeded
```

#### Explanation

This establishes authentication between Docker and the Azure Container Registry.

---

### Step 7: Navigate to the Dockerfile Directory

Move to the application directory:

```bash
cd /root/pyapp
```

Verify the Dockerfile exists:

```bash
ls
```

Expected output:

```text
Dockerfile
```

#### Explanation

The Docker image will be built using this Dockerfile.

---

### Step 8: Retrieve ACR Login Server

Run:

```bash
az acr show \
  --name datacenteracr17663 \
  --query loginServer \
  -o tsv
```

Expected output:

```text
datacenteracr17663.azurecr.io
```

#### Explanation

The login server is required when tagging Docker images for ACR.

---

### Step 9: Build the Docker Image

Build the image using the Dockerfile.

```bash
docker build \
  -t datacenteracr17663.azurecr.io/datacenteracr17663:latest .
```

Expected output:

```text
Successfully built
Successfully tagged datacenteracr17663.azurecr.io/datacenteracr17663:latest
```

#### Explanation

This creates the Docker image and tags it appropriately for Azure Container Registry.

---

### Step 10: Verify Local Image

List Docker images:

```bash
docker images
```

Expected output:

```text
datacenteracr17663.azurecr.io/datacenteracr17663   latest
```

#### Explanation

This confirms that the image was built successfully.

---

### Step 11: Push the Image to ACR

Push the image:

```bash
docker push datacenteracr17663.azurecr.io/datacenteracr17663:latest
```

Expected output:

```text
latest: digest: sha256:xxxxxxxxxxxxxxxxxxxx
```

#### Explanation

This uploads the Docker image to Azure Container Registry.

---

### Step 12: Verify Repository

List repositories:

```bash
az acr repository list \
  --name datacenteracr17663 \
  --output table
```

Expected output:

```text
datacenteracr17663
```

#### Explanation

This confirms that the repository exists inside the registry.

---

### Step 13: Verify Image Tag

List repository tags:

```bash
az acr repository show-tags \
  --name datacenteracr17663 \
  --repository datacenteracr17663 \
  --output table
```

Expected output:

```text
latest
```

#### Explanation

This confirms that the image was pushed successfully with the required tag.

---

## Final Verification Commands

**Verify ACR:**

```bash
az acr show \
  --name datacenteracr17663 \
  --output table
```

**Verify repository:**

```bash
az acr repository list \
  --name datacenteracr17663 \
  --output table
```

**Verify tags:**

```bash
az acr repository show-tags \
  --name datacenteracr17663 \
  --repository datacenteracr17663 \
  --output table
```

---

## Best Practices

- Use meaningful image tags instead of relying only on `latest`
- Store container images in a centralized registry for consistency
- Enable registry authentication before pushing images
- Scan images regularly for vulnerabilities and outdated packages
- Follow a structured repository naming convention
- Verify image uploads immediately after pushing to the registry

## Key Learnings

- Azure Container Registry provides private container image storage
- Docker images must be tagged with the ACR login server before pushing
- Azure CLI simplifies container registry management and automation
- ACR supports secure image distribution across Azure services
- Container registries are essential components of CI/CD pipelines
- Proper image versioning improves deployment reliability and rollback capabilities
