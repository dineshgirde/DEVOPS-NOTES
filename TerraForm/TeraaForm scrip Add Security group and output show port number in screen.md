# 🛡️ Terraform Infrastructure as Code (IaC) — AWS Security Group Setup

## 📚 Complete AWS Security Group Blueprint via Terraform
### ⚡ Automated Provisioning of Security Groups, Multi-Port Ingress Rules & Global Egress Routing

---

## 🏗️ Security Architecture Overview

This automation script provisions an AWS Security Group acting as a stateful firewall instance configured with granular Layer 4 rule isolation wrappers.

```text
Incoming Traffic (Internet)
   │
   ├──► Port 22  (SSH Access)       ──► [ Allowed from Everywhere: 0.0.0.0/0 ]
   ├──► Port 80  (HTTP Web Traffic) ──► [ Allowed from Everywhere: 0.0.0.0/0 ]
   └──► Ports 81-90 (App Range)     ──► [ Allowed from Everywhere: 0.0.0.0/0 ]
   │
┌──┴──────────────────────────────────────────────────────────┐
│ 🛡️ Stateful Firewall: test-sg (vpc-08c40d91b76b9b4a0)       │
└──┬──────────────────────────────────────────────────────────┘
   │
   └──► Global Egress (All Ports)   ──► [ Outbound Authorized to Anywhere ]
```

---

## 🛠️ Complete Terraform Deployment Manifest with Outputs

You can click the **Copy** button on the top-right corner of this block to paste it directly into your `main.tf` file. 

All your original blocks have been kept **exactly same to same**, and a single `output` block has been attached at the bottom to show the allowed port ranges directly on your terminal screen:

```hcl
resource "aws_security_group" "sg" {
  name        = "test-sg"
  description = "to test sg in terraform "
  vpc_id      = "vpc-08c40d91b76b9b4a0"

  tags = {
    Name = "test-sg-tag"
  }
}


resource "aws_vpc_security_group_ingress_rule" "inbound" {
  security_group_id = aws_security_group.sg.id

  cidr_ipv4   = "0.0.0.0/0"
  from_port   = 22
  ip_protocol = "tcp"
  to_port     = 22
}

resource "aws_vpc_security_group_ingress_rule" "inboun" {
  security_group_id = aws_security_group.sg.id

  cidr_ipv4   = "0.0.0.0/0"
  from_port   = 80
  ip_protocol = "tcp"
  to_port     = 80
}


resource "aws_vpc_security_group_ingress_rule" "inboun3" {
  security_group_id = aws_security_group.sg.id

  cidr_ipv4   = "0.0.0.0/0"
  from_port   = 81
  ip_protocol = "tcp"
  to_port     = 90
}


resource "aws_vpc_security_group_egress_rule" "out" {
  security_group_id = aws_security_group.sg.id
  cidr_ipv4          = "0.0.0.0/0"
  ip_protocol       = "-1" # semantically equivalent to all ports
}

# ==========================================
# 🔥 OUTPUTS MAPPING BLOCK (Show Ports)
# ==========================================

output "allowed_ports" {
  description = "Lists the configured inbound port rules on the console screen"
  value       = [
    "Allowed Port: ${aws_vpc_security_group_ingress_rule.inbound.from_port}",
    "Allowed Port: ${aws_vpc_security_group_ingress_rule.inboun.from_port}",
    "Allowed Ports Range: ${aws_vpc_security_group_ingress_rule.inboun3.from_port} to ${aws_vpc_security_group_ingress_rule.inboun3.to_port}"
  ]
}
```

---

## 🚀 Execution & Tracking Lifecycle

Use the standard execution sequence commands to build the resources and review the terminal screen outputs:

### 📦 1. Initialize Plugin Directory
```bash
terraform init
```

### 📐 2. Canonical Code Alignment
```bash
terraform fmt
```

### 📋 3. Blueprint Change Inspection
```bash
terraform plan
```

### ⚡ 4. Real-Time Resource Provisioning
```bash
terraform apply -auto-approve
```

#### 🖥️ Expected Outputs Display on Screen (Appears at the bottom after Apply completes):
```text
Apply complete! Resources: 4 added, 0 changed, 0 destroyed.

Outputs:

allowed_ports = [
  "Allowed Port: 22",
  "Allowed Port: 80",
  "Allowed Ports Range: 81 to 90",
]
```

---

## 💡 Quick Rules Reference Summary

| Rule Resource Reference | Traffic Flow | Port Configuration | Target Protocol | Network Source/Dest |
| :--- | :--- | :--- | :--- | :--- |
| `inbound` | Inbound (Ingress) | **22** | TCP | `0.0.0.0/0` (Anywhere) |
| `inboun` | Inbound (Ingress) | **80** | TCP | `0.0.0.0/0` (Anywhere) |
| `inboun3` | Inbound (Ingress) | **81 - 90** | TCP | `0.0.0.0/0` (Anywhere) |
| `out` | Outbound (Egress) | **All Ports** | `-1` (All) | `0.0.0.0/0` (Anywhere) |

---

## 🎯 Final Concept
> **Terraform Stateful Security Groups** decoupling definitions secure applications from low-level access loops. By abstracting standalone rule objects via `aws_vpc_security_group_ingress_rule`, you protect live operations from full security block flushes during manual system configuration updates.
