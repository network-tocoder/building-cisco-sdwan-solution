# 🔀 Module 07 — Service VPN & Segmentation

[← 06-underlay-transport](../06-underlay-transport/README.md) | [Back to Main](../../README.md) | [Next: 08-routing →](../08-routing/README.md)

---

## 🎯 Learning Objectives

- VPN 1 service VPN configuration
- Multi-VPN / VRF segmentation across the WAN
- Interface assignment to service VPNs
- DHCP server config on vEdge
- Connected and static routes in OMP
- Inter-VPN route leaking

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

[← 06-underlay-transport](../06-underlay-transport/README.md) | [Back to Main](../../README.md) | [Next: 08-routing →](../08-routing/README.md)
