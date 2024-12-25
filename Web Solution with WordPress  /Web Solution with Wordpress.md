```markdown
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
![db-logical volumes created](https://github.com/user-attachments/assets/ac8bed8e-e46e-4f0f-98d1-e2c64b0f21ca)


