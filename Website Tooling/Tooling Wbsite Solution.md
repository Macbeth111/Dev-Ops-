# DevOps Tooling Website Solution

## Table of Contents

1. [Introduction](#introduction)
2. [Setting Up AWS EC2 Instances](#setting-up-aws-ec2-instances)
3. [Configuring NFS Server](#configuring-nfs-server)
    - [EBS Setup](#ebs-setup)
    - [LVM Configuration](#lvm-configuration)
    - [Install and Configure NFS Server](#install-and-configure-nfs-server)
4. [Configuring Database Server](#configuring-database-server)
    - [EBS Setup](#ebs-setup-1)
    - [LVM Configuration](#lvm-configuration-1)
    - [AWS Security Group Configuration](#aws-security-group-configuration)
    - [Install and Configure MySQL Database Server](#install-and-configure-mysql-database-server)
5. [Configuring Web Servers](#configuring-web-servers)
    - [Installing NFS Client](#installing-nfs-client)
    - [Mounting NFS Shares](#mounting-nfs-shares)
    - [Installing PHP and Extensions](#installing-php-and-extensions)
6. [Deploying Tooling Website](#deploying-tooling-website)
7. [Final Steps and Reflections](#final-steps-and-reflections)

---

## Introduction
This project focuses on deploying a DevOps tooling website using AWS EC2 instances. The architecture consists of an NFS server for shared storage, a database server to manage data, and multiple web servers running the application. The objective is to set up and configure these components in an efficient and structured manner.

---

## Setting Up AWS EC2 Instances
To begin, I launched the necessary AWS EC2 instances following these steps:

### 1. Launching the NFS Server:
- I navigated to the EC2 dashboard in the AWS Console.
- Clicked **"Launch Instance"** and selected **Red Hat Enterprise Linux 9.4** as the AMI.
- Chose the `t2.small` instance type for moderate compute power.
- Configured instance details (including VPC and subnet settings).
- Added **3 x 15GB EBS volumes** for storage.
- Configured the security group (detailed later).
- Reviewed and launched the instance using my EC2 key pair.

### 2. Launching the Web Servers (Repeated for Three Instances):
- Followed the same steps as for the NFS server.
- Chose the `t2.small` instance type.
- Did **not** add additional EBS volumes.

### 3. Launching the Database Server:
- Followed similar steps as the NFS server.
- Chose the `t2.micro` instance type for cost efficiency.
- Added **3 x 10GB EBS volumes** for database storage.

### 4. Verifying Instances:
- Ensured that all instances were in a "running" state.
- Noted down the **private IP addresses** of all instances for configuration.
- Confirmed that all instances were in the same subnet.

### 5. Tagging Instances:
- Added descriptive tags such as `NFS Server`, `Web Server 1`, `DB Server`.
- Marked the environment as `Learning/Development` for clarity.

### 6. Configuring Elastic IPs (Optional):
- Allocated and associated **Elastic IPs** to the web servers for public access if required.

### **Important Considerations:**
- All instances were deployed within **the same VPC and subnet** for simplicity, but in production, they should be separated for security.
- Used the **same key pair** for all instances to facilitate access management.
- Ensured that the **correct EBS volumes** were attached to the NFS and DB servers.

> **Note:** In a production environment, it is best practice to separate web, application, and database servers into different subnets (e.g., public subnet for web servers, private subnet for databases) to enhance security through network segregation.

---

## Configuring NFS Server

### Connecting to the NFS Server
After launching the NFS server, I connected via SSH using:
```bash
ssh -i "my-key.pem" ec2-user@<NFS_SERVER_IP>
```

### EBS Setup
To confirm that the EBS volumes were correctly attached, I listed the block devices:
```bash
lsblk
```
The output showed the new volumes as `/dev/xvdb`, `/dev/xvdc`, and `/dev/xvdd`.

### LVM Configuration
I installed LVM and other necessary tools:
```bash
sudo dnf update -y
sudo dnf install lvm2 nano -y
```
Since these were new volumes, no partitions were present initially.

#### Creating Partitions on EBS Volumes
I used `gdisk` to create partitions on each block:
```bash
sudo gdisk /dev/xvdb
```
Inside the `gdisk` interface, I ran:
- `n` - Create a new partition
- `p` - Print the partition table
- `w` - Write changes to disk

I repeated this for `/dev/xvdc` and `/dev/xvdd`.

#### Creating Physical Volumes
I initialized the partitions as physical volumes:
```bash
sudo pvcreate /dev/xvdb1 /dev/xvdc1 /dev/xvdd1
```

#### Creating a Volume Group
I aggregated the physical volumes into a single **volume group**:
```bash
sudo vgcreate webdata-vg /dev/xvdb1 /dev/xvdc1 /dev/xvdd1
```

#### Creating Logical Volumes
I created separate logical volumes (LVs) for applications, logs, and Jenkins:
```bash
sudo lvcreate -n lv-apps -L 14G webdata-vg
sudo lvcreate -n lv-logs -L 14G webdata-vg
sudo lvcreate -n lv-opt -L 14G webdata-vg
```
To verify:
```bash
lsblk
```

#### Creating File System and Mount Points
I formatted the logical volumes using **XFS**:
```bash
sudo mkfs -t xfs /dev/webdata-vg/lv-apps
sudo mkfs -t xfs /dev/webdata-vg/lv-logs
sudo mkfs -t xfs /dev/webdata-vg/lv-opt
```

Then, I created mount points:
```bash
sudo mkdir /mnt/apps /mnt/logs /mnt/opt
```

### Install and Configure NFS Server
I installed and enabled the **NFS Server**:
```bash
sudo dnf install nfs-utils -y
sudo systemctl start nfs-server
sudo systemctl enable nfs-server
```

#### Configuring NFS Access
I modified the **exports file** to allow access within the subnet:
```bash
sudo nano /etc/exports
```
Added the following:
```bash
/mnt/apps 172.31.0.0/20(rw,sync,no_all_squash,no_root_squash)
/mnt/logs 172.31.0.0/20(rw,sync,no_all_squash,no_root_squash)
/mnt/opt 172.31.0.0/20(rw,sync,no_all_squash,no_root_squash)
```
Applied the configuration:
```bash
sudo exportfs -arv
```

Checked the **NFS Server Port**:
```bash
rpcinfo -p | grep nfs
```
---

# Configuring Database Server

## 1. Connect to the DB-Server Instance

### Significance:
- This step allows access to the database server via SSH, enabling administrative setup and configuration.

## 2. EBS Setup

### Check Volume Visibility:
```bash
lsblk
```
#### Significance:
- Lists all attached block devices, including newly attached EBS volumes.
- Helps identify device names like `/dev/xvdb`, `/dev/xvdc`, and `/dev/xvdd`.
- Ensures that the new volumes are visible to the system.

## 3. LVM Configuration

### Install LVM Tools:
```bash
sudo dnf update
sudo dnf install lvm2 nano wget
```
#### Significance:
- `dnf update`: Ensures the system is up-to-date before installing necessary tools.
- `lvm2`: Provides logical volume management capabilities.
- `nano`: A text editor for editing configuration files.
- `wget`: A tool for downloading files from the internet if needed.

### Check for Existing Partitions:
```bash
lvmdiskscan
```
#### Significance:
- Scans for available disk partitions.
- Since these are fresh EBS volumes, no partitions exist yet.

### Create Partitions on EBS Volumes:
```bash
sudo gdisk /dev/xvdb
```
#### Significance:
- `gdisk`: Used for partitioning GPT disks.
- Creates a new partition table and writes changes to the disk.

**Partition Commands:**
- `n`: Creates a new partition.
- `p`: Prints the partition table.
- `w`: Writes changes to disk.

Repeat the process for other EBS blocks:
```bash
sudo gdisk /dev/xvdc
sudo gdisk /dev/xvdd
```

### Confirm Partition Creation:
```bash
lsblk
```
#### Significance:
- Verifies that partitions (`xvdb1`, `xvdc1`, `xvdd1`) have been successfully created.

### Convert EBS Partitions to Physical Volumes:
```bash
sudo pvcreate /dev/xvdb1 /dev/xvdc1 /dev/xvdd1
```
#### Significance:
- Converts raw partitions into physical volumes (PVs) for LVM use.

### Create a Volume Group:
```bash
sudo vgcreate dbdata-vg /dev/xvdb1 /dev/xvdc1 /dev/xvdd1
```
#### Significance:
- Groups multiple physical volumes into a single volume group (VG) for easier storage management.

### Create Logical Volumes:
```bash
sudo lvcreate -n db-lv -L 14G dbdata-vg
sudo lvcreate -n log-lv -L 14G dbdata-vg
```
#### Significance:
- `db-lv`: Logical volume for database storage.
- `log-lv`: Logical volume for system logs.

### Verify Setup:
```bash
sudo vgdisplay -v
```
#### Significance:
- Displays details of volume groups, physical volumes, and logical volumes.

## 4. Create Filesystem and Mount LVs

### Format the Logical Volumes:
```bash
sudo mkfs -t ext4 /dev/dbdata-vg/db-lv
sudo mkfs -t ext4 /dev/dbdata-vg/log-lv
```
#### Significance:
- `mkfs -t ext4`: Creates an ext4 filesystem on logical volumes for data storage.

### Create Mount Directories:
```bash
sudo mkdir -p /db /home/recovery/logs
```
#### Significance:
- Creates necessary directories for mounting storage.
- `-p`: Ensures parent directories are created if they don’t exist.

### Backup Existing Logs:
```bash
sudo rsync -av /var/log/ /home/recovery/logs/
```
#### Significance:
- Creates a backup before mounting new storage.
- `-a`: Preserves file attributes.
- `-v`: Shows detailed output.

### Mount the Logical Volumes:
```bash
sudo mount /dev/dbdata-vg/db-lv /db
sudo mount /dev/dbdata-vg/log-lv /var/log
```
#### Significance:
- Mounts the logical volumes to their respective directories.

### Restore Logs:
```bash
sudo rsync -av /home/recovery/logs/ /var/log/
```
#### Significance:
- Restores backed-up log files to the new mounted volume.

## 5. Persistent Mounts (FSTAB Configuration)

### Get UUIDs of Logical Volumes:
```bash
sudo blkid
```
#### Significance:
- Retrieves UUIDs (unique identifiers) for disk partitions, ensuring correct mounting.

### Edit `/etc/fstab` for Persistent Mounting:
```bash
sudo nano /etc/fstab
```
**Add the following entries:**
```bash
UUID=ac925709-a000-4cb5-926d-bc9c792c499c /db ext4 defaults 0 0
UUID=44d455e2-38ff-414e-839e-39110887379d /var/log ext4 defaults 0 0
```
#### Significance:
- Ensures the storage volumes are mounted automatically on system boot.

### Reload System Daemon and Test Mounting:
```bash
sudo systemctl daemon-reload
sudo mount -a
```
#### Significance:
- Reloads system services to apply changes.
- `mount -a`: Tests if all entries in `/etc/fstab` mount correctly.

## 6. AWS Security Group Configuration

### MySQL Security Group Settings:
- **Port 3306:** Allows MySQL traffic from the subnet (172.31.0.0/20).
- **Port 22:** Enables SSH access for remote administration.

## 7. Install and Configure MySQL Database Server

### Install MySQL Server:
```bash
sudo dnf update
sudo dnf install mysql-server -y
```
#### Significance:
- Installs MySQL, the database engine for WordPress.

### Start and Enable MySQL Service:
```bash
sudo systemctl start mysqld
sudo systemctl enable mysqld
```
#### Significance:
- Ensures MySQL starts automatically on boot.

### Secure MySQL Installation:
```bash
sudo mysql_secure_installation
```
#### Significance:
- Improves security by setting root password, removing anonymous users, and disabling remote root login.

### Create Database and User for WordPress:
```bash
sudo mysql -u root -p
CREATE DATABASE tooling;
CREATE USER 'webaccess'@'172.31.%' IDENTIFIED WITH mysql_native_password BY 'Password.1';
GRANT ALL PRIVILEGES ON tooling.* TO 'webaccess'@'172.31.%';
FLUSH PRIVILEGES;
EXIT;
```
#### Significance:
- Creates a `tooling` database for WordPress.
- `webaccess` user is granted permissions to access the database remotely.

## 8. Test Remote MySQL Connection

### Install MySQL Client on Web Server:
```bash
sudo dnf update
sudo dnf install mysql nano
```

### Connect to Remote MySQL Server:
```bash
sudo mysql -h 172.31.3.89 -u webaccess -p
```
#### Significance:
- Ensures WordPress can connect to MySQL from a separate instance.
- If successful, the database setup is complete.

---

# Configuring Web Servers

## Overview
We will be configuring three web servers for NFS server connection and installing both Apache and PHP on them. This setup ensures that our application can handle HTTP requests efficiently while maintaining a centralized storage solution with NFS.

---

## Installing NFS Client

### Step 1: Install NFS Client and Apache
The NFS client is required to enable the web servers to access shared storage from the NFS server. Apache is necessary to handle HTTP requests.

```bash
sudo dnf install nfs-utils nfs4-acl-tools -y
sudo dnf install httpd git
```

- `nfs-utils`: Provides the necessary utilities for mounting and managing NFS shares.
- `nfs4-acl-tools`: Helps manage ACLs (Access Control Lists) on NFS shares.
- `httpd`: Installs Apache, the most commonly used web server for WordPress and PHP applications.
- `git`: Needed to clone the application repository from GitHub.

### Step 2: Start and Enable Apache

```bash
sudo systemctl start httpd
sudo systemctl enable httpd
```

- `start httpd`: Launches the Apache service immediately.
- `enable httpd`: Ensures Apache starts automatically at boot.

### Step 3: Verify Apache Status

```bash
sudo systemctl status httpd
```

If Apache is running correctly, you can access it via a browser:

```
http://your-server-ip
```

If configured correctly, the default RedHat page should be displayed.

---

## Mounting NFS Shares

### Step 1: Mount `/var/www/` and Target the NFS Server

```bash
sudo mount -t nfs -o rw,nosuid 172.31.1.101:/mnt/apps /var/www
```

- `mount`: Command to mount a file system.
- `-t nfs`: Specifies the file system type (NFS in this case).
- `-o rw,nosuid`:
  - `rw`: Grants read-write permissions.
  - `nosuid`: Prevents execution of privileged programs from the mounted share, improving security.
- `172.31.1.101:/mnt/apps`: IP address and directory of the NFS server.
- `/var/www`: Local directory where the remote share is mounted.

### Step 2: Verify NFS Mount

```bash
df -h
```

### Step 3: Make the Mount Persistent
To ensure the mount persists across reboots, update `/etc/fstab`:

```bash
sudo nano /etc/fstab
```

Add the following line:

```
172.31.1.101:/mnt/apps /var/www nfs defaults 0 0
```

Apply the changes:

```bash
sudo systemctl daemon-reload
sudo mount -a
```

---

## Installing PHP and Extensions

### Step 1: Verify RHEL Version

```bash
cat /etc/redhat-release
```

Expected Output:

```
Red Hat Enterprise Linux release 9.4 (Plow)
```

### Step 2: Enable Necessary Repositories
We need to enable additional repositories to install PHP 8.3, as RHEL’s default repositories provide only up to PHP 8.2.

```bash
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-9.rpm
```

### Step 3: Install PHP 8.3 and Required Extensions

```bash
sudo dnf module switch-to php:remi-8.3
sudo dnf module install php:remi-8.3
sudo dnf install php php-opcache php-gd php-curl php-mysqlnd php-xml php-json php-mbstring php-intl php-soap php-zip
```

Explanation of Key Extensions:
- `php-opcache`: Improves performance by caching precompiled scripts.
- `php-gd`: Required for image processing.
- `php-curl`: Enables external HTTP requests.
- `php-mysqlnd`: Native MySQL driver.
- `php-xml`: Enables XML parsing.
- `php-json`: Handles JSON data processing.
- `php-mbstring`: Supports multi-byte character strings.
- `php-intl`: Provides internationalization features.
- `php-soap`: Enables SOAP-based communication.
- `php-zip`: Needed for handling ZIP archives.

### Step 4: Start and Enable PHP-FPM

```bash
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
```

---

## Configuring SELinux

### Step 1: Verify SELinux Status

```bash
sestatus
```

Expected Output:

```
SELinux status:                 enabled
Current mode:                   enforcing
```

### Step 2: Configure SELinux Policies

```bash
sudo setsebool -P httpd_execmem 1
sudo setsebool -P httpd_can_network_connect 1
sudo setsebool -P httpd_use_nfs on
```

Verify:

```bash
getsebool httpd_use_nfs
getsebool httpd_execmem
getsebool httpd_can_network_connect
```

Expected output:
```
httpd_use_nfs --> on
httpd_execmem --> on
httpd_can_network_connect --> on
```

---

## Deploying Tooling Website

### Step 1: Clone the Application Repository

```bash
sudo git clone https://github.com/fmanimashaun/tooling.git
```

### Step 2: Import Database

```bash
cd tooling
sudo mysql -h 172.31.3.89 -u webaccess -p tooling < tooling-db.sql
```

### Step 3: Add Admin User to Database

```bash
sudo mysql -h 172.31.3.89 -u webaccess -p tooling
```

Inside MySQL:
```sql
INSERT INTO `users` (`username`, `password`, `email`, `user_type`, `status`)
VALUES ('myuser', MD5('password'), 'user@mail.com', 'admin', '1');
```

### Step 4: Update Database Connection in `function.php`

```bash
sudo nano /var/www/html/function.php
```

Update:
```php
$db = mysqli_connect('172.31.3.89', 'webaccess', 'Password.1', 'tooling');
```

### Step 5: Restart Apache

```bash
sudo systemctl restart httpd
```

### Step 6: Access the Application

Visit:

```
http://<public-ip-of-webserver>/index.php
```

Login using:
- **Username:** myuser
- **Password:** password

---






---

