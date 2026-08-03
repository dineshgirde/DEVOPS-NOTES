# 🛡️ Insure-Me — End-to-End CI/CD Pipeline using Jenkins, SonarQube, AWS S3 & Docker

This project demonstrates a complete **CI/CD pipeline** for a Java (Maven / Spring Boot) application called **Insure-Me**. The pipeline pulls code from GitHub, builds & tests it, runs static code analysis via **SonarQube**, backs up the build artifact to **AWS S3**, builds a **Docker image**, pushes it to **Docker Hub**, and finally deploys the container on a dedicated **Application Server** — all orchestrated through **two Jenkins pipeline jobs**.

---

## 📌 Architecture Overview

Two AWS EC2 servers were used:

| Server | Role | Jenkins Role | Tools Installed |
|---|---|---|---|
| **Master Server** | Jenkins + SonarQube host | Jenkins **Controller** (runs Job 1) | Jenkins, SonarQube, Maven, Java (JDK), Docker, AWS CLI |
| **Application Server** | Deployment target | Jenkins **Agent/Node** — label: `project` (runs Job 2) | Docker, Java (JDK), AWS CLI |

```
 GitHub Repo
     │  (push)
     ▼
 ┌────────────────────────────────────────────┐
 │              Master Server (Jenkins)         │
 │  JOB 1 — insureme-build-pipeline             │
 │  ┌────────────────────────────────────────┐ │
 │  │ code-pull → code-build → code-test      │ │
 │  │  (SonarQube) → quality-gate →           │ │
 │  │  code-push (S3) → docker-image →        │ │
 │  │  image-push (Docker Hub)                │ │
 │  └────────────────────────────────────────┘ │
 └──────────────┬────────────────┬─────────────┘
                │                │
        (artifact backup)   (image pushed)
                ▼                ▼
        ┌───────────────┐  ┌───────────────┐
        │   AWS S3       │  │  Docker Hub    │
        │  bucket        │  │  dineshgirde97 │
        │(artifact backup)│  │   /projecta   │
        └───────────────┘  └───────┬───────┘
                                    │
                                    ▼
                ┌───────────────────────────────────┐
                │   Application Server (Jenkins Node,│
                │        label = "project")          │
                │  JOB 2 — insureme-deploy-pipeline  │
                │  code-deploy → docker run container│
                │        (host:8089 → cont:8081)     │
                └───────────────────────────────────┘
```

---

## ⚙️ Tools & Technologies Used

- **Jenkins** – CI/CD orchestration (Master = controller, Application server = agent node)
- **SonarQube** – Static code analysis & Quality Gate
- **Maven** – Build tool (`mvn clean package`)
- **AWS S3** – Artifact backup/storage (`.jar` file)
- **Docker / Docker Hub** – Containerization & image registry
- **GitHub** – Source code management
- **Blue Ocean** – Visual pipeline view

---

## 🔌 Jenkins Plugins Installed

| Plugin | Purpose |
|---|---|
| Docker Pipeline / Docker plugin | `docker build`, `docker push`, `docker run` from pipeline |
| SonarQube Scanner | Provides the `sonar-scanner` tool + `withSonarQubeEnv` step |
| SonarQube Maven Integration | Sonar–Maven project support |
| Blue Ocean | Modern visual pipeline view |
| AWS Credentials / Pipeline: AWS Steps | `withCredentials([aws(...)])` support for AWS CLI/S3 |
| Pipeline: Stage View | Graphical stage timeline |
| SSH Build Agents | Required to register the Application server as a Jenkins agent node |

---

## 🛠️ Global Tool Configuration (Manage Jenkins → Tools)

| Tool Name (as used in Jenkinsfile) | Type |
|---|---|
| `maven` | Maven installation |
| `sonar-scanner` | SonarQube Scanner installation |

> These names must match exactly what's referenced in the `tools {}` and `environment { SCANNER_HOME = tool 'sonar-scanner' }` blocks of the Jenkinsfile.

---

## 🔑 Credentials Configured in Jenkins

