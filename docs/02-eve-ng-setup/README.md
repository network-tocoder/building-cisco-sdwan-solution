# 🖥️ Module 02 — EVE-NG Lab Setup (Complete Guide)

[← Architecture](../01-architecture/README.md) | [Back to Main](../../README.md) | [Next: Control Plane →](../03-control-plane/README.md)

---

## 🎯 Learning Objectives

- Install EVE-NG Community or Pro on a bare-metal/VM host
- Upload and correctly configure all Cisco SD-WAN QEMU images
- Fix the vManage secondary storage disk issue (REQUIRED)
- Build the full lab topology with correct interface wiring
- Configure bootstrap settings on all 4 device types
- Validate IP reachability before moving to certificates

---

## 📺 Video Timestamps

| Topic | Time |
|-------|------|
| EVE-NG installation overview | 0:00 |
| SD-WAN image upload + folder naming | 3:30 |
| Fix vManage secondary disk | 7:00 |
| Node settings (RAM/CPU/NICs) | 9:30 |
| Building the topology + wiring | 11:00 |
| vManage first boot + config | 16:00 |
| vSmart / vBond / vEdge config | 22:00 |
| Reachability verification | 28:00 |

---

## 2.1 EVE-NG Edition Comparison

| Feature | Community (Free) | Pro (Paid) |
|---------|-----------------|-----------|
| Node limit | ~63 nodes | Unlimited |
| SD-WAN QEMU images | ✅ Supported | ✅ Supported |
| Multi-user | ❌ | ✅ |
| HTML5 console (Guacamole) | ✅ | ✅ |
| Wireshark packet capture | ✅ | ✅ |
| Best for | Personal lab | Team / Training |

> 💡 **Community edition is sufficient** for this entire lab series.

---

## 2.2 Host System Requirements

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| RAM | 16 GB | 32–64 GB |
| CPU | 8 vCPUs | 16 vCPUs |
| Disk | 100 GB SSD | 500 GB NVMe |
| OS | Ubuntu 20.04 | Ubuntu 22.04 |
| Nested Virtualization | ✅ Required | ✅ Required |

**Node RAM breakdown for this lab:**

| Node | RAM |
|------|-----|
| vManage | 8 GB (minimum) |
| vSmart | 2 GB |
| vBond | 2 GB |
| vEdge × 4 | 4 GB (1 GB each) |
| vIOS-L2 switch | 1 GB |
| **Total** | **~17 GB** |

---

<details>
<summary>📦 Step 1 — Upload SD-WAN Images to EVE-NG</summary>

```bash
# ── Check what SD-WAN template prefix EVE-NG expects ─────────────
ls /opt/unetlab/html/templates/intel/ | grep -i vt
# Expected output:
#   vtbond.yml
#   vtedge.yml
#   vtmgmt.yml
#   vtsmart.yml

# Inspect a template to confirm settings
cat /opt/unetlab/html/templates/intel/vtmgmt.yml | grep -E "name|description|image"

# ── Create image directories (MUST match template prefix!) ────────
# ⚠️  Use "vt" prefix — NOT "viptela"
# EVE-NG templates expect exactly "vt" as the folder prefix
cd /opt/unetlab/addons/qemu/

mkdir -p vtmgmt-20.3.1      # vManage
mkdir -p vtsmart-20.3.1     # vSmart
mkdir -p vtbond-20.3.1      # vBond (separate image from vEdge)
mkdir -p vtedge-20.3.1      # WAN Edge (vEdge Cloud)

# ── Transfer images via SCP from your local machine ───────────────
# Disk image filename MUST be renamed to exactly: hda.qcow2

# scp viptela-vmanage-20.3.1.qcow2  root@<eve-ip>:/opt/unetlab/addons/qemu/vtmgmt-20.3.1/hda.qcow2
# scp viptela-smart-20.3.1.qcow2    root@<eve-ip>:/opt/unetlab/addons/qemu/vtsmart-20.3.1/hda.qcow2
# scp viptela-bond-20.3.1.qcow2     root@<eve-ip>:/opt/unetlab/addons/qemu/vtbond-20.3.1/hda.qcow2
# scp viptela-edge-20.3.1.qcow2     root@<eve-ip>:/opt/unetlab/addons/qemu/vtedge-20.3.1/hda.qcow2

# Windows users: Use WinSCP — drag-drop to the paths above

# ── Fix permissions (ALWAYS run after adding images) ──────────────
/opt/unetlab/wrappers/unl_wrapper -a fixpermissions

# ── Verify: 4 folders, each containing hda.qcow2 ─────────────────
ls -l /opt/unetlab/addons/qemu/ | grep vt
ls -l /opt/unetlab/addons/qemu/vt*/
# Should show hda.qcow2 inside each folder ✅
```

