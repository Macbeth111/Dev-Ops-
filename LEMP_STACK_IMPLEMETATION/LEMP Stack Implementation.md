# LEMP Stack Implementation Guide

## Table of Contents
1. [Prerequisite](#prerequisite)
2. [Step 1: Install the Nginx Web Server](#step-1-install-the-nginx-web-server)
3. [Step 2: Install MySQL Server](#step-2-install-mysql-server)
4. [Step 3: Install PHP Server](#step-3-install-php-server)
5. [Step 4: Configure Nginx to Use PHP Processor](#step-4-configure-nginx-to-use-php-processor)
6. [Step 5: Test PHP with Nginx](#step-5-test-php-with-nginx)
7. [Step 6: Retrieve Data from MySQL Database with PHP](#step-6-retrieve-data-from-mysql-database-with-php)
8. [Conclusion: Learnings and Challenges](#conclusion-learnings-and-challenges)

## Prerequisite
To connect to an EC2 instance via Git Bash, follow these steps:

1. **Access the EC2 instance**: Select the EC2 instance from the AWS Management Console, click on the "Connect" button, and choose the "SSH Client" option.
2. **Run the SSH command**: Use the provided sample SSH command in Git Bash:
   ```bash
   ssh -i "your-key-pair-name.pem" ubuntu@ec2-172-31-17-208.us-west-2.compute.amazonaws.com
![Image](https://github.com/user-attachments/assets/a93e4fef-17c4-4e85-8896-d9c1f3e3d52f)

---

## Step 1: Install the Nginx Web Server

Initially, I attempted to install the Nginx web server. The first step in installing any new software on a Linux machine involved updating the system’s package list to ensure the most up-to-date versions of the required packages were installed.

```bash
sudo apt update
sudo apt install nginx
```

Unfortunately, the Nginx installation failed. To troubleshoot, I conducted a connectivity test by running the following command:

```bash
ping google.com
```

The result indicated a 100% packet loss, suggesting that the instance was not receiving traffic. Recognizing this as a networking issue, I embarked on problem-solving measures to rectify the situation.

### Ensuring EC2 Was Receiving Traffic

I checked if the EC2 instance was in a public subnet with proper internet access.

- **Verified the Public Subnet:**
  - Navigated to the VPC Dashboard in AWS.
  - Checked the subnet where the instance was located, ensuring it had a Route Table with a route to the Internet Gateway (IGW).
  - Verified that the route table included a route directing `0.0.0.0/0` traffic to the Internet Gateway (IGW).

- **Checked the Internet Gateway:**
  - In the VPC Dashboard, went to Internet Gateways.
  - Confirmed that the Internet Gateway was attached to the VPC where the instance resided.

- **Elastic IP Allocation**
  - Realized that the EC2 Connect key was using the private IP address, so the instance needed an Elastic IP assigned for proper internet access.
  - Went to Elastic IPs under the EC2 Dashboard.
  - Clicked on Allocate Elastic IP Address and followed the steps to allocate one.
  - Associated the Elastic IP with the EC2 instance.
![Image](https://github.com/user-attachments/assets/dedf0505-3b2c-4c5f-80f5-70803ff8dbe5)

- **Reason for Elastic IP:** Without a public IP, the EC2 Connect key was only using the private IP, which meant that traffic couldn’t reach the instance from outside the VPC.

- **Configured Security Group and Network ACL:**
  - Checked outbound rules in Security Groups to confirm that they allowed all traffic to `0.0.0.0/0` on all ports.
  - Verified Network Access Control List (NACL) for the subnet where the instance was located to ensure inbound and outbound rules allowed traffic from `0.0.0.0/0` for all ports.

- **Testing Network Connectivity:** After configuring the subnet and assigning the Elastic IP, I tested connectivity by running:

```bash
ping google.com
```
![Image](https://github.com/user-attachments/assets/521f26e2-a558-4e06-b073-385a56c01228)


### Installing Nginx Server

After confirming network connectivity, I ran the following commands to install Nginx and ensure it was active and running:

```bash
sudo apt update
sudo apt install nginx
sudo systemctl status nginx
```
![Image](https://github.com/user-attachments/assets/5080abe3-eebd-4e2e-abb9-b21f77fa3775)


Finally, I checked if the Nginx server could respond to requests from the internet using the public IP address:

```bash
https://35.87.151.65:80/
```
![Image](https://github.com/user-attachments/assets/8a91f174-7b50-447e-8586-41c3c08b6190)

### Personal Notes:
- Key actions included verifying network settings and correctly configuring the Elastic IP.
- I learned the importance of public IP addresses for server accessibility and the need for proper security group configurations.
- The decision to use Nginx as the web server was based on its high-performance capabilities and efficiency in handling large amounts of concurrent connections, making it a better choice than alternatives like Apache for this deployment. Nginx's lightweight design ensures low memory usage and scalability, ideal for a web hosting environment. Installing Nginx through Ubuntu’s package manager ensures access to the most up-to-date and reliable version. Additionally, assigning an Elastic IP to the EC2 instance ensures a static, persistent IP address, necessary for reliable web server access, especially after reboots where instances might lose their default public IP.
- When installation issues arose, a systematic troubleshooting approach was used to diagnose the connectivity problem. After verifying that the instance lacked internet access, steps were taken to check the VPC configuration, including the subnet’s public status and its routing through an Internet Gateway. Assigning an Elastic IP and confirming the security group’s outbound rules and NACLs allowed for proper communication with external networks. This process emphasized the importance of correctly configuring AWS networking components to ensure the server was accessible over the internet.
---

## Step 2: Install MySQL Server

I followed a structured process for installing MySQL on a Linux-based system. The installation began with:

```bash
sudo apt install mysql-server
```
Here, sudo gives me administrator privileges to install software, apt is the package manager used by Debian-based systems (like Ubuntu), and mysql-server is the actual package I'm installing. When prompted, I confirm the installation by typing y and pressing ENTER. After confirming the installation, I logged into the MySQL console for configuration:

```bash
sudo mysql
```
This command starts the MySQL client. Since I use sudo, I’m logging in as the root user (the administrative user for MySQL), giving me full access to the server’s management interface.

### Setting MySQL Root User Password

To secure the MySQL installation, I set a password for the root user with the following command:

```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';
```
![Image](https://github.com/user-attachments/assets/7a4052b5-4169-4603-8e97-e719d38f7ffe)

This command changes the password for the root user (root), who can connect only from the local machine (localhost). The password is set to 'PassWord.1'. It's important to set a strong password at this stage to prevent unauthorized access to the server. After setting the root password, I exited the MySQL console:

```sql
mysql> exit
```

### Running the MySQL Secure Installation Script

I ran the secure installation script to enhance security:

```bash
sudo mysql_secure_installation
```
![Image](https://github.com/user-attachments/assets/84a3377e-a045-4473-bf20-aedff3bd505d)

During this process, I chose whether to enable the VALIDATE PASSWORD PLUGIN, which enforces stronger passwords.

### Selecting the Level of Password Validation


I was then prompted to choose the level of password validation, with options being LOW, MEDIUM, and STRONG.

### Setting and Confirming Root User Password

I was required to set and confirm the password for the MySQL root user, ensuring strong security measures were in place. Setting a strong password for the MySQL root user ensures that no one can access the MySQL server without authorization. Even though I set the password earlier, this step adds an extra layer of security. After setting the password, I used it to login to mySQL to ensure that it is working.
![Image](https://github.com/user-attachments/assets/59ac215c-f567-43c9-9948-5666d9f65152)

### Personal Notes:
- The importance of securing the MySQL installation was reinforced through the secure installation script and password validation settings.
- Lessons learned included the significance of maintaining strong passwords to prevent unauthorized access.
- The installation of MySQL was executed through a straightforward process using the Linux package manager apt, which ensures a stable and tested version of MySQL is installed. A key design choice here was the use of MySQL as the relational database management system, which was driven by its robustness, scalability, and wide adoption in web applications. Logging into MySQL using the sudo mysql command provided root user access, crucial for configuring the database securely and granting administrative privileges. Setting a strong root password and running the mysql_secure_installation script ensured the database was secured against unauthorized access. The choice to enable the VALIDATE PASSWORD PLUGIN adds a layer of security by enforcing stronger password policies, a necessary step to mitigate potential vulnerabilities.
---

## Step 3: Install PHP Server

I installed the PHP server using the command:

```bash
sudo apt install php-fpm php-mysql
```
### Personal Notes:
- The choice to install PHP as the server-side scripting language is due to its flexibility and widespread use in web development, particularly when working with MySQL databases. Using php-fpm (FastCGI Process Manager) ensures efficient handling of dynamic content, while php-mysql allows seamless integration between PHP and MySQL. This setup enables the server to process dynamic requests such as database queries. The decision to install these packages via apt was again due to the reliability of Ubuntu’s repositories, ensuring compatibility with the system environment. The combination of PHP and MySQL is a common and proven approach for building dynamic websites that can handle various data operations.
---

## Step 4: Configuring Nginx to Use PHP Processor

In this step, I configured the Nginx web server to use the PHP processor by setting up server blocks.

### Create Root Directory and Change Ownership

First, I created the root web directory:

```bash
sudo mkdir /var/www/projectLEMP
```

Next, the ownership of the directory was changed to the current system user:

```bash
sudo chown -R $USER:$USER /var/www/projectLEMP
```

### Create New Server Block for the Project

I created a new server block configuration file for projectLEMP in Nginx's sites-available directory:

```bash
sudo nano /etc/nginx/sites-available/projectLEMP
```
![Image](https://github.com/user-attachments/assets/f632c34a-3cbb-480f-b43c-ac01ed296dcb)

#### Explanation of Directives and Location Blocks

1. **listen 80:** Instructs Nginx to listen for incoming HTTP connections on port 80.
2. **server_name projectLEMP www.projectLEMP:** Specifies the domain names the server block should respond to.
3. **root /var/www/projectLEMP:** Defines the root directory for the site’s files.
4. **index index.html index.htm index.php:** Defines the default index files.
5. **location / { try_files $uri $uri/ =404; }:** Handles all requests to the root.
6. **location ~ .php$ { include snippets/fastcgi-php.conf; fastcgi_pass unix:/var/run/php/php8.3-fpm.sock; }:** Matches PHP files and processes them.
7. **location ~ /.ht { deny all; }:** Denies access to sensitive files.

### Activate Configuration

I activated the configuration by creating a symbolic link:

```bash
sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/
```

After linking, I ran the Nginx syntax test command to confirm no errors:

```bash
sudo nginx -t
```
![Image](https://github.com/user-attachments/assets/7ea5a099-43bc-4bb6-b077-9890950152c4)

### Disable Default Nginx Host and Reload New Configuration

To prevent conflicts, I disabled the default Nginx host:

```bash
sudo unlink /etc/nginx/sites-enabled/default
```

I then reloaded Nginx to apply the changes:

```bash
sudo systemctl reload nginx
```

### Creating an Index File

I created an `index.html` file to test server functionality:

```bash
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
sudo echo "Hello LEMP from hostname $(curl -H "X-aws-ec2-metadata-token: $TOKEN" -s http://169.254.169.254/latest/meta-data/public-hostname) with public IP $(curl -H "X-aws-ec2-metadata-token: $TOKEN" -s http://169.254.169.254/latest/meta-data/public-ipv4)" > /var/www/projectLEMP/index.html
```
![Image](https://github.com/user-attachments/assets/ee5500ef-270e-460e-a3c8-4a83a3934749)

### Personal Notes:
- I learned the significance of server block configurations for managing multiple domains.
- The importance of testing configurations was highlighted through syntax checks and test files.
- The configuration of Nginx to serve PHP content involved setting up server blocks to manage requests. The design choices in this step were centered on creating a well-structured environment to handle multiple domains or projects in the future. By defining a new root directory for the project and changing its ownership to the system user, the configuration ensures proper permissions and file management. The directives used, such as listen 80, server names, and root paths, were selected to define how the server should handle incoming requests and where it should serve content from. Configuring Nginx to use FastCGI for PHP processing optimizes performance, as FastCGI is designed for high-speed communication between the web server and the PHP processor. Testing the configuration through syntax checks ensured that no errors would disrupt the service.
---

## Step 5: Testing PHP with Nginx

After setting up the LEMP stack, I created a test PHP file to confirm that Nginx could process PHP files:

```bash
nano /var/www/projectLEMP/info.php
```
![Image](https://github.com/user-attachments/assets/ce9ff4bb-b501-43f1-a41f-d86e2d8a2dbd)

I added PHP code to this file that outputs information about the server’s PHP configuration.

After saving the file, I accessed it via the web browser:

```bash
http://35.87.151.65/info.php
```
![Image](https://github.com/user-attachments/assets/21380793-aa77-4354-a502-ddc8794b2f22)

### Personal Notes:
- The successful output of PHP information confirmed that the PHP processor was working correctly, validating the setup.
- To validate that PHP was properly configured with Nginx, I created a simple PHP file to display server information. This approach served as a troubleshooting measure to ensure PHP requests were being handled correctly by Nginx. Successfully accessing the file via a web browser confirmed that the server could process PHP files, indicating that the configuration was correct. This step was critical to ensuring that dynamic content could be served by the LEMP stack, as any misconfiguration in the PHP or Nginx setup would prevent the application from functioning properly. This basic test file is a widely used method in server environments to verify PHP functionality after installation.
---

## Step 6: Retrieving Data from MySQL Database with PHP

To retrieve data from MySQL using PHP, I created a test database.

### Connect to MySQL with Root Privileges and Create Database

I connected to the MySQL console:

```bash
sudo mysql
```

In the MySQL console, I created a new database:

```sql
mysql> CREATE DATABASE `mac_database`;
```

### Create User and Assign Privileges

I created a new user named `example_user`:

```sql
mysql> CREATE USER 'macbeth'@'%' IDENTIFIED WITH mysql_native_password BY 'Password.1';
```

I granted full permissions to manage the database:

```sql
mysql> GRANT ALL ON mac_database.* TO 'macbeth'@'%';
```
![Image](https://github.com/user-attachments/assets/9e6af726-e173-4f44-aab6-2db62bb14ead)

After exiting the MySQL shell, I logged back in using the new user's credentials:

```bash


mysql -u macbeth -p mac_database
```
![Image](https://github.com/user-attachments/assets/8ff56a0b-33f1-47fc-8fbc-e19b729825fe)


### Creating a Table

In the MySQL console, I created a table named `mac_database.todo_list`:

```sql
CREATE TABLE mac_database.todo_list (
    item_id INT AUTO_INCREMENT,
    content VARCHAR(255),
    PRIMARY KEY (item_id)
    ); 
```
This setup established a basic structure for storing data, ensuring that the “macbeth” could manage the table and retrieve data using PHP later on. After creating the todo_list table inside the mac_database, I proceeded to insert some data into the table to test that everything was working correctly. Each row inserted represents a different to-do item, with the content field storing the item's description.

### Inserting Sample Data

I inserted sample data into the `mac_database.todo_list ` table:

```sql
INSERT INTO `mac_database.todo_list` (content) VALUES ("My first important item");
```
To add more test data, I repeated this command multiple times, changing the content each time. This allowed me to have multiple entries to test the functionality further.
For example, I ran similar commands to add more items: 
```sql
        INSERT INTO `mac_database.todo_list` (content) VALUES ("My second important item");
        INSERT INTO `mac_database.todo_list` (content) VALUES ("My third important item");
        INSERT INTO `mac_database.todo_list` (content) VALUES ("and this one more thing"); 
```
After inserting the data, I wanted to confirm that everything was saved correctly. I used the SELECT command to retrieve and display all the entries from the todo_list table: 
```sql
mysql> SELECT * FROM `mac_database.todo_list`; 
```
![Image](https://github.com/user-attachments/assets/7b78b953-5254-4599-9322-e307d6f572be)

Seeing this output confirmed that the data had been successfully saved to the database.

After verifying the data, I exited the MySQL console with the following command: 
```sql
        exit
``` 
At this point, I had successfully created a table, inserted data into it, and retrieved the data, confirming that everything was functioning correctly. 

### Retrieving Data with PHP

I created a PHP script to retrieve and display data from the MySQL database:

```<?php
$user = "macbeth";
$password = "Password.1";
$database = "mac_database";
$table = "todo_list";

try {
    $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
    echo "<h2>TODO</h2><ol>";

    // Using the $table variable in the query
    foreach($db->query("SELECT content FROM $table") as $row) {
        echo "<li>" . $row['content'] . "</li>";
    }
    
    echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}
?>

```
![Image](https://github.com/user-attachments/assets/b0cee418-eed6-48db-8315-09b83eb69dd7)

I saved this script as `fetch.php` in the project root directory and accessed it via the browser:

```bash
http://35.87.151.65/fetch.php
```
![Image](https://github.com/user-attachments/assets/a9eb193e-13c0-41f3-ad5d-93ec52f32497)

### Personal Notes:
- The importance of proper user permissions was emphasized during user and privilege setup.
- I gained valuable experience in integrating PHP with MySQL to fetch and display data.
- In this step, I focused on the integration of PHP with MySQL by creating a database and connecting it to PHP. The choice to create a new user with specific privileges was a best practice for security, ensuring that the database was only accessible to authorized users. By creating a simple to-do list table, I established a basic structure for storing and retrieving data. The sample data inserted into the database was used to confirm that MySQL was functioning correctly and that the connection between PHP and the database was solid. The decision to use PDO (PHP Data Objects) for database interaction provides a flexible and secure way to manage database connections, protecting against SQL injection and ensuring compatibility with different database systems. The successful retrieval of data through a PHP script validated the seamless interaction between the server, database, and scripting language, forming the foundation of dynamic web applications.
---

## Conclusion : Learnings and Challenges

Throughout the process of implementing the LEMP stack, I gained significant insight into the critical components of Linux, Nginx, MySQL, and PHP, which form the foundation for creating a robust web server environment. I learned how to install each of these components, ensuring they work seamlessly together to deliver web applications efficiently. By starting with setting up an Ubuntu server, I developed my Linux command-line skills, followed by configuring Nginx as a web server to handle requests. Understanding how to secure the MySQL database, manage databases, and connect them to PHP was a valuable experience that reinforced my knowledge of database management and server-side scripting. The hands-on approach enabled me to troubleshoot configuration issues and deepen my understanding of server environments, which has been instrumental in developing a comprehensive view of web hosting architecture.
However, the process was not without its challenges. One major difficulty I faced was troubleshooting server errors, particularly when services failed to run after installation. Configuring the firewall to allow HTTP/HTTPS traffic and resolving issues with PHP and MySQL connectivity required detailed research and trial and error. Despite these challenges, I employed self-study and problem-solving strategies, such as consulting online documentation and forums, to overcome them. This experience not only enhanced my technical skills but also improved my ability to work independently and resolve technical issues, thereby reinforcing my confidence in implementing complex server environments.

```



