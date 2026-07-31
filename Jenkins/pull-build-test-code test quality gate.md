# Jenkins + SonarQube CI/CD Setup (Project: InsureMe)

This document explains, step by step, how to set up **Jenkins** and **SonarQube (Docker)**, install plugins, generate a token, and run the pipeline.

---

## 1. Prerequisites

This setup uses **two separate EC2 instances (Ubuntu)**:

| Server | Purpose | Open Ports (Security Group) |
|--------|------|------------------------------|
| **Server 1 - Jenkins Server** | Jenkins will be installed here | `8080` → Jenkins |
| **Server 2 - Application/Code Server** | Application code + SonarQube (Docker) + AWS CLI | `9000` → SonarQube |

---

## 2. Server 1: Jenkins Server Setup

### 2.1 Install Java

```bash
sudo apt update
sudo apt install fontconfig openjdk-21-jre -y
java -version
```

### 2.2 Install Jenkins

```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key

echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install jenkins -y
```

### 2.3 Start the Jenkins Service

```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
sudo systemctl status jenkins
```

- The status should show `active (running)`.

### 2.4 Open Jenkins in the Browser

- Open in browser: `http://<jenkins-server-ip>:8080`
- Wait a few seconds — the Jenkins first-time setup page will load (**"Unlock Jenkins"**)

### 2.5 Get the Initial Admin Password

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

- Copy this password and paste it into Jenkins' **"Administrator password"** box → click **Continue**

### 2.6 Install Plugins

- On the **"Customize Jenkins"** page → select **"Install suggested plugins"**
- Jenkins will automatically install the default plugins (Git, Pipeline, Credentials, etc.)
- Wait until all plugins finish installing

### 2.7 Create the Admin User

- Fill in the **"Create First Admin User"** form:
  - Username
  - Password
  - Confirm Password
  - Full Name
  - E-mail Address
- Click **Save and Continue**

### 2.8 Instance Configuration

- On the **"Instance Configuration"** page, confirm the Jenkins URL (it will already be filled in as: `http://<jenkins-server-ip>:8080/`)
- Click **Save and Finish**

### 2.9 Setup Complete

- Click the **"Start using Jenkins"** button
- The Jenkins Dashboard will now open ✅

---

## 3. Server 2: Application Server Setup (where the code lives)

On this server we will install: **Java**, **Docker + SonarQube**, and **AWS CLI**.

### 3.1 Install Java

```bash
sudo apt update
sudo apt install fontconfig openjdk-21-jre -y
java -version
```

### 3.2 Install Docker

```bash
sudo apt update -y
sudo apt install docker.io -y
sudo usermod -aG docker $USER
sudo usermod -aG docker jenkins
newgrp docker
sudo systemctl restart docker
```

### 3.3 Run SonarQube in Docker

```bash
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```

