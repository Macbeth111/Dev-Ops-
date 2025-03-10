# DevOps Tooling Website Deployment with CI/CD - Jenkins

## Table of Contents
- [Introduction](#introduction)
- [Project Overview](#project-overview)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Understanding CI/CD](#understanding-cicd)
- [Implementation Steps](#implementation-steps)
  - [Step 1: Set Up Jenkins Server](#step-1-set-up-jenkins-server)
  - [Step 2: Configure Jenkins for CI/CD](#step-2-configure-jenkins-for-cicd)
  - [Step 3: Integrate GitHub with Jenkins Webhook](#step-3-integrate-github-with-jenkins-webhook)
  - [Step 4: Configure Jenkins to Deploy to NFS Server](#step-4-configure-jenkins-to-deploy-to-nfs-server)
- [Testing and Validation](#testing-and-validation)
- [Challenges and Resolutions](#challenges-and-resolutions)
- [Lessons Learned](#lessons-learned)

---

## Introduction
This project extends previous implementations (DevOps Tooling Website and Load Balancer Solution) by introducing **Continuous Integration/Continuous Deployment (CI/CD) using Jenkins**. The goal is to create an automated deployment pipeline that improves efficiency, reduces errors, and accelerates delivery.

## Project Overview
We enhance the existing infrastructure by:
- Utilizing a **web, database, and NFS server setup**.
- Leveraging a **load balancer** for improved availability.
- Setting up **Jenkins** for automated build and deployment.
- Integrating **GitHub Webhooks** to trigger builds on code changes.

## Architecture
The updated architecture consists of:
- **Load Balancer**: Distributes traffic among multiple web servers.
- **Three Web Servers**: Connected to an NFS and database server.
- **CI/CD Layer (Jenkins)**: Automates build and deployment.
- **GitHub Integration**: Triggers Jenkins builds on repository updates.

## Prerequisites
Before starting, ensure:
- The **DevOps Tooling Website Solution** (NFS, Database, Web Servers) is set up.
- A **load balancer** is implemented.
- Basic **Linux and Jenkins** knowledge.
- Access to an **AWS account**.
- A **GitHub repository** for the project.

## Understanding CI/CD

### Continuous Integration (CI)
- Developers frequently merge changes.
- Automated builds and tests detect integration issues early.

### Continuous Delivery (CD)
- Code is automatically **prepared for release** after passing CI.
- Requires **manual approval** for production deployment.

### Continuous Deployment (CD)
- Code is automatically **deployed to production** with no manual intervention.

### Key Benefits
- **Faster delivery cycles**
- **Improved code stability**
- **Reduced manual effort**
- **Increased team efficiency**

## Implementation Steps

### Step 1: Set Up Jenkins Server
**Relevance:** Jenkins is the backbone of the CI/CD pipeline. Setting up a Jenkins server ensures that code changes are automatically built, tested, and deployed, reducing human errors and improving deployment speed.

1. **Provision an EC2 instance** (t2.micro, Ubuntu 24.04 LTS).
2. **Configure security groups** (allow SSH `22`, HTTP `8080`).
3. **Install Java and Jenkins**:
   ```sh
   sudo apt update
   sudo apt install default-jdk-headless
   wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian/jenkins.io-2023.key
   echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
   sudo apt-get update
   sudo apt-get install jenkins
   ```
4. **Start and enable Jenkins**:
   ```sh
   sudo systemctl start jenkins
   sudo systemctl enable jenkins
   ```
5. **Retrieve initial admin password**:
   ```sh
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```
6. **Access Jenkins via browser** and complete the setup (`http://<your-server-ip>:8080`).

### Step 2: Configure Jenkins for CI/CD
**Relevance:** Configuring Jenkins ensures that it can pull the latest changes from GitHub, build the application, and deploy it without manual intervention, enabling automation and consistency in deployments.

- Install **necessary plugins**.
- Set up **Jenkins Freestyle Project**.
- Configure **Source Code Management (GitHub repo URL)**.

### Step 3: Integrate GitHub with Jenkins Webhook
**Relevance:** The webhook ensures that every code push to GitHub automatically triggers a Jenkins build, making the process fully automated and reducing deployment delays.

1. **Configure Webhook in GitHub**:
   - Navigate to `Settings > Webhooks`.
   - Add webhook with:
     - **Payload URL**: `http://<Jenkins-IP>:8080/github-webhook/`
     - **Content type**: `application/json`
     - **Trigger on push events**.
2. **Configure Jenkins job**:
   - Enable **GitHub hook trigger for GITScm polling**.
   - Test by making a commit; verify the build triggers.

### Step 4: Configure Jenkins to Deploy to NFS Server
**Relevance:** Ensures that successfully built application files are copied to the NFS server so that all web servers can access them, leading to a consistent and centralized deployment approach.

1. **Install "Publish Over SSH" plugin** in Jenkins.
2. **Configure SSH Connection** to NFS server:
   - Add private IP of NFS server (`172.31.10.195`).
   - Provide SSH credentials.
3. **Test and Validate Deployment**:
   - Ensure Jenkins copies files to `/mnt/apps` on NFS.
   - Adjust permissions if needed:
     ```sh
     sudo chown -R nobody: /mnt/apps
     sudo chmod -R 777 /mnt/apps
     ```

## Testing and Validation
- Make a change in **GitHub**.
- Verify **Jenkins build triggers automatically**.
- Confirm **files deploy to NFS server**.
- Access the **website via load balancer**.

## Challenges and Resolutions

| Challenge | Resolution |
|-----------|------------|
| **Jenkins Webhook not triggering** | Verified Webhook settings in GitHub, checked Jenkins logs for connectivity issues. |
| **NFS file sync issues** | Checked permissions and ensured correct mount points. |
| **Database connection failures** | Ensured correct credentials in `function.php` file. |
| **Security Group restrictions** | Updated AWS security group rules to allow necessary ports. |

## Lessons Learned
1. **Automated deployment reduces manual effort and errors**.
2. **Security group configurations must be carefully managed**.
3. **CI/CD requires robust testing to prevent faulty releases**.
4. **Understanding Jenkins logs is crucial for debugging**.
5. **Proper file permissions ensure seamless deployment**.

---

This documentation provides a structured approach to implementing **CI/CD with Jenkins**, helping streamline **DevOps website deployments** effectively.
