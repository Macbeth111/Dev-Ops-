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
  
![IMG-20250131-WA0006](https://github.com/user-attachments/assets/ff209b88-d6cd-41cb-90e8-a28db7da508d)

### 5. Tagging Instances:
- Added descriptive tags such as `NFS Server`, `Web Server 1`, `DB Server`.
- Marked the environment as `Learning/Development` for clarity.

### **Important Considerations:**
- All instances were deployed within **the same VPC and subnet** for simplicity, but in production, they should be separated for security.
- Used the **same key pair** for all instances to facilitate access management.
- Ensured that the **correct EBS volumes** were attached to the NFS and DB servers.

> **Note:** In a production environment, it is best practice to separate web, application, and database servers into different subnets (e.g., public subnet for web servers, private subnet for databases) to enhance security through network segregation.

> **Note:** You must create volumes for the NFS and Database server and attach them to the instances respectively
![IMG-20250130-WA0002](https://github.com/user-attachments/assets/b19879cd-2138-498a-a1fc-b77f9783d678)
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
![lsblk 1](https://github.com/user-attachments/assets/4e62dd3d-f8a7-44ef-905b-6410c6a87699)

### LVM Configuration
I installed LVM and other necessary tools:
```bash
sudo dnf update -y
sudo dnf install lvm2 nano -y
```
#### Creating Partitions on EBS Volumes
I used `gdisk` to create partitions on each block:
```bash
sudo gdisk /dev/xvdb
```
![Partition](https://github.com/user-attachments/assets/67e58706-c992-4c2e-8e7c-89420836b63e)
Inside the `gdisk` interface,
 I ran:
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
![all volumes created](https://github.com/user-attachments/assets/039251dc-decd-4957-a921-37ef18028371)

#### Creating File System and Mount Points
I formatted the logical volumes using **XFS**:
```bash
sudo mkfs -t xfs /dev/webdata-vg/lv-apps
sudo mkfs -t xfs /dev/webdata-vg/lv-logs
sudo mkfs -t xfs /dev/webdata-vg/lv-opt
```
![File system and mount point](https://github.com/user-attachments/assets/1b48db88-6b2a-4c17-ba1e-85da5104214d)

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
![NFS status](https://github.com/user-attachments/assets/7fcf2095-02dc-4289-8871-8a2dbf5d5ff7)

#### Configuring NFS Access
I modified the **exports file** to allow access within the subnet:
```bash
sudo vi /etc/exports
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
![exports vi edited](https://github.com/user-attachments/assets/63827469-6747-4942-97ed-d05e903042e2)

Checked the **NFS Server Port**:
```bash
rpcinfo -p | grep nfs
```
![nfs port checked](https://github.com/user-attachments/assets/fe66d0d9-90da-4583-8f80-ebc7b3a94401)

Then I went ahead and edited the inbound rules for the 'NFS server'

![IMG-20250130-WA0001](https://github.com/user-attachments/assets/a68f61aa-7849-4d62-8f46-58f331dd64e4)

# Configuring Database Server

## 1. Connect to the DB-Server Instance

### Significance:
- This step allows access to the database server via SSH, enabling administrative setup and configuration.

## 2. EBS Setup

### Check Volume Visibility:
```bash
lsblk
```
![db lsblk](https://github.com/user-attachments/assets/182076aa-5697-4046-afb2-e7a88aebc2cc)

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
![db partitions confirmed](https://github.com/user-attachments/assets/41126d8b-4048-46d1-8e9a-bf378164b34e)
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
![db lv created](https://github.com/user-attachments/assets/3da2793d-1591-478d-8494-b9f8e0cf4d81)

#### Significance:
- `db-lv`: Logical volume for database storage.
- `log-lv`: Logical volume for system logs.

### Verify Setup:
```bash
sudo vgdisplay -v
```
![IMG-20250130-WA0028](https://github.com/user-attachments/assets/579b91ce-050f-4478-be69-008361588611)

![IMG-20250130-WA0030](https://github.com/user-attachments/assets/67baa961-4670-4e63-a251-21d04f05f5b8)

#### Significance:
- Displays details of volume groups, physical volumes, and logical volumes.

## 4. Create Filesystem and Mount LVs

### Format the Logical Volumes:
```bash
sudo mkfs -t ext4 /dev/dbdata-vg/db-lv
sudo mkfs -t ext4 /dev/dbdata-vg/log-lv
```
![db ext4](https://github.com/user-attachments/assets/2c720bd4-0061-4565-8788-20faa3d4005d)

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
![sudo blkid db](https://github.com/user-attachments/assets/f5ed63db-c68b-4560-a5ea-8fbd8715ef06)

#### Significance:
- Retrieves UUIDs (unique identifiers) for disk partitions, ensuring correct mounting.

### Edit `/etc/fstab` for Persistent Mounting:
```bash
sudo vi /etc/fstab
```
**Add the following entries:**

![db UUIDs](https://github.com/user-attachments/assets/99eb2fdc-e860-46ed-a3da-07d4ef63224c)

#### Significance:
- Ensures the storage volumes are mounted automatically on system boot.

### Reload System Daemon and Test Mounting:
```bash
sudo systemctl daemon-reload
sudo mount -a
```
![db mount a](https://github.com/user-attachments/assets/16bfbc45-3377-4b8d-a65b-3c6bbdea94a4)

#### Significance:
- Reloads system services to apply changes.
- `mount -a`: Tests if all entries in `/etc/fstab` mount correctly.

## 6. AWS Security Group Configuration

### MySQL Security Group Settings:
- **Port 3306:** Allows MySQL traffic from the subnet (172.31.0.0/20).

![IMG-20250130-WA0025](https://github.com/user-attachments/assets/8439a05c-db08-47fe-b159-ac5801f83e26)

## 7. Install and Configure MySQL Database Server

### Install MySQL Server:
```bash
sudo dnf update
sudo dnf install mysql-server -y
```
### Start and Enable MySQL Service:
```bash
sudo systemctl start mysqld
sudo systemctl enable mysqld
sudo systemctl status mysqld
```
![IMG-20250130-WA0021](https://github.com/user-attachments/assets/1df16261-3c50-4a66-ac32-d979e0f60401)

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
CREATE USER 'webaccess'@'172.31.45.61' IDENTIFIED WITH mysql_native_password BY 'Password.1';
GRANT ALL PRIVILEGES ON tooling.* TO 'webaccess'@'172.31.45.61';
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
sudo mysql -h 172.31.45.189 -u webaccess -p
```
![IMG-20250130-WA0056](https://github.com/user-attachments/assets/b3e596c8-a25a-4e8f-ac18-a52e6b3a8575)
![IMG-20250131-WA0002](https://github.com/user-attachments/assets/c0c196f5-9511-4830-a0ba-a97b805f8a86)

#### Significance:
- Ensures WordPress can connect to MySQL from a separate instance.
- If successful, the database setup is complete.

---

# Configuring Web Servers

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
sudo mount -t nfs -o rw,nosuid 172.31.42.162:/mnt/apps /var/www
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
![IMG-20250130-WA0026](https://github.com/user-attachments/assets/3cb734d6-c6a0-4b77-8479-66c4d8ca7075)

### Step 3: Make the Mount Persistent
To ensure the mount persists across reboots, update `/etc/fstab`:

```bash
sudo nano /etc/fstab
```

Add the following line:

```
172.31.42.162:/mnt/apps /var/www nfs defaults 0 0
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
Create PHP info page
```
sudo mkdir /var/www/html/
sudo nano /vr/www/html/info.php
```
Then add the following
```
<?php
phpinfo();
?>
```
Then restart Apache to apply the changes

Now access your web browser ```http://ypur-server-ip/info.php```

If everything is configured well, you should see the php info page.

![IMG-20250130-WA0055](https://github.com/user-attachments/assets/051ad8af-26a4-47af-a91b-3d243d412cc4)

## Deploying Tooling Website

### Step 1: Clone the Application Repository

```bash
sudo git clone https://github.com/Macbeth111/tooling.git
```

### Step 2: Import Database

```bash
cd tooling
sudo mysql -h 172.31.45.189 -u webaccess -p tooling < tooling-db.sql
```

### Step 3: Add Admin User to Database

```bash
sudo mysql -h 172.31.45.189 -u webaccess -p tooling
```
Inside MySQL:
```sql
INSERT INTO `users` (`username`, `password`, `email`, `user_type`, `status`)
VALUES ('myuser', MD5('password'), 'user@mail.com', 'admin', '1');
```

### Step 4: Update Database Connection in `function.php`
Next you need to go into the tooling directory to ensure that the 'html' file is listed there.
Then you copy the 'html' file to the '/var/www/' directory.
then you go to the '/var/www/html/' to access the access the functions .php file in there and update it with the private ip for your database. 
```bash
cd tooling
ls
cp -R html /var/www/
cd /var/www/html/
sudo nano function.php
```
Update:
```php
$db = mysqli_connect('172.31.45.189', 'webaccess', 'Password.1', 'tooling');
```
![IMG-20250131-WA0003](https://github.com/user-attachments/assets/b909dfac-d3f1-4352-9ccf-dec86124c3ba)

![image](https://github.com/user-attachments/assets/e2a56674-f67e-4c75-85e4-134978a7ddd6)

### Step 5: Restart Apache

```bash
sudo systemctl restart httpd
```

### Step 6: Access the Application

Visit:
```
http://<public-ip-of-webserver>/index.php
```
![IMG-20250130-WA0049](https://github.com/user-attachments/assets/39b9494d-7bd6-4d23-9545-c814fb4962fa)

Login using:
- **Username:** myuser
- **Password:** password
---
![IMG-20250130-WA0053](https://github.com/user-attachments/assets/509968b4-350f-4e7c-8d90-090c51f5e1e5)

### Lessons learned
One of the biggest challenges I faced in deploying the DevOps tooling website was configuring the storage solutions across multiple AWS EC2 instances. Setting up the NFS server required a deep understanding of block storage management, logical volume management (LVM), and network file system (NFS) configurations. Initially, I encountered issues where the EBS volumes were not properly recognized by the instances, and some partitions failed to mount due to incorrect formatting. Through extensive troubleshooting, I learned to verify volume attachments using `lsblk`, properly initialize partitions with `gdisk`, and correctly format logical volumes with XFS. Additionally, configuring the security groups to allow NFS communication across instances posed a challenge, as misconfigured rules blocked file sharing. By systematically debugging network permissions and testing with `rpcinfo` and `exportfs`, I successfully enabled seamless communication between the NFS server and web servers.

Another significant challenge was ensuring that the database and web servers communicated efficiently while maintaining security best practices. Initially, MySQL connectivity failed due to incorrect user privileges and firewall restrictions. By explicitly granting database access to the web server's IP and adjusting inbound rules to allow MySQL traffic on port 3306, I overcame this issue. Additionally, configuring PHP and Apache for seamless integration with the database required modifying configuration files and debugging connection errors within `function.php`. The experience taught me the importance of structured deployment—ensuring each component is properly set up before integrating it into the full architecture. Overall, I learned how to systematically troubleshoot infrastructure issues, optimize security while maintaining functionality, and document configurations effectively for future scalability.


---