- Open in browser: `http://<application-server-ip>:9000`
- Default login: `admin / admin` (you'll be asked to change the password on first login)

### 3.4 Install AWS CLI

```bash
sudo snap install aws-cli --classic
aws --version
```

> **Note:** We will **not** run `aws configure` manually here. The AWS Access Key ID and Secret Access Key will be stored as Jenkins **Credentials** instead (Section 7), so they can be used securely at pipeline run time.

---

## 4. Generate SonarQube Token + Create Webhook

### 4.1 Generate a Token

1. SonarQube dashboard → **Administration → Security → Users**
2. Next to your user (e.g. `admin`), click the **Tokens** icon
3. Give the token a name (e.g. `jenkins-token`)
4. Click **Generate**
5. Copy the token and save it somewhere safe (it won't be shown again)

### 4.2 Create a Webhook (so Jenkins receives the Quality Gate result)

1. SonarQube dashboard → **Administration → Configuration → Webhooks**
2. Click **Create**
3. Name: `jenkins-webhook`
4. URL: `http://<jenkins-server-ip>:8080/sonarqube-webhook/`
5. Click **Create**

---

## 5. Install Required Jenkins Plugins

Go to **Manage Jenkins → Plugins → Available plugins** and install these:

- [ ] SonarQube Scanner
- [ ] Maven Integration
- [ ] Pipeline
- [ ] Git
- [ ] AWS Credentials

Restart Jenkins after installing.

---

## 6. Add the SonarQube Token as a Jenkins Credential

1. **Manage Jenkins → Credentials → System → Global credentials → Add Credentials**
2. Kind: `Secret text`
3. Secret: (paste the token you copied from SonarQube)
4. ID: `sonar-cred`
5. Click **Create**

---

## 7. Add AWS Credentials (AWS Credentials Plugin)

First, install the **AWS Credentials plugin** (if not already installed):

1. **Manage Jenkins → Plugins → Available plugins**
2. Search for: `AWS Credentials`
3. Install it and restart Jenkins

Now add the credential:

1. **Manage Jenkins → Credentials → System → Global credentials → Add Credentials**
2. **Kind:** `AWS Credentials`
3. **Access Key ID:** `<your AWS Access Key ID>`
4. **Secret Access Key:** `<your AWS Secret Access Key>`
5. **ID:** `aws-cred`
6. Click **Create**


---

## 8. Configure the SonarQube Server in Jenkins

1. **Manage Jenkins → System → SonarQube servers**
2. Click **Add SonarQube**
   - Name: `sonar-server`
   - Server URL: `http://<application-server-ip>:9000`
   - Server authentication token: `sonar-cred` (the one you just created)
3. Click Save

---

## 9. Configure the Maven Tool in Jenkins

1. **Manage Jenkins → Tools → Maven installations**
2. Click **Add Maven**
   - Name: `maven`
   - Install automatically ✅ (or select your own Maven version)
3. Click Save

---

## 10. Configure the SonarQube Scanner Tool in Jenkins

1. **Manage Jenkins → Tools → SonarQube Scanner installations**
2. Click **Add SonarQube Scanner**
   - Name: `sonar-scanner`
   - Install automatically ✅
3. Click Save

---

## 11. Create the Jenkins Pipeline Job

1. **New Item → Pipeline** → give it a name (e.g. `InsureMe-Pipeline`)
2. In the **Pipeline** section → Definition: `Pipeline script`
3. Paste the script below:

```groovy
pipeline {
    agent any 
    tools{
        maven 'maven'
    }
    environment {
        SCANNER_HOME =  tool 'sonar-scanner'
    }
    stages {
        stage('code-pull'){
            steps{
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/mukundDeo9325/Project-InsureMe1.git']])
            }
        }
        stage('code-build'){
            steps{
                sh 'mvn clean package'
            }
        }
        stage("code-test") {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=InsureMe \
                        -Dsonar.projectName=InsureMe \
                        -Dsonar.sources=src \
                        -Dsonar.java.binaries=target/classes
                    '''
                }
            }
        }
        stage("code-test-quality gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-cred'
                }
            }
        }
    }
}
```

4. Click **Save**

---

## 12. Run the Pipeline

1. Open the job → click **Build Now**
2. Check the status of each stage (in Blue Ocean or the console output)
3. Go to the SonarQube dashboard (`http://<application-server-ip>:9000`), open the `InsureMe` project, and view the analysis results

---

## 13. Common Errors & Fixes

| Error | Fix |
|-------|----------|
| `mvn: command not found` | The Maven tool isn't configured correctly in Jenkins — check the Tools section |
| SonarQube connection refused | Check whether the Docker container is running (`docker ps`) |
| Quality gate timeout | Check whether the webhook is set up correctly (Section 4.2) |
| Permission denied on docker.sock | Run `sudo usermod -aG docker jenkins` and restart Jenkins |

---

## ✅ Final Result

After this setup, whenever code is pushed to the GitHub `main` branch (or the build is triggered manually), the Jenkins pipeline will automatically:
1. Pull the code
2. Build it with Maven
3. Analyze code quality with SonarQube
4. Check the Quality Gate result
