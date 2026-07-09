# Introduction to Terraform 🌍

[Terraform](https://github.com/hashicorp/terraform) is an open-source **Infrastructure as Code (IaC)** software tool created by [HashiCorp](https://developer.hashicorp.com/terraform/intro). It allows you to safely and predictably define, provision, and manage cloud infrastructure using a declarative configuration language.

Instead of manually clicking through cloud consoles, you write configuration files that describe your desired infrastructure components.

---

## 🚀 Key Features

* **Infrastructure as Code (IaC):** Describe your datacenter infrastructure using the high-level HashiCorp Configuration Language (HCL).
* **Execution Plans:** Terraform generates a "planning" step before building, showing exactly what will be added, changed, or destroyed.
* **Resource Graph:** It builds a visual dependency graph to create or modify non-dependent resources simultaneously, maximizing speed.
* **Change Automation:** Complex modifications can be safely applied to your live infrastructure with minimal human error.

---

## 🛠️ Supported Providers

Terraform interacts with cloud services through plugins called **Providers**. It supports thousands of public platforms out-of-the-box:
* **Cloud Platforms:** AWS, Google Cloud (GCP), Microsoft Azure
* **Orchestration:** Kubernetes, Helm, Docker
* **SaaS/Tools:** GitHub, Datadog, Splunk

---

## 📂 Core Workflow Commands

The standard lifecycle of any Terraform setup follows these 4 steps:

```bash
# 1. Initialize the workspace and download required cloud providers
terraform init

# 2. Preview the changes that Terraform plans to make
terraform plan

# 3. Create or update your real-world cloud infrastructure
terraform apply

# 4. Permanently destroy all infrastructure tracked by the configuration
terraform destroy
```

---

## 📐 Typical File Structure

A standard Terraform deployment consists of plain text files ending in a `.tf` extension:

* `main.tf` — Core logic file where your infrastructure resources are defined.
* `variables.tf` — Contains definitions for input arguments to customize deployment variables.
* `outputs.tf` — Exposes values of your infrastructure (like IP addresses) after application.
* `providers.tf` — Defines required cloud provider plugins and their configuration parameters.
* `terraform.tfvars` — File where you assign values to your declared input variables.

---

## 💾 Core Concept: Terraform State

Terraform records the exact state of your managed infrastructure in a file named `terraform.tfstate`.

* **The Purpose:** Maps real-world resources to your configuration files and tracks metadata.
* **Local State:** Stored by default on your computer as a plain text JSON file.
* **Remote State:** Best practice for teams. Stores state in remote storage (AWS S3, Azure Blob, Google Cloud Storage, or Terraform Cloud).
* **State Locking:** Prevents multiple team members from running configurations at the same time to avoid data corruption.

---

## 📦 Reusability: Modules

Modules are self-contained packages of Terraform configurations that managed tasks together.
* **Root Module:** The folder containing your primary `.tf` files.
* **Child Module:** External packages you call from your root module to keep code DRY (Don't Repeat Yourself).
* **Source:** Can be pulled from local folders, GitHub repositories, or the official public Terraform Registry.

---

## ⚙️ Advanced Commands Reference

Beyond the core workflow, you will frequently use these utilities:

```bash
# Format your .tf files to meet standard layout specifications
terraform fmt

# Check whether your configuration is syntactically valid and internal-consistent
terraform validate

# Manage workspaces to isolate different environments (e.g., dev, staging, prod)
terraform workspace list

# Manually pull a resource into your Terraform state file that was created elsewhere
terraform import <resource_address> <cloud_id>
```

---

## 📝 Best Practices Checklist

* [ ] **Never commit secrets:** Pass passwords/keys via environment variables, secret managers, or variable files excluded in `.gitignore`.
* [ ] **Always use Remote State:** Use backend locking mechanisms (like AWS DynamoDB) for team environments.
* [ ] **Lock provider versions:** Specify exact cloud provider versions to prevent breaking changes during updates.
* [ ] **Run `fmt` and `validate`:** Integrate formatting and validation tools directly into your CI/CD pipelines.
