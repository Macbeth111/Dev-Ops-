
# Setting Up My Web Solution with WordPress

## Step 1: Preparing My Web Server

### 1. Launching My EC2 Instance
The first thing I did was launch two EC2 instances to serve as my "Web Server" and "Database" Instances. The Web Server instance is the backbone of my WordPress setup.


![created two instances](https://github.com/user-attachments/assets/65af8145-9351-4dda-b977-605f2d29df81)

### 2. Creating Three EBS Volumes
Next, I navigated to the **Volumes** section in the AWS EC2 Management Console. There, I created three Elastic Block Storage (EBS) volumes:
- Each volume was 10 GiB.
- I ensured all three volumes were in the same availability zone (AZ) as my EC2 instance.
These volumes are critical for scaling my storage needs. They’ll be used to store application files, the database, and backups separately for better performance and easier management.

---

### 3. Attaching Volumes to My Web Server
Once the volumes were created, I attached them to my EC2 instance:
- I selected each volume one by one.
- Using the **Attach Volume** option, I connected each to my EC2 instance.
- I made sure each volume was correctly mapped to a device name like `/dev/xvdf`, `/dev/xvdg`, and `/dev/xvdh`.

Attaching these volumes allowed my EC2 instance to recognize and use the additional storage needed for my WordPress site.

---

### 4. Verifying the Attached Volumes
To confirm the volumes were successfully attached, I opened the terminal on my EC2 instance and ran the `lsblk` command:
```bash
lsblk
```
This command listed all block devices attached to the server, including the three new volumes. Seeing `/dev/xvdf`, `/dev/xvdg`, and `/dev/xvdh` assured me everything was set up correctly.

---
![all three volumes attached to web server instance](https://github.com/user-attachments/assets/2c58a99d-97b0-4949-abad-bade3ac78755)

## Step 2: Configuring the Attached Volumes

### 1. Checking All Mounts and Free Space
Before configuring the new volumes, I checked the existing mounts and free space using:
```bash
df -h
```
This gave me a clear view of my current storage situation and ensured I wouldn't overwrite anything important.

---
![amount of space on my server](https://github.com/user-attachments/assets/cc3dd68f-ae8b-439d-b2a7-bc17f7c8f37b)

### 2. Partitioning the New Volumes
Next, I used the `gdisk` utility to create a single partition on each of the three disks. Here’s how I did it for the first volume (`/dev/xvdbb`):
```bash
sudo gdisk /dev/xvdbb
```
Inside the `gdisk` interface, I followed these steps:
1. Pressed `n` to create a new partition.
2. Accepted the default values for the partition number, first sector, and last sector.
3. Pressed `w` to write the partition and confirmed my changes.
![first partition on first volume](https://github.com/user-attachments/assets/c34fe526-d144-441c-b2b3-6f16bc39e13a)

I repeated these steps for the other two volumes: `/dev/xvdbc` and `/dev/xvdbd`. Partitioning ensures each volume is ready for use.

---

### 3. Verifying the New Partitions
To double-check my work, I ran the `lsblk` command again:
```bash
lsblk
```
This time, I saw the partitions as `/dev/xvdbb1`, `/dev/xvdbc1`, and `/dev/xvdbd1`, confirming that the partitioning process was successful.

---
![ALL PARTITIONS CONFIRMED](https://github.com/user-attachments/assets/d39ca5cb-1932-4800-98df-6cbca2b41713)

### 4. Installing the `lvm2` Package
I installed the `lvm2` package to manage my storage more flexibly:
```bash
sudo yum install lvm2
```
Once installed, I checked the available partitions with:
```bash
sudo lvmdiskscan
```
LVM (Logical Volume Manager) will help me group these partitions and resize them as needed in the future. This flexibility is crucial for handling WordPress's dynamic storage requirements.

---

![LVM INSTALLED AND DISK SCANNED](https://github.com/user-attachments/assets/3abbb7c1-4ed1-4db9-bb4e-be3e68957aac)

### Step 5. Setting Up Logical Volumes with LVM

With my partitions prepared and LVM installed, I proceeded to configure logical volumes for more flexible storage management. Here's what I did:

---

### 6. Mark Disks as Physical Volumes
I used the `pvcreate` utility to mark each of my three partitions as Physical Volumes (PVs) for use with LVM:
```bash
sudo pvcreate /dev/xvdbb1
sudo pvcreate /dev/xvdbc1
sudo pvcreate /dev/xvdbd1
```
*Relevance*: Marking partitions as PVs allows LVM to recognize and manage them as part of a storage pool.

---

### 7. Verify Physical Volumes
To confirm the creation of Physical Volumes, I ran:
```bash
sudo pvs
```
The output displayed all three partitions as PVs, showing that they were ready for the next step.

---
![PVCREATED AND CHECKED](https://github.com/user-attachments/assets/1b319c00-aa7a-47e1-991a-d84e6c35b271)

### 8. Create a Volume Group
Using the `vgcreate` command, I combined all three PVs into a single Volume Group (VG) named `webdata-vg`:
```bash
sudo vgcreate webdata-vg /dev/xvdbb1 /dev/xvdbc1 /dev/xvdbd1
```

*Relevance*: A Volume Group pools together the storage from multiple physical volumes, enabling me to create logical volumes from the combined space.

---

### 9. Verify the Volume Group
To ensure the VG was created successfully, I checked it with:
```bash
sudo vgs
```
The output confirmed that `webdata-vg` was created with all three PVs included.

---
![VG created](https://github.com/user-attachments/assets/48eb3490-4544-43c0-b5ca-7c700770cc89)

### 10. Create Logical Volumes
I used the `lvcreate` utility to create two logical volumes:
- `apps-lv`: This will store data for the WordPress website and used half of the available space.
- `logs-lv`: This will store logs and used the remaining space.

Here are the commands I used:
```bash
sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg
```

*Relevance*: Logical Volumes allow me to allocate specific amounts of storage for different purposes, making the setup more organized and scalable.

---
### 11. Verify Logical Volumes
To confirm the logical volumes were created, I ran:
```bash
sudo lvs
```
The output showed `apps-lv` and `logs-lv` as part of the `webdata-vg` Volume Group.

---
![LOGICAL VOLUMES CREATED](https://github.com/user-attachments/assets/34713389-e4c6-4f3c-b440-b70fb7fc2f2b)

### 12. Verify the Entire Setup
Finally, I ran the following command to check the complete LVM setup, including VGs, PVs, and LVs:
```bash
sudo lsblk            # To view block device structure
```
Everything was correctly configured, and the setup was ready for formatting and mounting the logical volumes.

---
In the next steps, I’ll format the logical volumes, mount them to the appropriate directories, and begin installing and configuring WordPress. Stay tuned!

![VOLUME SETUP VERIFIED](https://github.com/user-attachments/assets/056264af-5bfe-42bf-993e-afdbdca7f8a1)

## Step 4: Formatting and Mounting Logical Volumes

With the logical volumes created, I proceeded to format them, set up directories, and mount the volumes to their respective locations.

---

### 13. Format Logical Volumes with ext4 Filesystem
To make the logical volumes usable, I formatted them with the ext4 filesystem:
```bash
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```

*Relevance*: Formatting prepares the logical volumes to store data, ensuring compatibility with the Linux file system.

---
![LOGICAL VOLUMES FORMATTED](https://github.com/user-attachments/assets/70cec4f9-d3e5-49d3-9e01-3d06fbbfcbb6)

### 14. Create Required Directories
I created directories to store website files and backup logs:
- For website files: `/var/www/html`
- For backup of log data: `/home/recovery/logs`

Commands used:
```bash
sudo mkdir -p /var/www/html
sudo mkdir -p /home/recovery/logs
```

*Relevance*: These directories ensure that the application files and logs are organized and stored in appropriate locations.

---

### 15. Mount Logical Volumes
I mounted the logical volumes to their respective directories:
- `apps-lv` to `/var/www/html`:
  ```bash
  sudo mount /dev/webdata-vg/apps-lv /var/www/html
  ```

- `logs-lv` to `/var/log`:
  Before mounting `/var/log`, I backed up its current contents as mounting would erase existing data.

---

### 16. Backup and Restore Log Files
Using the `rsync` utility, I backed up files from `/var/log` to `/home/recovery/logs`:
```bash
sudo rsync -av /var/log/ /home/recovery/logs/
```

I then mounted `logs-lv` to `/var/log`:
```bash
sudo mount /dev/webdata-vg/logs-lv /var/log
```

Finally, I restored the log files back to `/var/log`:
```bash
sudo rsync -av /home/recovery/logs/ /var/log
```

*Relevance*: This step ensures no data is lost during the transition and that the logs directory uses the new logical volume.

---

### 17. Update `/etc/fstab` for Persistent Mounting
To make the mount configuration persist after a system restart, I updated the `/etc/fstab` file using the UUIDs of the devices.

#### Get the UUIDs of Devices
I retrieved the UUIDs by running:
```bash
sudo blkid
```
# /etc/fstab
UUID=d726071f-b770-48f2-8f6f-3016a0245f0f  /boot           ext4    defaults        0 2
UUID=bfe15554-f93e-4a44-847b-29cd914e01ab  /               ext4    defaults        0 1
UUID=0DD3-9DEC                             /boot/efi       vfat    defaults        0 1
UUID=d235623a-f380-41da-8045-647c6c484bf8  /var/log        ext4    defaults        0 2
UUID=f624683e-6e88-4ed0-bfb9-2709b377ac6c  /apps           ext4    defaults        0 2

#### Edit the `/etc/fstab` File
I added the following entries to the file using the `vi` editor:
```bash
sudo vi /etc/fstab
```
![Screenshot 2024-12-20 234418](https://github.com/user-attachments/assets/a8e2e831-18bb-4d71-a6f4-8d5ee373d625)

```
*Relevance*: Updating `/etc/fstab` ensures that the logical volumes are automatically mounted to their directories after each reboot.
---
After this I tested the configuration and reloaded daemon using the commands
```bash
sudo mount -a
sudo system ctl daemon-reload
```
Then I proceeded to verify the set up again using
```bash
df -h
```
![VAR AND HTML UPDATED SUCCESSFULLY](https://github.com/user-attachments/assets/7f77c5ca-cc40-4041-aae6-4549926232c7)

NB: After this I repeated the whole process for my "Database Instance". However when I got to creating the logical volumes
instead of ``` apps-lv``` I created ```db-lv``` using the command
```bash
sudo lvcreate -n db-lv -L 14G webdata-vg
```
For direcotories, i created ```/db```
then I mounted the ```db-lv``` onto the ```/db```
```
![db-logical volumes created](https://github.com/user-attachments/assets/e783e4b4-93a5-441d-902f-209eef4d9dd0)
```
### Step 18 — Installing Wordpress on my Ec2 Web-server

To ensure my server has the latest software packages, I started by updating the repository:

```bash
sudo apt -y update
```

#### Why This Step is Important:

Updating the repository ensures that all installed packages and dependencies are up-to-date. This reduces potential compatibility issues and vulnerabilities.

---

### Step 19 — Install wget, Apache, and Its Dependencies

Next, I install Apache (a web server) along with `wget` and other required dependencies:

```bash
sudo apt -y install wget apache2 php php-mysqlnd php-fpm php-json
```
![apache1](https://github.com/user-attachments/assets/2560cc79-0481-44c8-a23d-05394873bcf2)

#### Why This Step is Important:

- **Apache**: Serves the WordPress site to users.
- **PHP and Dependencies**: WordPress relies on PHP to function.
- **wget**: Enables downloading files directly from the terminal, useful for obtaining WordPress later.

---

### Step 20 — Start Apache

Once Apache is installed, I enable and start the service:

```bash
sudo systemctl enable apache2
sudo systemctl start apache2
```
![apache2](https://github.com/user-attachments/assets/7351e048-dc5e-4327-b773-e6fc700d5243)

#### Why This Step is Important:

Starting Apache ensures that the web server is running and ready to serve content. Enabling it ensures that the service starts automatically after a reboot.

---

### Step 21 — Install PHP and Its Dependencies

To fully support WordPress, I install additional PHP modules and enable the PHP environment:

```bash
sudo apt install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo apt install yum-utils https://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo apt module list php
sudo apt module reset php
sudo apt module enable php:remi-7.4
sudo apt install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
```

#### Why This Step is Important:

- **Additional PHP Modules**: These modules expand PHP’s capabilities to handle image processing, database connections, and performance optimizations.
- **php-fpm**: Enhances the performance of PHP by reducing resource usage.
- **SELinux Configuration**: Adjusts security settings to ensure Apache and PHP can function together without restrictions.

---

### Step 22 — Restart Apache

To apply the new configurations, I restart Apache and check its status:

```bash
sudo systemctl restart apache2
sudo systemctl status apache2
```
![apache3](https://github.com/user-attachments/assets/9cf4ebe9-3840-48d5-9fad-56ba7e33df51)

#### Why This Step is Important:

Restarting Apache ensures that all changes and new configurations are applied properly.

---

### Step 23 — Download and Copy WordPress to /var/www/html

To set up WordPress, I first create a directory and download the latest WordPress package:

```bash
mkdir wordpress
cd wordpress
sudo wget http://wordpress.org/latest.tar.gz
sudo tar -xvf latest.tar.gz
sudo rm -rf latest.tar.gz
sudo cp -R wordpress/* /var/www/html/
```
![apache4](https://github.com/user-attachments/assets/154dcd88-ffe6-45e5-9450-a0fbc19e6600)

#### Why This Step is Important:

- **Download WordPress**: Ensures I have the latest version.
- **Extract and Copy Files**: Moves the WordPress files to the correct directory where Apache can serve them.

---

### Step 24 — Configure SELinux Policies

To configure SELinux policies for WordPress, I used:

```bash
sudo chown -R apache:apache /var/www/html/wordpress
sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
setsebool -P httpd_can_network_connect 1
```
![part where i changed apache to www-data](https://github.com/user-attachments/assets/85031b15-3abe-458f-bd5e-9bfbba3a7256)

#### Why This Step is Important:

1. **Ownership Permissions**: The `chown` command assigns ownership of the WordPress files to the Apache user. This ensures that Apache has the proper rights to serve and modify the WordPress content when necessary.

2. **Security Context**: The `chcon` command changes the SELinux context to allow Apache to read and write content within the specified directory. Without this, SELinux might block Apache from functioning correctly.

3. **Network Connectivity**: The `setsebool` command allows Apache to establish network connections, which is crucial for tasks such as downloading plugins, themes, or updates directly from the WordPress admin panel.

---

### Step 25 — Restart Apache Again

To apply all changes, I restart Apache one more time:

```bash
sudo systemctl restart apache2
```

#### Why This Step is Important:

- Restarting Apache ensures that all the newly configured settings and policies take effect.

---

# Step 27 — Install MySQL on My DB Server EC2

I ensured that my DB server is set up with MySQL for database management.

### Steps:
1. Update the repository:
   ```bash
   sudo apt update
   ```
2. Install the MySQL server:
   ```bash
   sudo apt install mysql-server
   ```
3. Verify that the service is up and running:
   ```bash
   sudo systemctl restart mysql
   sudo systemctl enable mysql
   sudo systemctl status mysql
   ```
   ![mysql1](https://github.com/user-attachments/assets/1c64357c-22d1-4344-ba55-3970fdebe7df)

### Importance:
- **Updating the repository** ensures I have the latest software packages.
- **Installing MySQL** provides the database server necessary for WordPress to store its data.
- **Enabling the service** ensures the database server remains operational, even after system reboots.

---

# Step 19 — Configure the Database to Work with WordPress

Next, I configure my database to allow WordPress to interact with it.

### Steps:
1. Log in to MySQL:
   ```bash
   sudo mysql
   ```
2. Create a database for WordPress:
   ```sql
   CREATE DATABASE wordpress;
   ```
3. Create a user and grant permissions:
   ```sql
   CREATE USER 'wordpressuser'@'172.31.20.34' IDENTIFIED BY 'Password.1';
   GRANT ALL ON wordpress.* TO 'wordpressuser'@'172.31.20.34';
   FLUSH PRIVILEGES;
   ```
   ![mysql2](https://github.com/user-attachments/assets/821ad60c-866c-431e-be31-3e9a5ea6e50d)

4. Verify the database creation:
   ```sql
   SHOW DATABASES;
   ```
   ![MYSQL4](https://github.com/user-attachments/assets/11f7e586-ad77-46c4-972e-5930e17333b7)

5. Exit MySQL:
   ```bash
   exit
   ```

### Importance:
- **Creating the database** provides a dedicated storage space for WordPress content.
- **Granting permissions** ensures that WordPress can access and manage the database securely.
- **Flushing privileges** updates MySQL's internal permissions table to reflect the changes.

---

# Step 20 — Configure WordPress to Connect to a Remote Database

Now, I need to connect WordPress to the MySQL database hosted on my DB Server EC2 instance.

### Steps:
1. Open MySQL port 3306 on my DB Server EC2:
   - For security, I ensure that only my Web Server’s IP address can access this port.
   - I configure the inbound rule in the EC2 security group, specifying the source as `/32` (single IP address):
   
![db-inbound rule updated](https://github.com/user-attachments/assets/35e1b425-75bd-4731-a47c-d5559ffd548e)


2. Test the connection from the Web Server to the DB Server using the `mysql-client`.

![image](https://github.com/user-attachments/assets/1a280680-0b41-487f-b673-89d430fd5a5e)

### Importance:
- **Opening the MySQL port** allows the Web Server to communicate with the DB Server.
- **Restricting access** enhances security by preventing unauthorized connections.
- **Testing the connection** ensures that WordPress can successfully connect to the database.

---

Here’s the continuation and completion of the notes based on the new content provided:

---
# Step 21 — Ensuring Apache Can Use WordPress

To finalize the setup, I ensure that Apache is properly configured to use WordPress.

### Steps:

1. **Change permissions and ownership for WordPress files**:
   I ensure that Apache has access to the WordPress files:
   ```bash
   sudo chown -R apache:apache /var/www/html/wordpress
   sudo chmod -R 755 /var/www/html/wordpress
   ```

2. **Enable SELinux policies**:
   To allow Apache to serve WordPress content and handle file uploads:
   ```bash
   sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
   sudo setsebool -P httpd_can_network_connect 1
   ```

3. **Restart Apache**:
   I restart the Apache server to apply the changes:
   ```bash
   sudo systemctl restart apache2
   ```

### Importance:
- **Changing ownership and permissions** ensures Apache has the necessary access to serve WordPress files securely.
- **Configuring SELinux policies** is essential for handling WordPress operations, including file uploads and database communication.
- **Restarting Apache** applies the changes, ensuring they take effect immediately.

---

# Step 22 — Enable Web Access to the WordPress Site

## Editing Inbound Rules
To allow external access to my server, I edited the inbound rules of my EC2 instance. 
I ensured that traffic on port 80, which is used for HTTP, was permitted. This is crucial for allowing users to access the WordPress site via their browsers.

![edited inbound rules](https://github.com/user-attachments/assets/ed4a674b-bf15-47eb-84b0-e44979258c16)

## Creating a Virtual Host Configuration
Next, I created a virtual host configuration file specifically for WordPress. I saved it in ```/etc/apache2/sites-available/wordpress.conf```. This configuration file defines how Apache handles requests for my WordPress site. By isolating this configuration, it becomes easier to manage settings and troubleshoot issues related to the site.

![Screenshot 2024-12-22 191244](https://github.com/user-attachments/assets/ff1ba04e-f93d-4e38-9190-805c54250c27)

![Screenshot 2024-12-22 175039](https://github.com/user-attachments/assets/5ffe7872-75cb-44d3-9c62-ac6210fc2531)

This setup ensures that requests to the server on port 80 are directed to the WordPress directory and that Apache can process `.htaccess` rules.

## Enabling the Site Configuration
After creating the virtual host configuration, I enabled it using the following commands:
```bash
sudo a2ensite wordpress.conf
sudo systemctl reload apache2
```
![Screenshot 2024-12-22 175039](https://github.com/user-attachments/assets/7d21cf06-1a09-4485-9847-c74ef973a6b8)

Enabling the site configuration tells Apache to include the new settings when handling incoming requests. Reloading Apache applies these changes without interrupting ongoing connections.

## Updating Database Details
To connect WordPress to the database, I updated the `wp-config.php` file located in the WordPress directory using the command
```
bash
sudo nano /var/www/html/wordpress/wp-config.php
```
I added the following lines to define the database name, user, password, and host:

![image](https://github.com/user-attachments/assets/1c664303-a045-4587-929d-8245e8ba5054)

This configuration ensures WordPress can access and manage its database, which is essential for storing content, user data, and site settings.

## Restarting Apache
Finally, I restarted Apache using:
```bash
sudo systemctl restart apache2
```
Restarting Apache is necessary to fully apply all the configurations, ensuring that the web server operates with the latest settings and connections.

---

Then I connected to wordpress using my ip ```http://35.162.233.145```

![wordpress configured](https://github.com/user-attachments/assets/4bea770b-d973-4261-9dc5-e146bf46ed45)

I entered the database details that I updated in the system


![wordpress successful](https://github.com/user-attachments/assets/1a5f3a2a-be7f-45f8-982c-8d1b6dcc8910)

Finally I was successful

![WORDPRESS FINAL](https://github.com/user-attachments/assets/f02095f3-0a8b-495d-a9f1-a4ed95449b80)

```markdown
# Challenges Faced and Resolutions in Setting Up WordPress Web Solution

## Challenges Encountered
During the setup of the web solution using WordPress, several challenges were encountered. Firstly, while attaching the Elastic Block Storage (EBS) volumes to the EC2 instances, mapping the correct device names (`/dev/xvdf`, `/dev/xvdg`, etc.) to the appropriate volumes was confusing and time-consuming. Missteps in this process led to failed recognitions by the EC2 instance during the initial configuration stages. Another significant hurdle arose while creating logical volumes using the `lvm2` package. Incorrect partitioning of the disks and minor syntax errors in the Logical Volume Manager (LVM) commands caused delays and required additional troubleshooting. Furthermore, formatting and mounting logical volumes posed challenges as the ext4 filesystem setup needed to be verified multiple times to ensure correctness. Lastly, while automating the mounting of volumes using the `/etc/fstab` file, there were instances of invalid UUID entries that resulted in mounting failures after system reboots.

## Resolutions Implemented
To address these challenges, a systematic approach was adopted. During volume attachment, careful documentation of device names and their respective EBS volumes ensured accurate mapping, while the use of `lsblk` and `df -h` commands verified successful attachments. For the LVM setup, repetitive errors were resolved by meticulously following LVM guidelines, and commands were validated using tools such as `pvscan`, `vgscan`, and `lvscan`. Logical volume formatting and mounting were streamlined by referencing filesystem compatibility issues and verifying configurations with `blkid` and `mount` commands. The `/etc/fstab` file issue was resolved by retrieving and cross-checking UUIDs through `sudo blkid`, and configurations were tested using `sudo mount -a` before restarting the system. These resolutions ensured the smooth completion of the setup and highlighted the importance of careful planning and validation at each step.
```