> ⚠️ **Common mistake:** Naming the folder `viptela-*` instead of `vt-*`. Always confirm against the `.yml` template file.

</details>

---

<details>
<summary>🖥️ Step 2 — EVE-NG Node Settings (RAM / CPU / NICs)</summary>

```
When adding nodes → Right-click canvas → Add Node → select template

┌─────────────────────────────────────────────────────────────────┐
│  vManage  (Template: vtmgmt)                                     │
│    Image: vtmgmt-20.3.1                                          │
│    RAM:   8192 MB  ← MINIMUM. vManage crashes with less          │
│    CPU:   2                                                      │
│    NICs:  3  (eth0=VPN512, eth1=VPN0, eth2=spare)               │
└─────────────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────────────┐
│  vSmart  (Template: vtsmart)                                     │
│    Image: vtsmart-20.3.1                                         │
│    RAM:   2048 MB                                                │
│    CPU:   2                                                      │
│    NICs:  2  (eth0=VPN512, eth1=VPN0)                           │
└─────────────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────────────┐
│  vBond  (Template: vtbond)                                       │
│    Image: vtbond-20.3.1                                          │
│    RAM:   2048 MB                                                │
│    CPU:   2                                                      │
│    NICs:  3  (eth0=VPN512, ge0/0=VPN0, ge0/1=VPN0)             │
└─────────────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────────────┐
│  vEdge Nodes × 4  (Template: vtedge)                             │
│    Image: vtedge-20.3.1                                          │
│    RAM:   1024 MB                                                │
│    CPU:   1                                                      │
│    NICs:  5  (eth0=VPN512, ge0/0–ge0/3 = VPN0/Service)         │
└─────────────────────────────────────────────────────────────────┘

⚠️  CRITICAL — VPN Interface Mapping:
    eth0 is HARDCODED to VPN 512 on ALL vBond and vEdge images.
    Control connections ONLY work over VPN 0.
    Wire the CORRECT VPN 0 interface to the transport switch:

      vManage  → eth1  → switch
      vSmart   → eth1  → switch
      vBond    → ge0/1 → switch
      vEdge    → ge0/1 → switch

    Wiring eth0 to transport = control connections will NEVER form.
```

</details>

---

<details>
<summary>⚠️ Step 3 — Fix vManage Secondary Storage (REQUIRED Before First Boot)</summary>

