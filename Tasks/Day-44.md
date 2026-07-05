# Day 44 – Integrating Azure Event Hub with Virtual Machines

---

## Task Overview

The Nautilus DevOps team wants to integrate an Azure Virtual Machine with Azure Event Hubs for centralized log collection. Follow these steps to complete the task:

- **Create Azure Event Hubs Namespace:**

  - Create an Event Hubs namespace named `xfusion-namespace` in the East US region.
  - Select the **Standard** pricing tier. Make sure to enable `Enable Auto-inflate`.

- **Create an Event Hub:**

  - Within the namespace, create an Event Hub named `xfusion-hub`.

- **Verify the Virtual Machine Configuration:**

  - A VM named `xfusion-vm` already exists.
  - A Python script named `send_logs.py` already exists on the VM under `/home/azureuser`. This script is used to send logs to the Event Hub. Make sure to execute this script mutiple times.

- **Verify Logs:**

  - Ensure the logs are successfully sent to the Event Hub by checking the Event Hubs metrics in the Azure portal.
	
`Notes:`

- Create the resources only in the `East US` region.
- Use the existing VM `xfusion-vm` to send logs.
- Verify the Event Hubs metrics to confirm successful log ingestion.

---

# Step-by-Step Implementation

### Step 1: Get the Resource Group

```bash
RG=$(az group list --query "[0].name" -o tsv)

echo $RG
```

#### Explanation

Retrieves the default Resource Group where all Azure resources will be created.

---

### Step 2: Create the Event Hubs Namespace (GUI)

Open:

```text
Azure Portal
→ Event Hubs
→ Create
```

Configure:

| Field               | Value               |
| ------------------- | ------------------- |
| Resource Group      | Existing RG         |
| Namespace           | `xfusion-namespace` |
| Region              | East US             |
| Pricing Tier        | Standard            |
| Enable Auto-inflate | Enabled             |

Click:

```text
Review + Create
→ Create
```

#### Explanation

Creates the Event Hubs namespace that will host one or more Event Hubs.

---

### Step 3: Create the Event Hub (GUI)

After deployment:

```text
xfusion-namespace
→ Event Hubs
→ + Event Hub
```

Configure:

| Field             | Value         |
| ----------------- | ------------- |
| Name              | `xfusion-hub` |
| Partition Count   | Default       |
| Message Retention | Default       |

Click:

```text
Create
```

#### Explanation

Creates an Event Hub that will receive log events from the Virtual Machine.

---

### Step 4: Verify the Existing Virtual Machine

```bash
az vm list -o table
```

Expected:

```text
Name
------------
xfusion-vm
```

#### Explanation

Ensures the required VM already exists.

---

### Step 5: Get the VM Public IP

```bash
VMIP=$(az vm show -d \
-g $RG \
-n xfusion-vm \
--query publicIps \
-o tsv)

echo $VMIP
```

#### Explanation

Retrieves the public IP required to SSH into the VM.

---

### Step 6: SSH into the VM

```bash
ssh azureuser@$VMIP
```

#### Explanation

Connects to the VM where the Python logging script already exists.

---

### Step 7: Verify the Python Script

```bash
ls -l /home/azureuser/send_logs.py
```

Expected:

```text
-rwxr-xr-x 1 azureuser azureuser ... send_logs.py
```

#### Explanation

Confirms that the log sender script is available.

---

### Step 8: Retrieve the Event Hub Connection String

Exit the VM if necessary and run:

```bash
az eventhubs namespace authorization-rule keys list \
--resource-group $RG \
--namespace-name xfusion-namespace \
--name RootManageSharedAccessKey
```

Copy the value of:

```text
primaryConnectionString
```

#### Explanation

This connection string allows the Python application to send logs to the Event Hub.

---

### Step 9: Update the Python Script

SSH back into the VM if needed:

```bash
ssh azureuser@$VMIP
```

Edit the script:

```bash
nano /home/azureuser/send_logs.py
```

Replace:

```python
connection_str = "<your_event_hub_connection_string>"
```

with:

```python
connection_str = "Endpoint=sb://xfusion-namespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=<your-key>"
```

Save:

```text
CTRL + O
Enter
CTRL + X
```