`Manage Jenkins → Credentials → System → Global`

| ID | Type | Used In | Purpose |
|---|---|---|---|
| `ubuntu` | Username/Password or SSH key | `code-pull` stage, and Application server Node config | GitHub authentication for `git` checkout, and connecting the Application server as a Jenkins Node |
| `aws-cred` | AWS Credentials (Access Key/Secret) | `code-push` stage | `aws s3 cp` — uploads build artifact to S3 |
| `docker-cred` | Username with Password | `image-push` stage | `docker login` + `docker push` to Docker Hub |
| `sonar-cred` | Secret Text (SonarQube Token) | `code-test-quality gate` stage | Authenticates the Quality Gate check with SonarQube |

Also configured under `Manage Jenkins → System → SonarQube Servers`: a server entry named **`sonar-server`** pointing to the SonarQube URL (`http://<master-ip>:9000`).

---

## 🔑 Step-by-Step: How Each Credential Was Created & Applied

**Path for all:** `Manage Jenkins → Credentials → System → Global credentials → Add Credentials`

### 1️⃣ `ubuntu` — GitHub / SSH access

1. Kind: **Username with password** (or **SSH Username with private key**)
2. Username: GitHub username / server SSH user
3. Password / Private Key: GitHub PAT or server's SSH private key
4. ID: `ubuntu`
5. Click **Create**

**Applied in:**
```groovy
git branch: 'main', credentialsId: 'ubuntu', url: 'https://github.com/mukundDeo9325/Project-InsureMe1.git'
```
Also selected as the SSH credential when adding the Application server as a Jenkins Node.

---

### 2️⃣ `aws-cred` — AWS access

1. Kind: **AWS Credentials**
2. Access Key ID: paste AWS IAM Access Key
3. Secret Access Key: paste AWS IAM Secret Key
4. ID: `aws-cred`
5. Click **Create**

**Applied in:**
```groovy
withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws-cred', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
    sh 'aws s3 cp ${warFile} s3://${S3_BUCKET}/Artifacts/ --region ${REGION}'
}
```

---

### 3️⃣ `docker-cred` — Docker Hub login

1. Kind: **Username with password**
2. Username: Docker Hub username (`dineshgirde97`)
3. Password: Docker Hub password / access token
4. ID: `docker-cred`
5. Click **Create**

**Applied in:**
```groovy
withCredentials([usernamePassword(credentialsId: 'docker-cred', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
    sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
    sh 'docker push dineshgirde97/projecta'
}
```

---

### 4️⃣ `sonar-cred` — SonarQube token

1. **First, generate the token in SonarQube:**
   `SonarQube → Administrator (top-right) → My Account → Security → Generate Tokens` → give name → **Generate** → copy the token
2. **Then, in Jenkins:**
   Kind: **Secret text**
   Secret: paste the copied SonarQube token
   ID: `sonar-cred`
3. Click **Create**

**Applied in:**
```groovy
waitForQualityGate abortPipeline: false, credentialsId: 'sonar-cred'
```

---

## 🖥️ Application Server — Registered as a Jenkins Node

