# 🚀 Jenkins Installation Guide

## 📚 Native Package Installation Lifecycle on Debian-based Systems (Ubuntu)
### ⚡ Automated Provisioning of JRE Runtime Environment and Verified Stable Package Repositories

---

```bash
sudo apt update
sudo apt install fontconfig openjdk-21-jre
java -version
```

---

### 🔹 Long Term Support release
A LTS (Long-Term Support) release is chosen every 12 weeks from the stream of regular releases as the stable release for that time period. It can be installed from the debian-stable apt repository.

---

```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins 
```
