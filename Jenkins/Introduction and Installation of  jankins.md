# 🚀 Jenkins Automation Server Guide

## 📚 Complete Jenkins Setup & Command Cheatsheet
### ⚡ CI/CD Automation, Building, Testing, and Deployment Framework

---

## 🔹 What is Jenkins?
Amazon Jenkins is a self-contained, open source automation server which can be used to automate all sorts of tasks related to building, testing, and delivering or deploying software.

Jenkins can be installed through native system packages, Docker, or even run standalone by any machine with a Java Runtime Environment (JRE) installed.

**👉 It helps you:**
* Automate code integration (CI)
* Build application packages
* Execute automated test suites
* Deliver continuous deployment pipelines (CD)

---

## 🛠️ Complete Jenkins Installation Methods

Choose any of the following implementation environments to install Jenkins completely on your server box.

### 🐧 Method 1: Native Installation on Linux (Ubuntu / Debian)

#### Step 1: Update System Packages & Install Java (LTS JRE Required)
```bash
sudo apt update -y
sudo apt install openjdk-17-jre -y
```

#### Step 2: Add Jenkins Official Repository Keys
```bash
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://jenkins.io
```

#### Step 3: Append Jenkins Repository to System Sources
```bash
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://jenkins.io binary/" | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
```

#### Step 4: Install Jenkins Server Engine
```bash
sudo apt update -y
sudo apt install jenkins -y
```

#### Step 5: Start & Enable Jenkins System Service
```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

#### Step 6: Verify Service Operational Status
```bash
sudo systemctl status jenkins
```

---

### 🐳 Method 2: Containerized Installation via Docker

#### Step 1: Run Jenkins Long-Term Support (LTS) Container
```bash
docker run -d \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  --name myjenkins \
  --restart always \
  jenkins/jenkins:lts
```

#### Step 2: Monitor Real-Time Container Initialization Logs
```bash
docker logs -f myjenkins
```

---

### 📦 Method 3: Standalone Java Executable (.WAR) Execution

#### Step 1: Fetch Official Jenkins Web Archive File
```bash
wget https://jenkins.io
```

#### Step 2: Launch Web Server Manually using Java Runtime
```bash
java -jar jenkins.war --httpPort=8080
```

---

## 🔓 How to Unlock Jenkins (Initial Access Setup)

After completing the installation block, access the web console page from your browser at `http://your_server_ip:8080`. The application will ask for an administrator secret token sequence.

#### 🖥️ Fetch Administrator Initialization Key from Server Console:

* **For Native Linux Setup:**
  ```bash
  sudo cat /var/jenkins_home/secrets/initialAdminPassword
  # Or look into alternative default directories if paths are updated
  sudo cat /var/lib/jenkins/secrets/initialAdminPassword
  ```

* **For Containerized Docker Setup:**
  ```bash
  docker exec -it myjenkins cat /var/jenkins_home/secrets/initialAdminPassword
  ```

Copy the printed text block and paste it straight inside the browser screen to unlock the setup console safely.

---

## 📋 Useful Jenkins System Administration Commands

Use these service controller instructions to manage native background instances directly over active execution runs:

### ⏹️ Managing Jenkins Daemon Processes
```bash
# Check running process logs
sudo systemctl status jenkins

# Stop the active server engine
sudo systemctl stop jenkins

# Boot up the background system service
sudo systemctl start jenkins

# Restart configuration lines and flush cache memory
sudo systemctl restart jenkins
```

---

## ⚠️ Common Production Mistakes & Troubleshooting
* **Nginx Port 8080 Collision Conflict:** Jenkins default web panel configuration binds to port **8080**. If alternative web applications (like Tomcat, Nginx proxies, or Spring Boot setups) reside on that exact port, Jenkins initialization calls will throw a `BindException`. Change ports inside config profiles if collision issues appear.
* **AWS EC2 Firewall Blocks:** Ensure your active AWS EC2 Security Group contains an explicit **Inbound Rule** authorizing incoming public traffic targeting TCP Port `8080`.

---

## 🧪 Quick Reference Verification Summary

| Command | Purpose | Expected Output Baseline |
| :--- | :--- | :--- |
| `java -version` | Check system runtime environment version | OpenJDK version "17.x.x" |
| `sudo systemctl is-active jenkins` | Check active status block value on console | active |
| `netstat -tulnp \| grep 8080` | Audit network interface port allocation loops | LISTEN on port 8080 |
| `docker ps` | View background active container lifecycles | Container running jenkins/jenkins:lts |

---

## 🎯 Final Concept
> **Jenkins Automation Server** remains the central processing backbone for enterprise software delivery pipelines. By decoupling development logic parameters through structural plugins and automating deployment workflows, engineering pipelines achieve unified, high-speed, repeatable CI/CD loops with low configuration drift overhead.