```bash
# ════════════════════════════════════════════════════════════════════
# vManage requires a SECONDARY DISK (hdc) for its NMS database.
# EVE-NG does NOT create this automatically — you MUST do it.
# Skipping this = vManage errors out OR loses all data on reboot.
# ════════════════════════════════════════════════════════════════════

# STEP 1: Create the lab in EVE-NG GUI and add ALL nodes
#         ⚠️  DO NOT start any nodes yet

# STEP 2: Find your lab UUID on the EVE-NG host
ls /opt/unetlab/tmp/0/
# Example: f68fea5b-312c-4bab-8aff-32b32d31b704

# STEP 3: List all node folders inside the lab UUID
ls /opt/unetlab/tmp/0/<your-lab-uuid>/
# Output: 1  2  3  4  5  6 ...  (one folder per node)

# STEP 4: Find which folder number belongs to vManage
grep -n "vtmgmt" /opt/unetlab/labs/<your-lab-name>.unl
# Example output:
#   <node id="12" ... template="vtmgmt" ...>
# → vManage is node ID 12

# STEP 5: Create the 30 GB secondary disk in the vManage node folder
cd /opt/unetlab/tmp/0/<your-lab-uuid>/12/
qemu-img create -f qcow2 hdc.qcow2 30G
# Output: Formatting 'hdc.qcow2', fmt=qcow2 size=32212254720 ...

ls -lh hdc.qcow2
# ~196 KB sparse file — grows as vManage uses it ✅

# STEP 6: Inject -hdc into the lab XML file
# ⚠️  CRITICAL: Target ONLY the vtmgmt node!
# Adding -hdc to other nodes will cause them to CRASH on boot.
sed -i '/<node.*template="vtmgmt"/s|qemu_options="\([^"]*\)"|qemu_options="\1 -hdc hdc.qcow2"|' \
  "/opt/unetlab/labs/<your-lab-name>.unl"

# STEP 7: Verify ONLY the vtmgmt node has -hdc
grep "hdc" "/opt/unetlab/labs/<your-lab-name>.unl"
# ✅ Good: exactly ONE line — on the vtmgmt node
# ❌ Bad:  multiple lines → manually edit .unl to remove from non-vManage nodes

# STEP 8: Now start vManage from the EVE-NG GUI

# STEP 9: Confirm QEMU picked up the secondary disk
ps aux | grep vManage | grep hdc
# Should show: ... -hdc hdc.qcow2 ... in the process command ✅
```

</details>

---

<details>
<summary>🗺️ Step 4 — Build Lab Topology & Wire Interfaces</summary>

```
TOPOLOGY DIAGRAM:
                         ┌─────────┐
                         │   Net   │  (Cloud0/pnet0 = EVE-NG host bridge)
                         └────┬────┘
                              │ Gi1/0
                       ┌──────┴──────┐
         Gi0/0 ─────── │ Vt_Mgmt_SW  │ ─────── Gi0/3
                       │  (vIOS-L2)  │
                       └┬─────┬─────┬┘
                        │Gi0/1│Gi0/2│
                   eth1─┘     │     └─ge0/1
                              │ge0/1
              ┌───────────────┤    ┌──────────────┐
              │               │    │
       ┌──────┴──────┐  ┌─────┴────┐  ┌──────────┐  ┌──────────────┐
       │  vManage    │  │  vSmart  │  │  vBond   │  │ vEdge-Site10 │
       │192.168.1.10 │  │192.168.1.12│ │192.168.1.11│ │192.168.1.21  │
       └─────────────┘  └──────────┘  └──────────┘  └──────────────┘

INTERFACE WIRING TABLE:
  Device            Interface   →  Connect To
  ────────────────────────────────────────────────
  vManage           eth1        →  Vt_Mgmt_SW Gi0/0
  vSmart            eth1        →  Vt_Mgmt_SW Gi0/1
  vBond             ge0/1       →  Vt_Mgmt_SW Gi0/2
  vEdge-Site10      ge0/1       →  Vt_Mgmt_SW Gi0/3
  Vt_Mgmt_SW        Gi1/0       →  Cloud0  (gateway reachability)

⚠️  DO NOT wire eth0 to the switch
    eth0 = VPN 512 (management OOB) — not transport
    ge0/0 on vBond is available for a second transport (optional)

HOW TO WIRE IN EVE-NG GUI:
  1. Hover over source node → drag the interface connector
  2. Drop on destination node → select matching interface
  3. Verify wiring: click any link to see both interface labels
```

</details>

---

## 2.3 IP Address Plan

### VPN 0 Transport — 192.168.1.0/24

| Device | Role | VPN 0 IP | System-IP | Site-ID | Credentials |
|--------|------|----------|-----------|---------|-------------|
| vManage | Management Plane | 192.168.1.10/24 | 10.10.10.1 | 1000 | admin / Admin@123 |
| vBond | Orchestration Plane | 192.168.1.11/24 | 10.10.10.3 | 1000 | admin / Admin@123 |
| vSmart | Control Plane | 192.168.1.12/24 | 10.10.10.2 | 1000 | admin / Admin@123 |
| vEdge-Site10 | WAN Edge | 192.168.1.21/24 | 10.10.10.11 | 10 | admin / Admin@123 |
| Gateway | Home Router | 192.168.1.1/24 | — | — | — |

