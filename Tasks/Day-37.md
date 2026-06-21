# Day 37 – Setting Up MySQL on a Virtual Machine in Azure

---

## Task Overview

The Nautilus DevOps team is tasked with integrating a PHP application hosted on an Azure VM with a MySQL database hosted on another Azure VM. This will validate the application's ability to connect to the database in the cloud. 

**1) Create the MySQL VM:** 

- Create a Mysql server named `xfusion-mysql-vm` using the **Percona Server for MySQL** image (published by **Jetware**) from the Azure Marketplace.
- Configure the VM in the **Central US** region.
- Use `Password` as the authentication type.
- Set the **username** as `xfusion_admin` and the **password** as `Namin@123456`.
- Size: `Standard B1s`
- Allow the public inbound port `22` during VM creation. Port `3306` is not selectable in the create wizard, so add it as an inbound rule to the VM's network security group (NSG) after the VM is created to enable MySQL access.
- Under Disks tab choose OS disk type : `standard HDD`

**2) Setup the MySQL Database:** 

- SSH into the `xfusion-mysql-vm`.
- Use the `sudo /jet/enter mysql` command to access the MySQL shell.
- Create a database named `xfusion_db`.
- Create a MySQL user named `xfusion_user` with password `password123`.
- The user must be created with the `'%'` host (i.e., `'xfusion_user'@'%'`) so it can connect remotely from the PHP VM.
- Grant all privileges on the `xfusion_db` database to `'xfusion_user'@'%'`.
- Ensure the privileges are applied by running the appropriate command to flush privileges.

**3) PHP VM Setup:** 

- A VM named `xfusion-php-vm` already exists in the **East US** region.
- This VM is hosting a PHP application and contains a pre-existing `db_test.php` file in the `/var/www/html/` directory.
- Passwordless SSH access to this **VM is already configured from the lab host**, so you can simply run `ssh azureuser@<php-vm-public-ip>` to log in (no password required).

**4) Database Connection Configuration:** 

- Retrieve the public IP address of the `xfusion-mysql-vm`.
- SSH into the `xfusion-php-vm` by running `ssh azureuser@<php-vm-public-ip>` from the lab host, then open the `/var/www/html/db_test.php` file.
- Update the connection variables to reflect the following:
```bash
$servername = "<mysql-vm-public-ip>";
$username = "xfusion_user";
$password = "password123";
$dbname = "xfusion_db";
$port = 3306;
```
- Ensure that the mysqli connection utilizes the port variable as the fifth argument, for example:
```bash
$conn = new mysqli($servername, $username, $password, $dbname, $port);
```
- Save the file and verify there are no syntax errors before testing the connection in a browser.

**5) Validation:** 

- Access the `db_test.php` file from the `xfusion-php-vm` using its public IP address/db_test.php.
- Ensure the file displays the message `Connected successfully`, confirming the connection between the PHP application and the MySQL database.

`Notes:` 

- Ensure the MySQL database allows inbound traffic on port `3306`.
- Verify that the PHP application on the `xfusion-php-vm` successfully connects to the MySQL database on the `xfusion-mysql-vm`.

---

## Step-by-Step Implementation

### Step 1: Verify Existing PHP VM

On azure-client:

```bash
RG=$(az group list --query "[0].name" -o tsv)

az vm list -o table
```

#### Explanation

Verify that the existing PHP VM (`xfusion-php-vm`) is available in the lab environment.

---

### Step 2: Create MySQL VM

Navigate to:

```text
Azure Portal
→ Virtual Machines
→ Create
```

Configure:

| Field                | Value                              |
| -------------------- | ---------------------------------- |
| VM Name              | `xfusion-mysql-vm`                 |
| Region               | `Central US`                       |
| Image                | Percona Server for MySQL (Jetware) |
| Size                 | Standard_B1s                       |
| Authentication Type  | Password                           |
| Username             | xfusion_admin                      |
| Password             | Namin@123456                       |
| Public Inbound Ports | SSH (22)                           |

#### Explanation

Creates the MySQL server VM using the Percona Server image from Azure Marketplace.

---

### Step 3: Configure OS Disk

Under the **Disks** tab:

| Field        | Value              |
| ------------ | ------------------ |
| OS Disk Type | Standard HDD (LRS) |

#### Explanation

Ensures compatibility with lab policies and minimizes storage costs.

---

### Step 4: Create the VM

