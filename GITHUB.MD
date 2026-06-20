# 🌐 GitHub Cheatsheet

## 🚀 Complete GitHub Guide for Beginners to Advanced Users
### 🔄 Learn GitHub Repositories, Branching, Pull Requests, GitHub Actions, Security, CI/CD & Best Practices

---

## 📚 Table of Contents
1. Introduction to GitHub
2. Getting Started with GitHub
3. Basic GitHub Operations
4. Branching and Merging
5. GitHub Issues and Projects
6. GitHub Actions (CI/CD)
7. GitHub Packages
8. Advanced GitHub Features
9. GitHub Security
10. GitHub CLI (gh)
11. GitHub API
12. GitHub Best Practices
13. References

---

## 🚀 1. Introduction to GitHub

### 🔹 What is GitHub?
GitHub is a web-based platform that uses Git for version control and collaboration.

It helps developers:
* Store source code
* Track changes
* Collaborate with teams
* Automate CI/CD workflows
* Manage projects

### 🔑 Key Features of GitHub

| Feature | Description |
| :--- | :--- |
| **Git Repositories** | Host and manage source code |
| **Pull Requests** | Review and merge code changes |
| **GitHub Actions** | Automate CI/CD workflows |
| **Issues & Projects** | Project management and tracking |
| **Security Features** | Secret scanning and vulnerability alerts |
| **GitHub Packages** | Store packages and Docker images |
| **GitHub CLI** | Manage GitHub from terminal |

### 🔹 Important GitHub Definitions

| Term | Definition |
| :--- | :--- |
| **Repository (Repo)** | Storage location for source code |
| **Commit** | Snapshot of code changes |
| **Branch** | Separate line of development |
| **Pull Request (PR)** | Request to merge code changes |
| **Fork** | Copy of another repository |
| **Clone** | Download repository locally |
| **Merge** | Combine changes from branches |
| **Issue** | Task, bug, or feature request |
| **Workflow** | Automated GitHub Actions process |
| **Runner** | Machine that executes workflows |
| **Secret** | Encrypted sensitive variable |
| **Release** | Packaged application version |
| **Tag** | Marker for a specific commit |

---

## 🛠️ 2. Getting Started with GitHub

### 🔹 Create GitHub Account
Create account on GitHub platform and configure your profile.

### 🔹 Generate SSH Key
```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

### 🔹 Add SSH Key to GitHub
Go to:
* Settings → SSH and GPG Keys → New SSH Key

### 🔹 Clone Repository
```bash
git clone git@github.com:username/repository.git
```

### 🔹 Create New Repository
Steps:
1. Go to Repositories
2. Click New Repository
3. Add repository name
4. Choose Public or Private
5. Initialize README if needed

---

## ⚙️ 3. Basic GitHub Operations

### 🔹 Configure Git Username
```bash
git config --global user.name "Your Name"
```

### 🔹 Configure Git Email
```bash
git config --global user.email "you@example.com"
```

### 🔹 Check Git Status
```bash
git status
```

### 🔹 Add Files
```bash
git add .
```

### 🔹 Commit Changes
```bash
git commit -m "Initial commit"
```

### 🔹 Push Changes
```bash
git push origin main
```

### 🔹 Pull Latest Changes
```bash
git pull origin main
```

### 🔹 View Commit History
```bash
git log
```

---

## 🌿 4. Branching and Merging

### 🔹 Create New Branch
```bash
git checkout -b feature-branch
```

### 🔹 Switch Branch
```bash
git checkout main
```

### 🔹 Push Branch
```bash
git push origin feature-branch
```

### 🔹 Merge Branch
```bash
git merge feature-branch
```

### 🔹 Delete Local Branch
```bash
git branch -d feature-branch
```

### 🔹 Delete Remote Branch
```bash
git push origin --delete feature-branch
```

### 🔹 Pull Request Workflow
Steps:
1. Push branch
2. Open Pull Request
3. Review code
4. Merge changes
5. Delete branch

---

## 📋 5. GitHub Issues and Projects

### 🔹 Create Issue
Go to:
* Repository → Issues → New Issue

### 🔹 Link PR with Issue
```text
Fixes #10
Closes #15
```

### 🔹 Labels in Issues
Used for:
* Bug tracking
* Feature requests
* Priority management

### 🔹 GitHub Projects
Used for:
* Agile workflows
* Kanban boards
* Sprint tracking

### 🔹 Milestones
Milestones help group issues for releases or project goals.

---

## 🚀 6. GitHub Actions (CI/CD)

### 🔹 What is GitHub Actions?
GitHub Actions is a CI/CD automation platform used to automate workflows directly from GitHub repositories.

### 🔹 Workflow File Location
`.github/workflows/`

### 🔹 Sample Workflow
```yaml
name: CI Pipeline

