# 🚀 Jenkins CI/CD Pipeline Setup — AWS EC2 (Master-Agent Architecture)

This project demonstrates how to set up a complete **CI/CD pipeline using Jenkins** on **AWS EC2 instances**, with a **Master-Agent architecture**, Maven build integration, and automated **Pull → Build → Test → Deploy** stages.

---

## 📌 Project Objective

To configure a Jenkins Master-Agent setup on AWS EC2, install required tools and plugins, connect an Application Server as a Jenkins Agent via SSH, and write a Jenkins Pipeline script to automate the build, test, and deployment process.

---

## 🏗️ Architecture Overview

```
                 ┌─────────────────────┐
                 │   Jenkins Master     │
                 │   (EC2 Instance 1)   │
                 │  - Java JDK          │
                 │  - Jenkins           │
                 └──────────┬───────────┘
                            │ SSH Agent Connection
                            ▼
                 ┌─────────────────────┐
                 │  Application Server  │
                 │   (EC2 Instance 2)   │
                 │  - Java JDK          │
                 │  - App Deployment    │
                 └─────────────────────┘
```

---

## 🛠️ Prerequisites

- AWS account with EC2 access
- Basic knowledge of Linux commands
- GitHub repository (for pulling source code)
- Key pair (.pem file) for SSH access

---

## 🚀 Step-by-Step Implementation

### Step 1: Create 2 EC2 Instances

Launch **2 EC2 instances** on AWS with the following names:

| Instance Name | Purpose |
|----------------|---------|
| `Jenkins-Master` | Runs Jenkins server, manages pipeline |
| `Application-Server` | Acts as Jenkins Agent/Node, hosts the deployed application |

**Configuration used:**
- AMI: Ubuntu 22.04 / Amazon Linux 2
- Instance Type: t2.medium (recommended, t2.micro also works for practice)
- Security Group: Allow ports `22 (SSH)`, `8080 (Jenkins)`, `80 (HTTP)`
- Key Pair: Same `.pem` file for both instances (needed for SSH agent connection)

---

### Step 2: Install Java JDK on Application Server

SSH into the **Application Server** and install Java:

```bash
sudo apt update
sudo apt install openjdk-17-jdk -y
java -version
```

---

### Step 3: Install Java JDK and Jenkins on Jenkins Master Server

SSH into the **Jenkins-Master** instance and run:

```bash
# Update packages
sudo apt update

# Install Java (required for Jenkins)
sudo apt install openjdk-17-jdk -y
java -version

# Add Jenkins repository key
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null

# Add Jenkins repository
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

# Install Jenkins
sudo apt update
sudo apt install jenkins -y

# Start Jenkins service
sudo systemctl start jenkins
sudo systemctl enable jenkins
sudo systemctl status jenkins
```

Access Jenkins in browser:
```
http://<Jenkins-Master-Public-IP>:8080  - #enter master public ip in browser with attached :8080 port
```

Get the initial admin password:
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

---

### Step 4: Login to Jenkins → Create Credentials → Setup SSH Agent → Connect Node

1. Login to Jenkins using the initial admin password
2. Install **suggested plugins** and create the admin user
3. Go to **Manage Jenkins → Credentials → System → Global Credentials → Add Credentials**
   - Kind: **SSH Username with private key**
   - Username: `ubuntu` (or `ec2-user`)
   - Private Key: paste content of your `.pem` file
   - ID: `app-server-ssh-key`
4. Go to **Manage Jenkins → Nodes → New Node**
   - Node Name: `Application-Server`
   - Type: **Permanent Agent**
   - Remote root directory: `/home/ubuntu/jenkins-agent`
   - Launch method: **Launch agents via SSH**
   - Host: `<Application-Server-Private/Public-IP>`
   - Credentials: select the SSH credential created above
   - Host Key Verification Strategy: **Non verifying** (for practice setup)
5. Save and check node status — it should show **"Connected"** ✅

---

### Step 5: Install Required Jenkins Plugins

Go to **Manage Jenkins → Plugins → Available Plugins**, search and install:

- ✅ Pipeline Stage View
- ✅ Blue Ocean
- ✅ Maven Integration Plugin
- ✅ AWS Credentials Plugin

Restart Jenkins after installation if prompted.

---

### Step 6: Configure Maven Tool in Jenkins

1. Go to **Manage Jenkins → Tools (Global Tool Configuration)**
2. Under **Maven installations**, click **Add Maven**
3. Name: `Maven-3.9`
4. Check **Install automatically** and select the version
5. Click **Save**

---

### Step 7: Write Jenkins Pipeline Script (Pull → Build → Test → Deploy)

Create a **New Item → Pipeline** in Jenkins and add the following script:

```groovy
pipeline {
    agent any

    tools {
        maven 'Maven-3.9'
    }

    stages {
        stage('Pull') {
            steps {
                git branch: 'main', url: 'https://github.com/<your-username>/<your-repo>.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Deploy') {
            steps {
                sshagent(['app-server-ssh-key']) {
                    sh '''
                        scp -o StrictHostKeyChecking=no target/*.war ubuntu@<Application-Server-IP>:/home/ubuntu/deploy/
                        ssh -o StrictHostKeyChecking=no ubuntu@<Application-Server-IP> "sudo systemctl restart tomcat"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed. Check logs for details.'
        }
    }
}
```

> 📝 Note: Replace `<your-username>`, `<your-repo>`, and `<Application-Server-IP>` with your actual values.

---

## ✅ Output

- Jenkins Master and Application Server successfully connected via SSH Agent
- Pipeline executed all 4 stages: **Pull → Build → Test → Deploy**
- Application successfully deployed on Application Server
- Pipeline visualized using **Blue Ocean** and **Pipeline Stage View**

---

## 📸 Screenshots

*(Add your screenshots here — Jenkins dashboard, node connected status, pipeline stage view, Blue Ocean pipeline run, deployed application)*

```
![Jenkins Dashboard](screenshots/dashboard.png)
![Node Connected](screenshots/node-connected.png)
![Pipeline Stages](screenshots/pipeline-stages.png)
```

---

## 📝 Conclusion

This practical helped in understanding:
- How to set up a Jenkins Master-Agent architecture on AWS EC2
- Configuring SSH-based agent connections
- Installing and managing Jenkins plugins
- Writing a declarative pipeline script for full CI/CD automation (Pull, Build, Test, Deploy)

---

## 🧰 Tech Stack

`AWS EC2` `Jenkins` `Java JDK` `Maven` `SSH` `Blue Ocean` `Groovy Pipeline`

---

## 👤 Author

**[Your Name]**
📧 [your-email@example.com]
🔗 [LinkedIn Profile]
