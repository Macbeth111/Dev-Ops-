# Setting Up a Client-Server Architecture with MySQL

This guide outlines the step-by-step process of setting up a client-server architecture using MySQL. Each step highlights its importance and provides instructions in first person to help you follow along.

---

## Step 1: Launch Two Linux-Based Virtual Servers

First, I create and configure two Linux-based virtual servers. I name them:
- `mysql-server` (Server A)
- `mysql-client` (Server B)
![Two Linux servers launched](https://github.com/user-attachments/assets/dc7df6fc-6550-453d-b2fa-7d99b01301a1)

**Why is this important?**
This separation allows me to implement a dedicated MySQL database server while keeping the client machine distinct. It mirrors real-world client-server architecture, enabling efficient communication and secure data handling.

---

## Step 2: Install MySQL Server on `mysql-server`

On the `mysql-server` instance, I install the MySQL Server software:
```bash
sudo apt update
sudo apt install mysql-server -y
```

**Why is this important?**
The MySQL Server is the backbone of the architecture. It stores, processes, and manages the data that the client interacts with.

---

## Step 3: Secure the MySQL Server Installation

After installing the MySQL server, I run the following command to secure the installation:
```bash
sudo mysql_secure_installation
```
Then I went ahead to set root password for Mysql using
```bash
sudo mysql -u root -p
```
![mysql installed and root password set](https://github.com/user-attachments/assets/6a06a39b-3b0c-46c2-b510-59b30f89ad97)

**Why is this important?**
This step enhances security by setting a root password, removing test databases, and disabling remote root access. These measures ensure my server is not vulnerable to unauthorized access.

---

## Step 4: Install MySQL Client on `mysql-client`

On the `mysql-client` instance, I install the MySQL Client software:
```bash
sudo apt update
sudo apt install mysql-client -y
```

**Why is this important?**
The MySQL Client allows me to connect to the MySQL Server from a remote machine. It acts as the interface through which I interact with the database.

---

## Step 5: Configure the MySQL Server to Allow Remote Connections

I edit the MySQL configuration file on `mysql-server`:
```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```
I locate the line:
```
bind-address = 127.0.0.1
```
and change it to:
```
bind-address = 0.0.0.0
```
Then, I restart the MySQL service:
```bash
sudo systemctl restart mysql
```
![127 replace use](https://github.com/user-attachments/assets/8d1909fa-4e82-4d9e-960c-2179f8e7b35d)

**Why is this important?**
By default, MySQL only listens for connections on `localhost`. Changing the bind address allows remote machines to connect to the server.

---

## Step 6: Create a Dedicated User for Remote Access

I log into the MySQL server:
```bash
sudo mysql
```
I create a new user with remote access privileges:
```sql
CREATE USER 'macbeth'@'%' IDENTIFIED BY 'Leticia5';
GRANT ALL PRIVILEGES ON *.* TO 'macbeth'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```
![user macbeth created](https://github.com/user-attachments/assets/00c8f910-186c-45e5-98b2-f1c049f44e84)

**Why is this important?**
Creating a dedicated user for remote access ensures secure and controlled interaction with the database. Using `%` as the host allows connections from any IP address.

---

## Step 7: Configure Security Group Rules

In my cloud platform, I modify the security group for `mysql-server` to allow incoming traffic on port `3306` from the private IP address of `mysql-client`.
![inbound rules edited](https://github.com/user-attachments/assets/04b094c1-bd76-4abe-9ecf-471fc5270ee7)

**Why is this important?**
Port `3306` is the default port for MySQL. Allowing access ensures that `mysql-client` can communicate with the server while preventing unauthorized access.

---

## Step 8: Test the Connection

From `mysql-client`, I test the connection:
```bash
mysql -u macbeth -p -h <mysql-server-private-ip>
```
I enter the password when prompted.
![Screenshot 2024-12-11 154758](https://github.com/user-attachments/assets/3df42406-56aa-4c57-8cd3-79f279d462d5)

**Why is this important?**
Testing verifies that the configuration is correct and ensures the client-server communication works as expected.

---

Following these steps, I successfully set up a client-server architecture using MySQL, enabling remote database management and secure interaction between the client and server.

# Lessons Learned and Challenges Faced

## Lessons Learned
During the setup of the MySQL client-server architecture, I gained a deeper understanding of how client-server communication works in a database environment. 
I learned how to configure two separate Linux-based virtual servers for specific roles—one as a MySQL server and the other as a MySQL client—and the importance of 
security configurations such as managing inbound rules in security groups. 
This experience reinforced the significance of setting appropriate user privileges and securely connecting to the database using the MySQL utility. 
Additionally, I now appreciate how IP configurations enable controlled access between the client and server, highlighting the balance between accessibility and security in cloud environments.

## Challenges Faced and Solutions
One major challenge I faced was ensuring that the client could connect to the server. 
Initially, there were connectivity issues because the MySQL server's inbound rules were not configured to allow the client’s IP address. 
To resolve this, I updated the security group settings to permit traffic on port 3306 only from the client's IP address, ensuring a secure connection. 
Another difficulty was an error in granting privileges to the user due to incorrect SQL syntax. I overcame this by carefully reviewing the MySQL documentation to ensure proper command usage. 
These challenges underscored the importance of attention to detail and leveraging resources like documentation and community forums to troubleshoot effectively.

