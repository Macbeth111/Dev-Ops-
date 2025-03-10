# DevOps Tooling Website Deployment with CI/CD - Jenkins

## Table of Contents
- [Introduction](#introduction)
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

## Implementation Steps

### Step 1: Set Up Jenkins Server
**Relevance:** Jenkins is the backbone of the CI/CD pipeline. Setting up a Jenkins server ensures that code changes are automatically built, tested, and deployed, reducing human errors and improving deployment speed.

1. **Provision an EC2 instance** (t2.micro, Ubuntu 24.04 LTS).
2. **Configure security groups** (allow SSH `22`, HTTP `8080`).

  ![Port 8080 on Jenkins server](https://github.com/user-attachments/assets/3027658d-b359-4d5e-9679-82f33ccb46d1)

4. **Install Java and Jenkins**:
   ```sh
   sudo apt update
   sudo apt install default-jdk-headless
   wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian/jenkins.io-2023.key
   echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
   sudo apt-get update
   sudo apt-get install jenkins
   ```
5. **Start and enable Jenkins**:
   ```sh
   sudo systemctl start jenkins
   sudo systemctl enable jenkins
   sudo systemctl status jenkins
   ```
   ![Jenkins Working Properly](https://github.com/user-attachments/assets/2183a0b2-149e-4e93-a6e0-c8c7aa81aa82)

6. **Retrieve initial admin password**:
   Retrieve the Jenkins Admin password and use it to access Jenkins platform
   ```sh
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```
7. ** Follow the on-screen instructions to complete the installation, including installing recommended plugins and creating the first admin user.
   
![step1](https://github.com/user-attachments/assets/8e7f5ff1-bf35-415f-9fcb-f7e7cd72ea6a)

![step2](https://github.com/user-attachments/assets/da2d98a2-73f6-4a9e-9b83-d196e2f21d85)

![step3](https://github.com/user-attachments/assets/cb5ec5b6-f795-44b1-936d-c556f54c3163)

![step4](https://github.com/user-attachments/assets/95e59019-b0d2-4d7f-ab5d-beb43fd60412)

![step5](https://github.com/user-attachments/assets/e8ffb1aa-3a16-4d61-bce5-f1622198d217)

8. **Access Jenkins via browser** and complete the setup (`http://<your-server-ip>:8080`).

![Screenshot 2025-03-08 103130](https://github.com/user-attachments/assets/a02fc050-7dee-4a02-af44-b140b4bb3c3e)

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
![webhook added](https://github.com/user-attachments/assets/d60cadca-b97a-446e-862f-6ab5fe4b23fe)

02. **Then confirm in Jenkins to see if Github webhook is successfully linked to Jenkins.**

![second Jenkins connected successfully](https://github.com/user-attachments/assets/24e4b77d-78ea-4cdb-a80e-5ec288349357)

03. **Configure Jenkins job**:
   - Enable **GitHub hook trigger for GITScm polling**.
   - Test by making a commit; verify the build triggers.

![Trigger Updated successfully](https://github.com/user-attachments/assets/a5e75b8b-eecf-46ae-b49b-db2be2c9707c)

### Step 4: Configure Jenkins to Deploy to NFS Server
**Relevance:** Ensures that successfully built application files are copied to the NFS server so that all web servers can access them, leading to a consistent and centralized deployment approach.

1. **Install "Publish Over SSH" plugin** in Jenkins.

  ![installing publish over ssh plugin](https://github.com/user-attachments/assets/5525dbfe-3224-4730-bb18-1440dc63e300)

3. **Configure SSH Connection** to NFS server:
   - Add NFS private key to Jenkins
   - Add private IP of NFS server (`172.31.33.158`).
   - Provide SSH credentials.
   - Edit NFS server Inbound rules to allow SSH access to Jenkins Sever
   
![Screenshot 2025-03-08 160208](https://github.com/user-attachments/assets/160a82b8-4e87-4f93-9003-4fbfbd08ac9a)

4. **Test and Validate Deployment**:
   - Ensure Jenkins copies files to `/mnt/apps` on NFS.
   - Adjust permissions if needed:
     ```sh
     sudo chown -R nobody: /mnt/apps
     sudo chmod -R 777 /mnt/apps
     ```
## Testing and Validation
- Make a change in **GitHub**.(In this exercise I added a note the README.md file in Github at exactly 4:21 pm)

![Screenshot 2025-03-08 162511](https://github.com/user-attachments/assets/98718e87-ff71-4cb9-a9a7-7f08cbd688a5)

- Verify **Jenkins build triggers automatically**.
- Confirm **files deploy to NFS server**.
- Access the file you edited in Github in your NFS console to see if the changes reflect.

![Note successfully reflected in console](https://github.com/user-attachments/assets/3ce4b152-dc96-4199-96db-c90b65740737)

![last build time in console](https://github.com/user-attachments/assets/3810fc2e-4f2b-4362-89a1-ea2d513de83b)

## Challenges and Resolutions

| Challenge | Resolution |
|-----------|------------|
| **Jenkins Webhook not triggering** | Verified Webhook settings in GitHub, checked Jenkins logs for connectivity issues. |
| **NFS file sync issues** | Checked permissions and ensured correct mount points. |
| **Security Group restrictions** | Updated AWS security group rules to allow necessary ports. |

## Lessons Learned
1. **Automated deployment reduces manual effort and errors**.
2. **Security group configurations must be carefully managed**.
3. **CI/CD requires robust testing to prevent faulty releases**.
4. **Understanding Jenkins logs is crucial for debugging**.
5. **Proper file permissions ensure seamless deployment**.

---

