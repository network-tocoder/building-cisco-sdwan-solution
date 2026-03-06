# ⚙️ Module 03 — Control Plane Bring-Up

[← EVE-NG Setup](../02-eve-ng-setup/README.md) | [Back to Main](../../README.md) | [Next: Edge Onboarding →](../04-edge-onboarding/README.md)

---

## 🎯 Learning Objectives

- Bootstrap vBond, vManage, and vSmart in the correct order
- Understand the certificate and authentication model
- Establish OMP control plane peering
- Verify control plane via CLI and GUI

---

## ⚠️ Bring-Up Order

```
1. vBond  →  2. vManage  →  3. vSmart  →  4. WAN Edges
```

---

## 3.1 vBond Bootstrap

```bash
vbond# config
vbond(config)# system
vbond(config-system)#  host-name         vBond
vbond(config-system)#  system-ip         10.0.0.12
vbond(config-system)#  site-id           100
vbond(config-system)#  organization-name "SDWAN-LAB"
vbond(config-system)#  vbond 10.0.0.12 local vbond-only

vbond(config)# vpn 0
vbond(config-vpn-0)#  interface eth0
vbond(config-interface-eth0)#   ip address 10.0.0.12/24
vbond(config-interface-eth0)#   no shutdown
vbond(config-interface-eth0)#   tunnel-interface
vbond(config-tunnel-interface)#   encapsulation ipsec
vbond(config-tunnel-interface)#   allow-service all

vbond(config)# vpn 512
vbond(config-vpn-512)#  interface eth1
vbond(config-interface-eth1)#   ip address 192.168.100.13/24
vbond(config-interface-eth1)#   no shutdown
vbond(config)# commit
```

---

## 3.2 vManage Bootstrap

```bash
vmanage# config
vmanage(config)# system
vmanage(config-system)#  host-name         vManage
vmanage(config-system)#  system-ip         10.0.0.10
vmanage(config-system)#  site-id           100
vmanage(config-system)#  organization-name "SDWAN-LAB"
vmanage(config-system)#  vbond 10.0.0.12

vmanage(config)# vpn 0
vmanage(config-vpn-0)#  interface eth0
vmanage(config-interface-eth0)#   ip address 10.0.0.10/24
vmanage(config-interface-eth0)#   no shutdown

vmanage(config)# vpn 512
vmanage(config-vpn-512)#  interface eth1
vmanage(config-interface-eth1)#   ip address 192.168.100.10/24
vmanage(config-interface-eth1)#   no shutdown
vmanage(config)# commit

# GUI: https://192.168.100.10:8443 → admin / admin
# Administration → Settings:
#   Organization Name = SDWAN-LAB
#   vBond Address     = 10.0.0.12
```

---

## 3.3 vSmart Bootstrap

```bash
vsmart# config
vsmart(config)# system
vsmart(config-system)#  host-name         vSmart-1
vsmart(config-system)#  system-ip         10.0.0.11
vsmart(config-system)#  site-id           100
vsmart(config-system)#  organization-name "SDWAN-LAB"
vsmart(config-system)#  vbond 10.0.0.12

vsmart(config)# vpn 0
vsmart(config-vpn-0)#  interface eth0
vsmart(config-interface-eth0)#   ip address 10.0.0.11/24
vsmart(config-interface-eth0)#   no shutdown
vsmart(config-interface-eth0)#   tunnel-interface
vsmart(config-tunnel-interface)#   encapsulation ipsec
vsmart(config-tunnel-interface)#   allow-service all

vsmart(config)# vpn 512
vsmart(config-vpn-512)#  interface eth1
vsmart(config-interface-eth1)#   ip address 192.168.100.11/24
vsmart(config-interface-eth1)#   no shutdown
vsmart(config)# commit
```

---

## 3.4 Verification

```bash
# Control connections (run on any WAN Edge or controller)
show sdwan control connections

# Expected: vBond, vSmart, vManage all show "up"
show sdwan omp peers
show sdwan omp summary
show sdwan certificate installed
```

---

## ✅ Checklist

- [ ] vBond responds to DTLS connections
- [ ] vManage GUI accessible on port 8443
- [ ] vSmart shows connected in vManage dashboard
- [ ] All `show sdwan control connections` entries = up
- [ ] Certificates installed on all controllers

---

[← EVE-NG Setup](../02-eve-ng-setup/README.md) | [Back to Main](../../README.md) | [Next: Edge Onboarding →](../04-edge-onboarding/README.md)
