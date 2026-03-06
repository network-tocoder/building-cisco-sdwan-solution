# 🔌 Module 06 — Underlay & Transport

[← 05-vmanage-templates](../05-vmanage-templates/README.md) | [Back to Main](../../README.md) | [Next: 07-service-vpn →](../07-service-vpn/README.md)

---

## 🎯 Learning Objectives

- VPN 0 (Transport VPN) design principles
- Dual-uplink: MPLS + Internet configuration
- TLOC concept and color assignment
- Tunnel interface configuration (IPSec/GRE)
- BFD Hello timers and path liveliness
- NAT traversal for vBond reachability

---

## 🔗 Quick Reference Commands

```bash
show sdwan control connections
show sdwan omp peers
show sdwan bfd sessions
show sdwan omp routes
show sdwan ip route vpn 1
show sdwan policy from-vsmart
show sdwan tunnel statistics
```

---

## 📁 Related Config Files

- [configs/day0-bootstrap/](../../configs/day0-bootstrap/)
- [configs/feature-templates/](../../configs/feature-templates/)
- [configs/policy-templates/](../../configs/policy-templates/)

---

## 🤖 Automation for This Module

| Tool | Link |
|------|------|
| Python | [automation/python/](../../automation/python/README.md) |
| Ansible | [automation/ansible/](../../automation/ansible/README.md) |
| Postman | [automation/postman/](../../automation/postman/README.md) |
| Terraform | [automation/terraform/](../../automation/terraform/README.md) |

---

[← 05-vmanage-templates](../05-vmanage-templates/README.md) | [Back to Main](../../README.md) | [Next: 07-service-vpn →](../07-service-vpn/README.md)