#### Explanation

Updates the script with the valid Event Hubs connection string.

---

### Step 10: Execute the Script

Run once:

```bash
python3 /home/azureuser/send_logs.py
```

Expected:

```text
Log entry 1 sent.
Log entry 2 sent.
...
Log entry 10 sent.
```

Execute multiple times:

```bash
for i in {1..5}
do
    python3 /home/azureuser/send_logs.py
done
```

#### Explanation

Sends multiple batches of log messages to the Event Hub.

---

### Step 11: Verify Incoming Messages Using Azure CLI

Exit the VM and run:

```bash
RESOURCE_ID=$(az eventhubs namespace show \
-g $RG \
-n xfusion-namespace \
--query id -o tsv)
```

Then:

```bash
az monitor metrics list \
--resource "$RESOURCE_ID" \
--metric IncomingMessages \
--aggregation Total
```

Expected output:

```json
{
  "value": [
    {
      "name": {
        "value": "IncomingMessages"
      },
      "timeseries": [
        {
          "data": [
            {
              "total": 65
            }
          ]
        }
      ]
    }
  ]
}
```

#### Explanation

Confirms that the Event Hub has received the log messages.

---

### Step 12: Verify in Azure Portal

Navigate to:

```text
Azure Portal
→ Event Hubs
→ xfusion-namespace
→ xfusion-hub
→ Monitoring
→ Metrics
```

Configure:

| Field       | Value             |
| ----------- | ----------------- |
| Metric      | Incoming Messages |
| Aggregation | Sum               |

Expected:

```text
Incoming Messages (Sum): 65
```

#### Explanation

Provides visual confirmation that logs are being ingested successfully.

---

## Azure CLI Verification

### Verify Namespace

```bash
az eventhubs namespace show \
-g $RG \
-n xfusion-namespace \
-o table
```

---

### Verify Event Hub

```bash
az eventhubs eventhub show \
-g $RG \
--namespace-name xfusion-namespace \
-n xfusion-hub \
-o table
```

---

### List Event Hubs

```bash
az eventhubs eventhub list \
-g $RG \
--namespace-name xfusion-namespace \
-o table
```

---

### Verify Metrics

```bash
RESOURCE_ID=$(az eventhubs namespace show \
-g $RG \
-n xfusion-namespace \
--query id -o tsv)

az monitor metrics list \
--resource "$RESOURCE_ID" \
--metric IncomingMessages \
--aggregation Total
```

---

## Common Errors & Fixes

### Error: Connection String is Blank or Malformed

**Cause**

The placeholder connection string was not replaced.

**Fix**

Retrieve the namespace connection string:

```bash
az eventhubs namespace authorization-rule keys list \
--resource-group $RG \
--namespace-name xfusion-namespace \
--name RootManageSharedAccessKey
```

Update the `connection_str` variable in `send_logs.py`.

---

### Error: AuthorizationRuleNotFound

**Cause**

Using a non-existent Event Hub authorization rule.

**Fix**

Retrieve keys from the namespace-level authorization rule:

```text
RootManageSharedAccessKey
```

---

### Error: ModuleNotFoundError

**Cause**

Required Python SDK is missing.

**Fix**

Install the Azure Event Hub SDK:

```bash
pip3 install azure-eventhub
```

---

### Error: Incoming Messages Metric Shows Zero

**Cause**

The script wasn't executed or the connection string is incorrect.

**Fix**

* Verify the connection string.
* Execute `send_logs.py` multiple times.
* Wait 1–2 minutes for metrics to update.

---

## Best Practices

* Use the **Standard** pricing tier when Auto-inflate is required.
* Store Event Hub connection strings securely (e.g., Azure Key Vault) instead of hardcoding them.
* Verify metrics after sending events, as Azure Monitor may take a minute to refresh.
* Test scripts multiple times to confirm reliable event ingestion.

## Key Learnings

* Creating Azure Event Hubs namespaces and Event Hubs.
* Configuring Auto-inflate for scalable event ingestion.
* Connecting Virtual Machines to Azure Event Hubs.
* Sending logs using a Python application.
* Monitoring Event Hub metrics to verify successful log ingestion.
* Troubleshooting connection string and authorization issues.
