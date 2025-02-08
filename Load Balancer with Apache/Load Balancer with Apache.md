# Load Balancer Solution with Apache

## Table of Contents

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Project Objectives](#project-objectives)
- [Implementation Steps](#implementation-steps)
  - [Step 1: Provision EC2 Instance for Load Balancer](#step-1-provision-ec2-instance-for-load-balancer)
  - [Step 2: Open Necessary Ports](#step-2-open-necessary-ports)
  - [Step 3: Install Apache and Configure as a Load Balancer](#step-3-install-apache-and-configure-as-a-load-balancer)
  - [Step 4: Configure Load Balancing](#step-4-configure-load-balancing)
  - [Step 5: Verify Load Balancer Configuration](#step-5-verify-load-balancer-configuration)
- [Challenges and Solutions](#challenges-and-solutions)
---

## Introduction

In this project, I implemented a load balancing solution using Apache to extend the DevOps Tooling Website Solution. The purpose of this implementation is to evenly distribute traffic among the web servers, enhancing the scalability and reliability of the application.

By completing this project, I have gained hands-on experience in configuring a load balancer and understanding its significance in a production environment.

---

## Prerequisites

Before beginning, I ensured the following prerequisites were met:

- **Infrastructure Setup:**
  - NFS Server
  - Database Server
  - At least two Web Servers

- **Knowledge Requirements:**
  - Basic understanding of Linux systems and command line
  - Familiarity with Apache web server
  - Access to an AWS account

---

## Project Objectives

The main objectives of this project were:

- Implement a load balancing solution using Apache
- Ensure even distribution of traffic across all web servers
- Improve the overall scalability and reliability of the DevOps Tooling Website
- Gain practical experience in configuring and managing a load balancer

---

## Implementation Steps

### Step 1: Provision EC2 Instance for Load Balancer

To create an EC2 instance for the load balancer, I:

1. Logged in to AWS Management Console
2. Navigated to EC2 and launched an instance
3. Selected **Ubuntu Server 20.04 LTS** as the OS
4. Chose **t2.micro** instance type (sufficient for this project)
5. Configured the instance to:
   - Use the same VPC as my web servers
   - Be in a public subnet
6. Named the instance **Load-Balancer Intance**
7. Configured security settings (to be set up in the next step)
8. Launched the instance
![Load Balancer Instance launched](https://github.com/user-attachments/assets/41755acc-c0e5-493f-9828-a4bb60361a0b)

### Lessons Learned:
Proper instance selection and networking setup ensure smooth integration with web servers.

---

### Step 2: Open Necessary Ports

To allow traffic, I updated the security group:

1. In the EC2 Dashboard, navigated to **Security Groups**
2. Edited inbound rules to allow:
   - **HTTP (port 80) from anywhere (0.0.0.0/0)**
   - **SSH (port 22)**
![ports 22 and 80 opened on load balancer instance](https://github.com/user-attachments/assets/f3c8a113-adc3-449f-b647-cf47945d2cb0)

### Lessons Learned:
Ensuring correct security group configurations is crucial to enabling communication between instances.

---

### Step 3: Install Apache and Configure as a Load Balancer

1. Connected to the instance via SSH:
   ```sh
   ssh -i <your-private-key.pem> ubuntu@<your-instance-public-ip>
   ```
2. Updated the package list:
   ```sh
   sudo apt update && sudo apt upgrade -y
   ```
3. Installed Apache and required modules:
   ```sh
   sudo apt install apache2 -y
   sudo apt install libxml2-dev -y
   ```
4. Enabled necessary Apache modules:
   ```sh
   sudo a2enmod rewrite proxy proxy_balancer proxy_http headers lbmethod_bytraffic
   ```
5. Restarted Apache and check status:
   ```sh
   sudo systemctl restart apache2
   sudo systemctl status apache2
   ```
![Apache installed and running](https://github.com/user-attachments/assets/6c1007e8-af78-49af-947b-a8b3f2cd9d2d)

### Lessons Learned:
Apache provides robust load balancing features through its proxy modules.

---

### Step 4: Configure Load Balancing

1. Edited Apache's configuration file:
   ```sh
   sudo nano /etc/apache2/sites-available/000-default.conf
   ```
2. Configured the load balancer:
   ```sh
   <Proxy "balancer://mycluster">
       BalancerMember http://<private-IP-Web1>:80 loadfactor=5 timeout=1
       BalancerMember http://<private-IP-Web2>:80 loadfactor=5 timeout=1
       BalancerMember http://<private-IP-Web3>:80 loadfactor=5 timeout=1
       ProxySet lbmethod=bytraffic
   </Proxy>
   ProxyPass / balancer://mycluster/
   ProxyPassReverse / balancer://mycluster/
   ```
   Replace '<private-IP-Web1>' with the private IPs of web servers and then restart Apache

   ![Private IP of webservers inputed](https://github.com/user-attachments/assets/17b7d8b6-d52a-4493-8306-33df0124b924)

4. Restarted Apache:
   ```sh
   sudo systemctl restart apache2
   ```

### Lessons Learned:
Apache provides multiple load balancing methods, and selecting the right one depends on the application’s traffic pattern.

### 5 Configure Webserver to receive traffic from Load Balancer
The webservers need to be configured to allow traffic from load balancer security group.
Open port 80 on webserver and replace the source with load balancer security group. 
(you will need to access security groups to do this)


![Port 80 open on webserver and source load balancer security group](https://github.com/user-attachments/assets/ec627453-4aef-4b98-8d7b-9b6c1f7f5b43)

---

### Step 5: Verify Load Balancer Configuration

1. Opened the load balancer's public IP in a browser
2. Refreshed the page multiple times to confirm traffic distribution, thus ensure that the
   load balancer IP actually allows you to access the steghub tooling site.

![webpage accessed via load balancer](https://github.com/user-attachments/assets/e5c0f37d-c76f-4196-a81a-59409df1bbc0)

4. Check webserver Apache logs to verify traffic distribution:
   ```sh
   sudo tail -f /var/log/apache2/access.log
   ```
![Access logs checked for web server](https://github.com/user-attachments/assets/f9152842-9937-42f2-a7fa-214bb68fc7ee)

Explanation to access logs:

Logs with 302 Status:

    Redirection: The server is redirecting the client to another resource (possibly a login or main page).

Repeated Access to /login.php:

    Multiple successful requests (200 status) suggest users or scripts frequently access the login page.

### Lessons Learned:
Testing is essential to ensure proper load balancing functionality.

---
### Challenges Faced and Solutions
During the implementation of the Apache load balancer, several challenges were encountered and addressed effectively. 
Networking and instance configuration required careful setup to ensure the load balancer and web servers were in the same VPC and subnet, 
and security groups were configured to allow HTTP (port 80) and SSH (port 22) traffic while restricting web server access to the load balancer's security group. 
Installing Apache and its modules posed challenges due to potential missing dependencies, which were resolved by enabling required modules like proxy, 
proxy_balancer, proxy_http, and lbmethod_bytraffic, followed by restarting the Apache service to apply changes. Configuring the load balancer involved defining
a proxy cluster (balancer://mycluster) in the Apache configuration file, specifying web server private IPs, and using the bytraffic method to distribute traffic 
evenly based on data volume, which was tested iteratively for correctness. Security concerns were mitigated by restricting web servers to only accept traffic from
the load balancer's security group, ensuring a secure setup. Verification involved accessing the load balancer’s public IP in a browser to test traffic distribution and 
inspecting web server logs to confirm even routing of requests. Debugging during testing helped identify and resolve issues such as missing resources (e.g., favicon.ico) and 
redirection errors (e.g., HTTP 302 status for login pages). These steps highlighted the importance of meticulous configuration, robust testing, and leveraging Apache’s load 
balancing features to create a scalable, secure, and reliable solution.