> **Organization Name:** `NetworkCoder` — identical on ALL devices, case-sensitive.

---

<details>
<summary>💻 Step 5 — vManage First Boot & Configuration</summary>

```
# ════════════════════════════════════════════════════════════════════
# vManage First Boot
# ⚠️  "Login incorrect" on first attempts is NORMAL.
#     The NMS authentication service takes 2–3 min to start.
#     Keep trying every 30 seconds.
# ════════════════════════════════════════════════════════════════════

vmanage login: admin
Password: admin

# ── Storage Device Selection ──────────────────────────────────────
# vManage detects the 30 GB hdc.qcow2 created in Step 3

Available storage devices:
hdc     30GB

1) hdc
Select storage device to use: 1
Would you like to format hdc? (y/n): y
# Wait ~30 seconds for format to complete

# ── Password Change ───────────────────────────────────────────────
New password:     Admin@123
Confirm password: Admin@123

# ── System + VPN 0 Configuration ─────────────────────────────────
config

system
 host-name             vManage
 system-ip             10.10.10.1
 site-id               1000
 organization-name     "NetworkCoder"
 vbond 192.168.1.11
!

vpn 0
 interface eth1
  ip address 192.168.1.10/24
  tunnel-interface
   allow-service all
  !
  no shutdown
 !
 ip route 0.0.0.0/0 192.168.1.1
!

commit
end

# ── Verification ──────────────────────────────────────────────────
show interface description
# eth1 → up/up → 192.168.1.10 in VPN 0 ✅

ping 192.168.1.1
# Gateway ping OK ✅

show system status
# Confirms: system-ip 10.10.10.1, site-id 1000, org NetworkCoder

# ── Access GUI ────────────────────────────────────────────────────
# Browser → https://192.168.1.10:8443
# Login:    admin / Admin@123
# Accept certificate warning

# GUI: Administration → Settings
#   Organization Name:                  NetworkCoder
#   vBond Address:                      192.168.1.11
#   Controller Certificate Authorization: Enterprise Root Certificate
```

</details>

---

<details>
<summary>💻 Step 6 — vSmart First Boot & Configuration</summary>

```
# No storage step needed for vSmart

vsmart login: admin
Password: admin

New password:     Admin@123
Confirm password: Admin@123

config

system
 host-name             vSmart
 system-ip             10.10.10.2
 site-id               1000
 organization-name     "NetworkCoder"
 vbond 192.168.1.11
!

vpn 0
 interface eth1
  ip address 192.168.1.12/24
  tunnel-interface
   allow-service all
  !
  no shutdown
 !
 ip route 0.0.0.0/0 192.168.1.1
!

commit
end

# ── Verification ──────────────────────────────────────────────────
show interface description
# eth1 → up/up → 192.168.1.12 ✅

ping 192.168.1.10    # vManage ✅
ping 192.168.1.1     # Gateway ✅

show system status

# ⚠️  Control connections still empty — expected at this stage
show control connections
# (empty)
```

</details>

---

<details>
<summary>💻 Step 7 — vBond First Boot & Configuration</summary>

```
# ⚠️  vBond uses ge0/x for VPN 0, NOT eth1
# ⚠️  "local vbond-only" keyword is REQUIRED in the vbond config line

vbond login: admin
Password: admin

New password:     Admin@123
Confirm password: Admin@123

config

system
 host-name             vBond
 system-ip             10.10.10.3
 site-id               1000
 organization-name     "NetworkCoder"
 vbond 192.168.1.11 local vbond-only
!

vpn 0
 interface ge0/1
  ip address 192.168.1.11/24
  tunnel-interface
   encapsulation ipsec
   allow-service all
  !
  no shutdown
 !
 ip route 0.0.0.0/0 192.168.1.1
!

commit
end

# ── Verification ──────────────────────────────────────────────────
show interface description
# ge0/1 → up/up → 192.168.1.11 ✅

ping 192.168.1.10    # vManage ✅
ping 192.168.1.12    # vSmart  ✅
ping 192.168.1.1     # Gateway ✅
```

