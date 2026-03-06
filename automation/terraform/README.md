# 🟪 Automation — Terraform for Cisco SD-WAN

[← Back to Main](../../README.md) | [Python →](../python/README.md) | [Ansible →](../ansible/README.md) | [Postman →](../postman/README.md)

---

## 📦 Installation

```bash
terraform init
terraform --version  # 1.5+
```

---

## ⚙️ Provider Config

```hcl
# main.tf
terraform {
  required_providers {
    catalystwan = {
      source  = "CiscoDevNet/catalystwan"
      version = "~> 0.3"
    }
  }
}

provider "catalystwan" {
  url      = var.vmanage_url
  username = var.vmanage_user
  password = var.vmanage_password
}

resource "catalystwan_system_feature_template" "branch_system" {
  name         = "Branch-System-Template"
  description  = "System template for branch vEdge routers"
  device_types = ["vedge-cloud"]
  hostname     = "{{system_hostname}}"
  system_ip    = "{{system_ip}}"
  site_id      = "{{site_id}}"
  organization = var.org_name
  vbond_address = var.vbond_ip
}

resource "catalystwan_device_template" "branch_vedge" {
  name        = "Branch-vEdge-Full-Template"
  description = "Complete device template for branch vEdge routers"
  device_type = "vedge-cloud"
  general_templates = [
    {
      template_id   = catalystwan_system_feature_template.branch_system.id
      template_type = "system-vedge"
    }
  ]
}
```

---

## 📊 Variables

```hcl
# variables.tf
variable "vmanage_url"      { default = "https://192.168.100.10:8443" }
variable "vmanage_user"     { default = "admin" }
variable "vmanage_password" { sensitive = true }
variable "org_name"         { default = "SDWAN-LAB" }
variable "vbond_ip"         { default = "10.0.0.12" }
```

---

## 🚀 Usage

```bash
terraform init
terraform plan
terraform apply
terraform destroy
```

Add to `.gitignore`:
```
terraform.tfvars
*.tfstate
.terraform/
```

---

## 💡 Why Terraform for SD-WAN?

| Traditional (GUI) | Terraform (IaC) |
|------------------|-----------------|
| Manual clicks | Declarative code |
| No audit trail | Git history = full audit |
| Hard to reproduce | Version controlled |
| Error-prone | Idempotent & testable |

---

[← Back to Main](../../README.md) | [Python →](../python/README.md)
