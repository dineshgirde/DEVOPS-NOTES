# 🗺️ Terraform Infrastructure as Code (IaC) — AWS VPC Setup

## 📚 Complete AWS Networking Blueprint via Terraform
### ⚡ Automated Provisioning of VPC, Subnets, Internet Gateway, NAT Gateway & Route Tables

---

## 🏗️ Infrastructure Architecture Preview

This code automates the creation of a production-ready, highly secure network isolation partition on AWS containing both Public and Private subnets.

```text
AWS Cloud
   │
   ▼
┌─────────────────────────────────────────────────────────────┐
│ ☁️ VPC: main (10.0.0.0/16)                                   │
│                                                             │
│   🚪 Internet Gateway (internet-getway)                    │
│                        │                                    │
│                        ▼                                    │
│   ┌─────────────────────────────────────────────────────┐   │
│   │ 🔹 Public Subnet: sub1 (10.0.0.0/22)                │   │
│   │                                                     │   │
│   │   ⚡ NAT Gateway  ◄─── [Binds Elastic IP]           │   │
│   │   🛣️ Route Table: Public-rt (0.0.0.0/0 ──► IGW)     │   │
│   └────────────────────▲────────────────────────────────┘   │
│                        │                                    │
│                        └──────────┐                         │
│                                   │ (Secure Outbound Only)  │
│   ┌───────────────────────────────┴─────────────────────┐   │
│   │ 🔒 Private Subnet: sub2 (10.0.4.0/22)               │   │
│   │                                                     │   │
│   │   💻 Isolated Workloads (App / DB)                  │   │
│   │   🛣️ Route Table: private-rt (0.0.0.0/0 ──► NAT)    │   │
│   └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Complete Terraform Infrastructure Deployment Manifest

Aap is block ke top-right corner se complete code ko directly **Copy** karke apni `main.tf` file me paste kar sakte hain. Saara core configuration aapke logic ke hisab se exact same rakha gaya hai:

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"

  tags = {
    Name = "main"
  }
}


resource "aws_subnet" "sub1" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.0.0/22"

  tags = {
    Name = "sub1"
  }
}

resource "aws_subnet" "sub2" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.4.0/22"

  tags = {
    Name = "sub2"
  }
}

resource "aws_internet_gateway" "i-gw" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "internet-getway"
  }
}

resource "aws_eip" "elastic" {
  domain = "vpc"
}

resource "aws_nat_gateway" "nat-getway" {
  allocation_id = aws_eip.elastic.id
  subnet_id     = aws_subnet.sub1.id


  tags = {
    name = "nat-getway"

  }
}

resource "aws_route_table" "rout-igw" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.i-gw.id
  }

  tags = {
    name = "Public-rt"

  }
}

resource "aws_route_table" "rout-nat" {
  vpc_id = aws_vpc.main.id


  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_nat_gateway.nat-getway.id
  }

  tags = {
    name = "private-rt"

  }
}

resource "aws_route_table_association" "tr-as" {
  subnet_id      = aws_subnet.sub1.id
  route_table_id = aws_route_table.rout-igw.id
}



resource "aws_route_table_association" "rt-nat" {
  subnet_id      = aws_subnet.sub2.id
  route_table_id = aws_route_table.rout-nat.id
}
```

---

## 🚀 Step-by-Step Infrastructure Deployment Execution Lifecycle

Is configuration ko active AWS account me apply karne ke liye apni terminal baseline workspace me niche diye gaye commands series me execute karein:

### 📦 1. Workspace Initialization
```bash
terraform init
```
* **Why?** Deploys the official AWS provider plugin infrastructure schemas required to interpret this resource manifest card block.

### 🔍 2. Syntax & Grammar Verification Audit
```bash
terraform validate
```
* **Why?** Runs static code evaluation across configuration boundaries to eliminate block formatting loops.

### 📐 3. Code Alignment Canonical Normalization
```bash
terraform fmt
```
* **Why?** Automatically formats and aligns every attribute line cleanly according to official HCL styling guidelines.

### 📋 4. Execution Plan Dry-Run Blueprint Preview
```bash
terraform plan
```
* **Why?** Simulates a dry run against active cloud environments to output an exact summary mapping of variables to be generated.

### ⚡ 5. Real-Time Resource Provisioning Deployment
```bash
terraform apply -auto-approve
```
* **Why?** Instantly executes plan definitions to construct your isolation tiers without blocking for terminal interactions.

### 🧹 6. Complete Infrastructure Deletion Purge
```bash
terraform destroy -auto-approve
```
* **Why?** Deletes all active tracking resources instantly from the targeted cloud platform provider once validation concludes.