Click:

```text
Review + Create
→ Create
```

Wait until deployment completes successfully.

#### Explanation

Deploys the MySQL virtual machine in Azure.

---

### Step 5: Open MySQL Port 3306

Navigate:

```text
xfusion-mysql-vm
→ Networking
→ Inbound Port Rules
→ Add
```

Configure:

| Field            | Value       |
| ---------------- | ----------- |
| Protocol         | TCP         |
| Destination Port | 3306        |
| Action           | Allow       |
| Priority         | 300         |
| Name             | Allow-MySQL |

#### Explanation

Allows remote MySQL connections from the PHP application server.

---

### Step 6: Retrieve MySQL VM Public IP

```bash
az vm show -d \
--resource-group $RG \
--name xfusion-mysql-vm \
--query publicIps \
-o tsv
```

#### Explanation

Obtain the public IP address required for PHP database connectivity.

---

### Step 7: SSH into MySQL VM

```bash
ssh xfusion_admin@<MYSQL_PUBLIC_IP>
```

Password:

```text
Namin@123456
```

#### Explanation

Connect to the MySQL VM for database configuration.

---

### Step 8: Access MySQL Console

```bash
sudo /jet/enter mysql
```

#### Explanation

Launches the Percona MySQL shell provided by the Jetware image.

---

### Step 9: Create Database

```sql
CREATE DATABASE xfusion_db;
```

#### Explanation

Creates the application database.

---

### Step 10: Create MySQL User

```sql
CREATE USER 'xfusion_user'@'%' IDENTIFIED BY 'password123';
```

#### Explanation

Creates a remote-access database user.

---

### Step 11: Grant Permissions

```sql
GRANT ALL PRIVILEGES ON xfusion_db.* TO 'xfusion_user'@'%';
```

#### Explanation

Provides full access to the application database.

---

### Step 12: Apply Changes

```sql
FLUSH PRIVILEGES;
```

#### Explanation

Reloads MySQL privilege tables.

---

### Step 13: Verify Database

```sql
SHOW DATABASES;
```

Expected:

```text
xfusion_db
```

Exit MySQL:

```sql
EXIT;
```

#### Explanation

Confirms successful database creation.

---

### Step 14: Connect to PHP VM

Get the PHP VM public IP and SSH:

```bash
ssh azureuser@<PHP_VM_PUBLIC_IP>
```

#### Explanation

Access the existing PHP application server.

---

### Step 15: Edit Application Configuration

Open:

```bash
sudo nano /var/www/html/db_test.php
```

Update the file:

```php
<?php
$servername = "<MYSQL_PUBLIC_IP>";
$username = "xfusion_user";
$password = "password123";
$dbname = "xfusion_db";
$port = 3306;

$conn = new mysqli(
    $servername,
    $username,
    $password,
    $dbname,
    $port
);

if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}

echo "Connected successfully";
?>
```

#### Explanation

Configures the PHP application to connect to the MySQL database.

---

### Step 16: Verify PHP Syntax

```bash
php -l /var/www/html/db_test.php
```

Expected:

```text
No syntax errors detected
```

#### Explanation

Validates the PHP configuration file.

---

### Step 17: Verify MySQL Connectivity

```bash
timeout 5 bash -c '</dev/tcp/<MYSQL_PUBLIC_IP>/3306' && echo OPEN || echo CLOSED
```

Expected:

```text
OPEN
```

#### Explanation

Confirms that port 3306 is reachable.

---

### Step 18: Verify Apache Service

```bash
sudo systemctl status apache2 --no-pager
```

Expected:

```text
active (running)
```

#### Explanation

Ensures the web server is operational.

---

### Step 19: Test the Application

Open a browser:

```text
http://<PHP_VM_PUBLIC_IP>/db_test.php
```

Expected output:

```text
Connected successfully
```

#### Explanation

Confirms successful communication between the PHP application and MySQL database.

---

## Best Practices

* Use dedicated database users instead of root accounts.
* Restrict NSG rules to required ports only.
* Validate connectivity before application testing.
* Always flush MySQL privileges after permission changes.
* Store database credentials securely in production environments.

## Key Learnings

* Deploying Percona MySQL from Azure Marketplace.
* Configuring Azure NSG rules for database access.
* Creating remote MySQL users using `%` host.
* Connecting PHP applications to external MySQL databases.
* Troubleshooting database connectivity issues in Azure.
