# 🖥️ Module 02 — EVE-NG Lab Setup

[← Architecture](../01-architecture/README.md) | [Back to Main](../../README.md) | [Next: Control Plane →](../03-control-plane/README.md)

---

## 🎯 Learning Objectives

- Install and configure EVE-NG
- Import Cisco SD-WAN images
- Design and deploy the lab topology
- Configure management networking

---

## 2.1 EVE-NG Installation

```bash
# After first boot (default: root / eve)
apt update && apt upgrade -y
apt install eve-ng-guacamole -y
```

### Host Requirements

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| RAM | 32 GB | 64 GB |
| CPU | 8 cores | 16 cores |
| Disk | 200 GB SSD | 500 GB NVMe |

---

## 2.2 Importing SD-WAN Images

```bash
# SCP images to EVE-NG
scp vmanage-*.qcow2 root@<EVE-IP>:/opt/unetlab/addons/qemu/

# Create image directories
mkdir -p /opt/unetlab/addons/qemu/vmanage-20.9.1
mv vmanage-20.9.1.qcow2 /opt/unetlab/addons/qemu/vmanage-20.9.1/hda.qcow2

mkdir -p /opt/unetlab/addons/qemu/vsmart-20.9.1
mv vsmart-20.9.1.qcow2 /opt/unetlab/addons/qemu/vsmart-20.9.1/hda.qcow2

mkdir -p /opt/unetlab/addons/qemu/vbond-20.9.1
mv vbond-20.9.1.qcow2 /opt/unetlab/addons/qemu/vbond-20.9.1/hda.qcow2

mkdir -p /opt/unetlab/addons/qemu/vedge-cloud-20.9.1
mv vedge-cloud-20.9.1.qcow2 /opt/unetlab/addons/qemu/vedge-cloud-20.9.1/hda.qcow2

# Fix permissions
/opt/unetlab/wrappers/unl_wrapper -a fixpermissions
```

---

## 2.3 Node Resource Allocation

| Node | vCPU | RAM | Notes |
|------|------|-----|-------|
| vManage | 2 | 8 GB | Scale up for production |
| vSmart | 2 | 4 GB | Deploy 2 for HA |
| vBond | 1 | 2 GB | Must be publicly reachable |
| vEdge Cloud | 1 | 2 GB | Per branch/site |
| CSR1000v | 1 | 4 GB | IOS-XE SD-WAN mode |

---

## 2.4 Management Network

```
EVE-NG Cloud0 → 192.168.100.0/24
  vManage   192.168.100.10
  vSmart-1  192.168.100.11
  vSmart-2  192.168.100.12
  vBond     192.168.100.13
  vEdge-HQ  192.168.100.20
  vEdge-BR1 192.168.100.21
  vEdge-BR2 192.168.100.22
  cEdge-DC1 192.168.100.23
```

---

## ✅ Checklist

- [ ] EVE-NG installed and licensed
- [ ] All images imported and permissions fixed
- [ ] Lab topology created in EVE-NG
- [ ] Management network reachable
- [ ] All nodes boot successfully

---

[← Architecture](../01-architecture/README.md) | [Back to Main](../../README.md) | [Next: Control Plane →](../03-control-plane/README.md)
