# 📦 EKS Cluster Setup

### launch one instance

---

### 1: Install eksctl CLI tool for creating EKS Clusters on AWS
https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html

```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_\$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

---

### 2: Install kubectl
https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

```bash
curl -LO "https://dl.k8s.io/release/\$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
chmod +x kubectl
mkdir -p ~/.local/bin
mv ./kubectl ~/.local/bin/kubectl
kubectl version --client
```

---

### 3: Install AWS CLI on Ubuntu
https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

```bash
sudo apt install unzip -y
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

---

### 4: Configure AWS CLI

```bash
aws configure
```
*provide acesskey id and secrete acess key id aws eks permission required*

---

### 5: Create Amazon EKS cluster using eksctl

```bash
eksctl create cluster --name eks-cdec-b71 --region ap-south-1 --version 1.35 --nodegroup-name linux-nodes --node-type c7i-flex.large --nodes 1
```

---

### 6: Log In Into EKS cluster

```bash
aws eks update-kubeconfig --name eks-cdec-b71 --region ap-south-1
```

---

### =========To Delete Cluster =============

---

### 7: Delete EKS Cluster

```bash
eksctl delete cluster --name eks-cdec-b71 --region ap-south-1
```