on:
  push:
    branches:
      - main

jobs:

  build:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Install Dependencies
        run: npm install

      - name: Run Tests
        run: npm test
```

### 🔹 Workflow Triggers

| Trigger | Description |
| :--- | :--- |
| **push** | Trigger on code push |
| **pull_request** | Trigger on pull request |
| **schedule** | Trigger using cron jobs |
| **workflow_dispatch** | Manual trigger |

### 🔹 GitHub Secrets
Go to:
* Settings → Secrets and Variables → Actions

Used for storing:
* AWS Keys
* Docker Passwords
* API Tokens

### 🔹 Example Using Secrets
```yaml
env:
  AWS_ACCESS_KEY_ID: \${{ secrets.AWS_ACCESS_KEY_ID }}
```

---

## 📦 7. GitHub Packages

### 🔹 GitHub Container Registry Login
```bash
docker login ghcr.io -u USERNAME -p TOKEN
```

### 🔹 Build Docker Image
```bash
docker build -t ghcr.io/username/image-name:tag .
```

### 🔹 Push Docker Image
```bash
docker push ghcr.io/username/image-name:tag
```

### 🔹 Install GitHub Package
```json
"dependencies": {
  "package-name": "github:username/repository"
}
```

---

## 🚀 8. Advanced GitHub Features

### 🔹 Branch Protection Rules
Go to:
* Settings → Branches

Features:
* Prevent force push
* Require pull request reviews
* Require status checks

### 🔹 Dependabot
Dependabot automatically:
* Scans dependencies
* Detects vulnerabilities
* Creates update PRs

### 🔹 Code Owners
`CODEOWNERS`
Used to assign reviewers automatically.

### 🔹 GitHub Webhooks
Webhooks notify external systems when events occur in GitHub repositories.

### 🔹 GitHub Apps
Used to extend GitHub functionality and automation.

---

## 🔒 9. GitHub Security

### 🔹 Enable 2FA
Go to:
* Settings → Password and Authentication

### 🔹 Secret Scanning
GitHub scans repositories for leaked secrets automatically.

### 🔹 Code Scanning
Example:
```yaml
- name: CodeQL Analysis
  uses: github/codeql-action/analyze@v2
```

### 🔹 Dependency Scanning
GitHub detects vulnerable dependencies using Dependabot alerts.

### 🔹 Security Best Practices
* Enable branch protection
* Use least privilege access
* Rotate tokens regularly
* Use secrets instead of hardcoding passwords

---

## 💻 10. GitHub CLI (gh)

### 🔹 Install GitHub CLI

**macOS**
```bash
brew install gh
```

**Linux**
```bash
sudo apt install gh
```

### 🔹 Authenticate CLI
```bash
gh auth login
```

### 🔹 Clone Repository
```bash
gh repo clone username/repository
```

### 🔹 Create Pull Request
```bash
gh pr create --title "Feature Update" --body "Details"
```

### 🔹 List Issues
```bash
gh issue list
```

---

## 🔗 11. GitHub API

### 🔹 Access GitHub API
```bash
curl -H "Authorization: token YOUR_TOKEN" \
https://api.github.com/user/repos
```

### 🔹 Create Issue Using API
```bash
curl -X POST \
-H "Authorization: token YOUR_TOKEN" \
-H "Content-Type: application/json" \
-d '{"title":"Bug Report","body":"Bug details"}' \
https://api.github.com/repos/username/repository/issues
```

---

## ✅ 12. GitHub Best Practices

### 🔹 Use Meaningful Commit Messages
Example:
* Fix login authentication bug

### 🔹 Protect Main Branch
Enable:
* PR reviews
* Status checks
* Restricted pushes

### 🔹 Automate CI/CD
Use GitHub Actions for:
* Testing
* Building
* Deployment

### 🔹 Use Pull Requests
Benefits:
* Code review
* Collaboration
* Better tracking

### 🔹 Organize Issues Properly
Use:
* Labels
* Milestones
* Assignees
* Projects

---

## 📚 13. References

### 🔹 Official Documentation
* GitHub Documentation
* GitHub Actions Documentation
* GitHub CLI Documentation

### 🔹 Community Resources
* GitHub Blog
* GitHub Marketplace
* GitHub Learning Lab

---

## 🎯 Final Takeaway
GitHub is more than just a code hosting platform.

It helps teams:
* Collaborate efficiently
* Automate CI/CD
* Secure repositories
* Track projects
* Improve software delivery workflows

🌐 GitHub Powers Modern DevOps & Collaboration 🚀