Job 2 runs with `agent { label 'project' }`, meaning the Application server was **added as a Jenkins Node** (not just SSH'd into from the pipeline):

`Manage Jenkins → Nodes → New Node`
- **Name:** e.g. `application-server`
- **Labels:** `project`
- **Launch method:** Launch agent via SSH (Host = Application server IP, Credentials = `ubuntu`)
- **Remote root directory:** e.g. `/home/ubuntu/jenkins-agent`

Once connected, this node executes Job 2 (`docker run`) directly on the Application server, using the Docker daemon installed there.

---

## 🖥️ Server Setup (Both Master & Application Servers)

```bash
sudo apt update -y

# Java (required by Jenkins, SonarQube, Maven build & Jenkins agent)
sudo apt install openjdk-17-jdk -y

# Docker
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER

# AWS CLI
sudo apt install awscli -y
```

### Docker Socket Permission (applied on BOTH Master & Application servers)

Since Job 1 (on Master) runs `docker build`/`docker push`, and Job 2 (on the Application server, via the Jenkins agent) runs `docker run`, the Jenkins process on **both** servers needs access to the Docker daemon socket:

```bash
sudo chmod 777 /var/run/docker.sock
```

> ⚠️ **Permission note:** `chmod 777 /var/run/docker.sock` gets Jenkins → Docker access working quickly on both servers, which is fine for this lab. In production, prefer adding the Jenkins/agent user to the `docker` group instead (`sudo usermod -aG docker jenkins` on the Master, and the equivalent agent-launch user on the Application server, followed by a restart) — same access without exposing the socket to every local user. Also note this permission **resets on reboot** since `/var/run` is typically a `tmpfs`; for persistence across reboots, use the `usermod -aG docker` approach or a systemd override instead of a manual `chmod`.

### Master Server – Additional Setup (Jenkins + SonarQube)

```bash
# Jenkins
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update -y
sudo apt install jenkins -y
sudo systemctl enable jenkins
sudo systemctl start jenkins

sudo systemctl restart jenkins
sudo systemctl restart docker
```

**SonarQube:**

```bash
sudo adduser sonarqube
sudo su - sonarqube
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.9.0.65466.zip
sudo apt install unzip -y
unzip sonarqube-9.9.0.65466.zip
mv sonarqube-9.9.0.65466 sonarqube
cd sonarqube/bin/linux-x86-64/
./sonar.sh start
```

SonarQube → `http://<master-server-ip>:9000`

---

## 🚀 JOB 1 — Build, Test, Analyze & Push (`agent any`, runs on Master)

```groovy
pipeline {
    agent any
    tools {
        maven 'maven'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        S3_BUCKET = "awsbackends3blocktestkaraychaahe"
        REGION = "ap-northeast-1"
        warFile = "target/Insurance-0.0.1-SNAPSHOT.jar"
    }
    stages {
        stage('code-pull') {
            steps {
                git branch: 'main', credentialsId: 'ubuntu', url: 'https://github.com/mukundDeo9325/Project-InsureMe1.git'
            }
        }
        stage('code-build') {
            steps {
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
        stage('code-push') {
            steps {
                withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws-cred', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                   sh 'aws s3 cp ${warFile} s3://${S3_BUCKET}/Artifacts/ --region ${REGION}'
                }
            }
        }
        stage('docker-image') {
            steps {
                sh 'docker build -t dineshgirde97/projecta .'
            }
        }

        stage('image-push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-cred', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
                    sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
                    sh 'docker push dineshgirde97/projecta'
                }
            }
        }
    }
}
```

**Stage-by-stage summary:**

| Stage | What it does |
|---|---|
| `code-pull` | Clones `main` branch from GitHub using `ubuntu` credential |
| `code-build` | `mvn clean package` → produces `target/Insurance-0.0.1-SNAPSHOT.jar` |
| `code-test` | Runs SonarQube static analysis via `sonar-scanner` against `sonar-server` |
| `code-test-quality gate` | Waits for SonarQube's Quality Gate result (`abortPipeline: false` → pipeline continues even if gate fails, only reports status) |
| `code-push` | Backs up the built `.jar` artifact to S3 bucket `awsbackends3blocktestkaraychaahe` (region `ap-northeast-1`) |
| `docker-image` | Builds Docker image `dineshgirde97/projecta` from the `Dockerfile` |
| `image-push` | Logs in to Docker Hub and pushes the image |

---

## 🚀 JOB 2 — Deploy (`agent { label 'project' }`, runs on Application Server)

```groovy
pipeline {
    agent { label 'project' }
    stages {
         stage('code-deploy'){
            steps{
                sh 'docker run -itd --name insure-me -p 8089:8081 dineshgirde97/projecta'
            }
        }
    }
}
```

**What it does:** Runs on the Application server node (label `project`) and starts the container in detached mode — host port **8089** mapped to container port **8081**.

> ⚠️ If Job 2 runs more than once without cleanup, `docker run` will fail with a "container name already in use" error, since the old `insure-me` container isn't removed first. Recommended fix:
> ```groovy
> sh 'docker rm -f insure-me || true'
> sh 'docker run -itd --name insure-me -p 8089:8081 dineshgirde97/projecta'
> ```

---

## 🌐 Application Access

```
http://<application-server-ip>:8089
```

---

## 🔗 SonarQube ↔ Jenkins Integration Steps

1. **SonarQube:** `Administration → Security → Users → Tokens` → generate a token (stored in Jenkins as `sonar-cred`).
2. **Jenkins:** `Manage Jenkins → Credentials` → add token as **Secret Text**, ID = `sonar-cred`.
3. **Jenkins:** `Manage Jenkins → System → SonarQube Servers` → add server, name = `sonar-server`, URL = `http://<master-ip>:9000`.
4. **Jenkins:** `Manage Jenkins → Tools → SonarQube Scanner installations` → name = `sonar-scanner`.
5. **SonarQube:** `Administration → Webhooks` → add `http://<jenkins-ip>:8080/sonarqube-webhook/` so the Quality Gate result is reported back to Jenkins for the `waitForQualityGate` step.

---

## 🔗 Chaining Job 1 → Job 2 (optional improvement)

Currently Job 1 and Job 2 are two separate pipelines that must be run independently. To fully automate deploy right after a successful build/push, add this stage to the end of Job 1:

```groovy
stage('trigger-deploy') {
    steps {
        build job: 'insureme-deploy-pipeline', wait: false
    }
}
```

---

## 🧾 Repository Structure

```
Project-InsureMe1/
├── src/                 # Application source code
├── images/              # Static assets/images
├── SonarQube/           # Sonar related configs
├── Dockerfile           # Docker image definition
├── Jenkinsfile          # Job 1 — build/test/push pipeline
├── playbook.yml         # Ansible playbook (alt. deployment option)
├── pom.xml              # Maven project file
├── mvnw / mvnw.cmd       # Maven wrapper scripts
├── demo-pipeline.md     # Pipeline notes
├── withansible.md       # Ansible deployment notes
├── test.md              # Testing notes
└── README.md
```

---

## ✅ Recommended Additions / Things to Double-Check

- [ ] **Security Group inbound rules:** `8080` (Jenkins), `9000` (SonarQube), `8089` (App), `22` (SSH), plus the Jenkins agent connection port if using SSH/JNLP.
- [ ] **Container cleanup in Job 2** — add `docker rm -f insure-me || true` before `docker run` to make Job 2 safely re-runnable.
- [ ] **Image tagging** — `dineshgirde97/projecta` currently has no explicit tag (defaults to `latest`); consider `${BUILD_NUMBER}` for rollback/traceability, and update Job 2 to pull that specific tag.
- [ ] **`docker pull` before `docker run`** in Job 2, so the Application server always deploys the freshest pushed image instead of a locally cached one.
- [ ] **Quality Gate strictness** — `abortPipeline: false` means a failed gate won't stop the pipeline; set to `true` if failed code quality should block S3/Docker/deploy stages.
- [ ] **Jenkins backup** — install the *ThinBackup* plugin or periodically back up `/var/lib/jenkins`.
- [ ] **`.gitignore`** — exclude `target/`, `*.log`, `.idea/`, `.DS_Store`.
- [ ] **`docker.sock` permission** — currently `chmod 777` on both Master and Application server; replace with `usermod -aG docker <jenkins/agent-user>` for better security, and note that `chmod 777` won't survive a reboot.
- [ ] **HTTPS** — Jenkins/SonarQube currently served over plain `http://` (browser shows "Not secure"); add Nginx + Let's Encrypt reverse proxy for production-like setup.

---

## 👤 Author

**Mukund Deo**
GitHub: [@mukundeo9325](https://github.com/mukundeo9325)

---

## 📄 License

This project is for learning/practice purposes (DevOps CI/CD lab).