</details>

---

<details>
<summary>💻 Step 8 — vEdge Bootstrap Configuration</summary>

```
# ⚠️  vEdge uses ge0/x for VPN 0, NOT eth1
# Full multi-site configs in Module 04 — this is minimal bootstrap only

vedge login: admin
Password: admin

New password:     Admin@123
Confirm password: Admin@123

config

system
 host-name             vEdge-Site10
 system-ip             10.10.10.11
 site-id               10
 organization-name     "NetworkCoder"
 vbond 192.168.1.11
!

vpn 0
 interface ge0/1
  ip address 192.168.1.21/24
  tunnel-interface
   encapsulation ipsec
   allow-service all
  !
  no shutdown
 !
 ip route 0.0.0.0/0 192.168.1.1
!

commit
end

# ── Verification ──────────────────────────────────────────────────
show interface description
# ge0/1 → up/up → 192.168.1.21 ✅

ping 192.168.1.10    # vManage ✅
ping 192.168.1.11    # vBond   ✅
ping 192.168.1.12    # vSmart  ✅
ping 192.168.1.1     # Gateway ✅

# ⚠️  Control connections empty — expected, certs needed next
show control connections
# (empty — this is correct)
```

</details>

---

<details>
<summary>✅ Step 9 — End-of-Module Reachability Validation</summary>

```bash
# Run all pings from vManage to confirm full reachability
# before proceeding to Module 03 (Certificates)

ping 192.168.1.11    # vBond   → ✅
ping 192.168.1.12    # vSmart  → ✅
ping 192.168.1.21    # vEdge   → ✅
ping 192.168.1.1     # Gateway → ✅

# Verify system config on each device
show running-config system
# Confirm: host-name, system-ip, site-id, organization-name, vbond

# Verify interface status
show interface description
# All VPN 0 interfaces should show up/up

# ── These SHOULD be empty at this stage (correct): ───────────────
show control connections
# (empty) ← correct, certs not yet installed

show omp peers
# (empty) ← correct, OMP needs control connections first

# ── Module 02 is COMPLETE when all pings succeed ──────────────────
# Proceed to Module 03 for certificate installation
```

</details>

---

## 2.4 Common Issues & Fixes

| Problem | Root Cause | Fix |
|---------|-----------|-----|
| Template missing in EVE-NG node picker | Wrong folder prefix | Must use `vt` prefix — check `.yml` template file |
| vManage crashes at boot | Insufficient RAM | Set minimum 8192 MB for vManage |
| vManage loses config on restart | Missing hdc disk | Create `hdc.qcow2` per Step 3 before first boot |
| "Login incorrect" on console | Auth service still starting | Wait 2–3 min, keep trying |
| `hda.qcow2 not found` | Wrong filename | Rename image file exactly to `hda.qcow2` |
| Ping fails between nodes | Wrong interface wired | Confirm VPN 0 interface (eth1/ge0/1) is connected |
| Control connections empty after config | Expected | Certs not installed yet — proceed to Module 03 |
| Permission error on images | Forgot fixpermissions | `/opt/unetlab/wrappers/unl_wrapper -a fixpermissions` |

---

## ✅ Module 02 Completion Checklist

- [ ] EVE-NG installed and GUI accessible
- [ ] All 4 image folders created with `vt` prefix, each with `hda.qcow2`
- [ ] `fixpermissions` run — all 4 templates visible in node picker
- [ ] `hdc.qcow2` (30 GB) created in vManage node folder
- [ ] `-hdc` injected into ONLY the vManage node in `.unl` file
- [ ] Topology built with correct switch wiring (VPN 0 interfaces only)
- [ ] vManage formatted storage disk on first boot
- [ ] All 4 devices configured: system-ip, site-id, org-name = `NetworkCoder`
- [ ] All VPN 0 interfaces up/up with correct IPs
- [ ] All pings succeed between all devices and gateway
- [ ] `show control connections` = empty (expected — certs needed next)

---

[← Architecture](../01-architecture/README.md) | [Back to Main](../../README.md) | [Next: Control Plane →](../03-control-plane/README.md)
