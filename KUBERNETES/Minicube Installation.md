## Minikube Installation
https://minikube.sigs.k8s.io/docs/start/?arch=%2Fmacos%2Fx86-64%2Fstable%2Fbinary+download

## Step 1: lanch an EC2 instance on Ubuntu (apt)
Instance Size: c7i-flex.large with 2 CPUs, 32 GB Storage <-- volume_size 

## Step 2 : Docker installation on EC2 
````
sudo apt update -y
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ubuntu
newgrp docker
sudo chmod 777 /var/run/docker.sock
````
## Install kubectl
Download the latest release with the command:
````
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
````

Install kubectl:
````
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
````
- use only when you did not have and root acess (sudo) on EC2 
````
chmod +x kubectl
mkdir -p ~/.local/bin
mv ./kubectl ~/.local/bin/kubectl
````
````
kubectl version --client    # to check version 
````
## Step 4: Install Minikube
````
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
````
````
sudo install minikube-linux-amd64 /usr/local/bin/minikube
````
````
minikube start
````

## your miniqube cluster is redy to use 
### test your project here 


````
minikube stop
````
🚀👀




---




- crete an pod file 
`vim pod.yaml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
  labels:
    app: nginx
spec:
  containers:
  - name: cont-1
    image: nginx
    ports:
    - containerPort: 80
```
`kubectl expose pod pod-1 --type=NodePort --port 80` 

---
## Expose deployment 

```bash
vim deploy.yaml
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
  minReadySeconds: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-pod-game
        image: mukunddeo9325/super-mario
        ports:
        - containerPort: 80
```

```
kubectl expose deployment my-app --type=LoadBalancer --port=80
```
## expose application on forwarded port 
```
kubectl port-forward --address 0.0.0.0 service/my-app 8081:80
```
#### check using Ec2 instance public IP and poert number 
`http://ec2_public_ip:8081`
