# 🚀 Terraform CLI Cheatsheet

## 📚 Complete Terraform Command Guide (Beginner → Production)
### ⚡ Infrastructure as Code (IaC) Automation Service

---

## 🔹 Core Workflow Commands

### 🔹 `terraform init`
Initialize a working directory containing Terraform configuration files.
* **What it does:** Downloads provider plugins (AWS, Azure, GCP, etc.) and sets up the backend storage configuration.

### 🔹 `terraform validate`
Validate the configuration files in a directory.
* **What it does:** Checks whether the syntax, attributes, and types inside the `.tf` files are accurate before execution.

### 🔹 `terraform fmt`
Reformat configuration files to a canonical format and style.
* **What it does:** Automatically fixes the alignment, indentation, and spacing across all configuration files.

### 🔹 `terraform plan`
Generate and show an execution plan.
* **What it does:** Gives a preview blueprint of the infrastructure resources that are going to be created, modified, or destroyed.

### 🔹 `terraform apply`
Builds or changes infrastructure.
* **What it does:** Executes the actions proposed in a terraform plan. Requires manual confirmation entry.

### 🔹 `terraform apply -auto-approve`
Apply the changes and skip interactive approval.
* **What it does:** Automatically approves the execution blueprint and pushes configuration updates instantly.

### 🔹 `terraform destroy -auto-approve`
Destroy all remote infrastructure managed by the configuration and approve it.
* **What it does:** Purges and deletes all tracked cloud infrastructure assets instantly without manual confirmation.

---

## 🔥 Advanced & Important Production Commands

### 🔹 `terraform state list`
List resources managed by the current state file.
* **What it does:** Displays every single resource currently tracked in your infrastructure footprint directory.

### 🔹 `terraform show`
Show human-readable output from a state or plan file.
* **What it does:** Inspects the current state database to show the exact live attributes of your active cloud assets.

### 🔹 `terraform refresh`
Update the state file against real-world resources.
* **What it does:** Queries the cloud provider directly to discover any drift or manual changes done outside Terraform.

### 🔹 `terraform output`
Read an output variable from a state file.
* **What it does:** Extracts and prints user-defined output values (like instance public IPs or database endpoints).

### 🔹 `terraform taint`
Manually mark a resource for recreation during the next apply.
* **What it does:** Forces Terraform to delete and recreate a specific corrupted or modified resource on the next run.
* *Example:* `terraform taint aws_instance.my_app`

### 🔹 `terraform untaint`
Manually unmark a tainted resource.
* **What it does:** Cancels the recreation instruction if the resource was marked as tainted by mistake.

### 🔹 `terraform import`
Associate existing cloud infrastructure with a Terraform resource block.
* **What it does:** Brings resources created manually via the AWS Console under Terraform management.
* *Example:* `terraform import aws_instance.web i-1234567890abcdef0`

### 🔹 `terraform graph`
Generate a visual graph of Terraform resources.
* **What it does:** Outputs a DOT-formatted graph configuration showing resource dependencies.

### 🔹 `terraform workspace list`
List existing configuration workspaces.
* **What it does:** Shows all active working environments (e.g., `dev`, `staging`, `prod`) inside the same directory layout.

### 🔹 `terraform workspace new`
Create a clean isolated configuration workspace.
* **What it does:** Spawns a separate state layer to isolate code variables across target deployment platforms.
* *Example:* `terraform workspace new dev`

---

## 🧪 Quick Reference Summary

| Command | Purpose | Requires Approval |
| :--- | :--- | :--- |
| `terraform init` | Initialize folder & download plugins | No |
| `terraform validate` | Syntax & attribute validation check | No |
| `terraform fmt` | Fix formatting & code indentation | No |
| `terraform plan` | Blueprint dry-run preview check | No |
| `terraform apply` | Build cloud resources | **Yes** |
| `terraform apply -auto-approve` | Fast-track resource generation | No |
| `terraform destroy -auto-approve` | Fast-track infrastructure purging | No |
| `terraform state list` | Print all tracked infra items | No |
| `terraform output` | Show exposed tracking parameters | No |

---

## 🎯 Final Concept
> **Terraform CLI** is the control panel for Infrastructure as Code. By mastering the sequence from initialization (`init`) to state management (`state`), engineering teams can safely plan, model, validate, scale, and destroy large-scale cloud operations without drifting configurations.
