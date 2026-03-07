# 🔐 Module 03 — Certificates & Control Plane Bring-Up

[← EVE-NG Setup](../02-eve-ng-setup/README.md) | [Back to Main](../../README.md) | [Next: Edge Onboarding →](../04-edge-onboarding/README.md)

---

## 🎯 Learning Objectives

- Understand the SD-WAN PKI certificate trust model
- Generate an enterprise Root CA using OpenSSL
- Install the Root CA on vManage
- Generate CSRs from each controller
- Sign CSRs with the Root CA
- Install signed certificates on all controllers
- Validate that DTLS/TLS control connections come up
- Upload the WAN Edge authorized serial file
- Verify full OMP peering with all expected states

---

## 📺 Video Timestamps

| Topic | Time |
|-------|------|
| Certificate theory overview | 0:00 |
| Generate Root CA with OpenSSL | 2:30 |
| Install Root CA on vManage GUI | 5:00 |
| Generate CSRs — all controllers | 7:00 |
| Sign CSRs with Root CA | 10:00 |
| Install signed certificates | 12:00 |
| Watch control connections come up | 15:00 |
| Upload WAN Edge serial file | 18:00 |
| Full verification walkthrough | 20:00 |

---

## 3.1 Certificate Theory

### Why Certificates?

Before any SD-WAN component can communicate with another, it must **prove its identity**. This is done through mutual DTLS/TLS authentication using certificates. Without valid certificates:

- vBond will refuse connections from controllers and WAN Edges
- vSmart will not establish OMP sessions
- WAN Edges will never form control connections

### Trust Chain

```
┌─────────────────────────────────────────────────────────────────┐
│              SD-WAN CERTIFICATE TRUST CHAIN                      │
│                                                                  │
│  Root CA  (Enterprise OpenSSL  OR  Cisco/Symantec PKI)           │
│     │                                                            │
│     ├── Signs → vManage.pem                                     │
│     ├── Signs → vSmart.pem                                      │
│     ├── Signs → vBond.pem                                       │
│     └── Signs → vEdge.pem  (from authorized serial file)        │
│                                                                  │
│  ✅ All signed by same Root CA = they trust each other           │
│  ❌ Any mismatch in org-name or CA = "CHALLENGE" error           │
└─────────────────────────────────────────────────────────────────┘
```

### Two CA Options

| Option | When to Use | How |
|--------|------------|-----|
| **Enterprise Root CA** | Lab, private/isolated deployments | Generate with OpenSSL (this module) |
| **Cisco PKI / Symantec** | Production deployments | Automatic — no OpenSSL needed |

### Organization Name Binding

The **Organization Name** (`O=NetworkCoder`) in every certificate must exactly match the `organization-name "NetworkCoder"` in every device's system config.

```
Certificate Subject (must match):
  /C=US/ST=Lab/L=Lab/O=NetworkCoder/CN=vManage

Device System Config (must match):
  system
   organization-name "NetworkCoder"

Mismatch = connection refused with error: CHALLENGE or NOENTSVR
```

---

## 3.2 Certificate Workflow Overview

```
STEP 1:  Generate Root CA key + certificate (OpenSSL on Linux)
              ↓
STEP 2:  Install Root CA on vManage (GUI → Administration → Settings)
              ↓
STEP 3:  Generate CSR on each controller (vManage, vSmart, vBond)
              ↓
STEP 4:  Sign each CSR with Root CA (OpenSSL on Linux)
              ↓
STEP 5:  Install signed certificates on each controller
              ↓
STEP 6:  Watch control connections establish automatically
              ↓
STEP 7:  Upload WAN Edge authorized serial file (for WAN Edges to join)
              ↓
STEP 8:  Verify — all connections UP, OMP peers established
```

---

<details>
<summary>📋 Step 1 — Generate Enterprise Root CA (OpenSSL)</summary>

```bash
# ════════════════════════════════════════════════════════════════════
# Run these commands on your Linux machine
# (EVE-NG host, Ansible node, or any Ubuntu/Kali machine)
# ════════════════════════════════════════════════════════════════════

mkdir -p ~/sdwan-certs && cd ~/sdwan-certs

# ── Generate Root CA Private Key ──────────────────────────────────
openssl genrsa -out CA.key 2048

# Verify the key was created
ls -lh CA.key
# Output: -rw------- 1 root root 1.7K ...  CA.key

# ── Generate Root CA Certificate (self-signed, valid 10 years) ────
openssl req -new -x509 -days 3650 -key CA.key -out CA.pem \
  -subj "/C=US/ST=Lab/L=Lab/O=NetworkCoder/CN=NetworkCoder-Root-CA"

# ⚠️  O=NetworkCoder  MUST match the organization-name in device configs
# ⚠️  Change "NetworkCoder" to your actual org name if different

# ── Verify the Root CA certificate ───────────────────────────────
openssl x509 -in CA.pem -text -noout | head -30
# Check: Issuer, Subject, Validity dates

# ── Summary of files created ─────────────────────────────────────
ls -lh ~/sdwan-certs/
# CA.key    ← Root CA private key  (KEEP THIS SECURE)
# CA.pem    ← Root CA certificate  (install on vManage)
```

</details>

---

<details>
<summary>🖥️ Step 2 — Install Root CA on vManage</summary>

```bash
# ════════════════════════════════════════════════════════════════════
# Option A: via vManage GUI  (recommended)
# ════════════════════════════════════════════════════════════════════

# 1. Browser → https://192.168.1.10:8443
# 2. Login: admin / Admin@123
# 3. Navigate: Administration → Settings
# 4. Find: "Controller Certificate Authorization"
# 5. Click: "Edit"
# 6. Change type to: "Enterprise Root Certificate"
# 7. In the certificate field: paste the FULL contents of CA.pem
#    (include -----BEGIN CERTIFICATE----- and -----END CERTIFICATE-----)
# 8. Click: "Import"
# 9. A green success banner should appear

# ════════════════════════════════════════════════════════════════════
# Option B: via vManage CLI
# ════════════════════════════════════════════════════════════════════

# Copy CA.pem to vManage:
scp ~/sdwan-certs/CA.pem admin@192.168.1.10:/home/admin/CA.pem

# SSH into vManage and install:
ssh admin@192.168.1.10
request root-cert-chain install /home/admin/CA.pem

# ── Verify Root CA is installed ──────────────────────────────────
show certificate root-ca-cert
# Should display the CA.pem certificate contents
# Look for: Subject: O=NetworkCoder, CN=NetworkCoder-Root-CA ✅
```

</details>

---

<details>
<summary>📝 Step 3 — Generate CSRs from All Controllers</summary>

```bash
# ════════════════════════════════════════════════════════════════════
# Option A: vManage GUI  (recommended — generates CSRs for all)
# ════════════════════════════════════════════════════════════════════

# 1. Navigate: Configuration → Certificates → Controllers
# 2. You will see vManage, vSmart, and vBond listed
# 3. For each controller:
#    → Click the three-dot menu (⋮)
#    → Click "Generate CSR"
#    → Copy the CSR text shown OR download it
# 4. Save each CSR to your Linux machine:
#    ~/sdwan-certs/vManage.csr
#    ~/sdwan-certs/vSmart.csr
#    ~/sdwan-certs/vBond.csr

# ════════════════════════════════════════════════════════════════════
# Option B: via CLI on each controller
# ════════════════════════════════════════════════════════════════════

# On vManage:
ssh admin@192.168.1.10
request csr upload /home/admin/vManage.csr
# Copy output CSR text to ~/sdwan-certs/vManage.csr on Linux machine

# On vSmart:
ssh admin@192.168.1.12
request csr upload /home/admin/vSmart.csr

# On vBond:
ssh admin@192.168.1.11
request csr upload /home/admin/vBond.csr

# ── Transfer CSR files to Linux signing machine ───────────────────
scp admin@192.168.1.10:/home/admin/vManage.csr ~/sdwan-certs/
scp admin@192.168.1.12:/home/admin/vSmart.csr  ~/sdwan-certs/
scp admin@192.168.1.11:/home/admin/vBond.csr   ~/sdwan-certs/

# ── Verify CSRs were received ─────────────────────────────────────
ls -lh ~/sdwan-certs/
# CA.key  CA.pem  vManage.csr  vSmart.csr  vBond.csr

# Quick sanity check on a CSR:
openssl req -in ~/sdwan-certs/vManage.csr -text -noout | grep -E "Subject:|CN="
# Should show: CN=vManage or similar
```

</details>

---

<details>
<summary>✍️ Step 4 — Sign CSRs with Root CA (OpenSSL)</summary>

```bash
cd ~/sdwan-certs

# ── Sign vManage CSR ──────────────────────────────────────────────
openssl x509 -req \
  -in vManage.csr \
  -CA CA.pem \
  -CAkey CA.key \
  -CAcreateserial \
  -out vManage.pem \
  -days 3650 \
  -sha256

# Expected output:
# Signature ok
# subject=/CN=vManage...
# Getting CA Private Key

# ── Sign vSmart CSR ───────────────────────────────────────────────
openssl x509 -req \
  -in vSmart.csr \
  -CA CA.pem \
  -CAkey CA.key \
  -CAcreateserial \
  -out vSmart.pem \
  -days 3650 \
  -sha256

# ── Sign vBond CSR ────────────────────────────────────────────────
openssl x509 -req \
  -in vBond.csr \
  -CA CA.pem \
  -CAkey CA.key \
  -CAcreateserial \
  -out vBond.pem \
  -days 3650 \
  -sha256

# ── Verify all signed certificates were created ───────────────────
ls -lh ~/sdwan-certs/
# CA.key  CA.pem  CA.srl
# vManage.csr  vManage.pem
# vSmart.csr   vSmart.pem
# vBond.csr    vBond.pem

# ── Verify certificate content ────────────────────────────────────
openssl x509 -in vManage.pem -text -noout | grep -E "Issuer:|Subject:|Not After"
# Issuer should show:  O=NetworkCoder, CN=NetworkCoder-Root-CA
# Subject should show: CN=vManage (or similar)
# Not After: should be 10 years out ✅
```

</details>

---

<details>
<summary>📥 Step 5 — Install Signed Certificates on Controllers</summary>

```bash
# ════════════════════════════════════════════════════════════════════
# Option A: via vManage GUI  (installs on ALL controllers at once)
# ════════════════════════════════════════════════════════════════════

# 1. Navigate: Configuration → Certificates → Controllers
# 2. For each controller (vManage, vSmart, vBond):
#    → Click the three-dot menu (⋮)
#    → Click "Install Certificate"
#    → Paste the contents of the signed .pem file
#      (include BEGIN/END CERTIFICATE lines)
#    → Click "Install"
# 3. Repeat for vSmart.pem and vBond.pem

# ⚠️  After installing certs, controllers will AUTOMATICALLY attempt
#    to establish DTLS/TLS control connections.
#    Watch the control connections come up in real time!

# ════════════════════════════════════════════════════════════════════
# Option B: via CLI on each controller
# ════════════════════════════════════════════════════════════════════

# Transfer signed certs to each controller:
scp ~/sdwan-certs/vManage.pem admin@192.168.1.10:/home/admin/
scp ~/sdwan-certs/vSmart.pem  admin@192.168.1.12:/home/admin/
scp ~/sdwan-certs/vBond.pem   admin@192.168.1.11:/home/admin/

# Install on vManage:
ssh admin@192.168.1.10
request certificate install /home/admin/vManage.pem

# Install on vSmart:
ssh admin@192.168.1.12
request certificate install /home/admin/vSmart.pem

# Install on vBond:
ssh admin@192.168.1.11
request certificate install /home/admin/vBond.pem
```

</details>

---

<details>
<summary>✅ Step 6 — Validate Control Connections</summary>

```bash
# ════════════════════════════════════════════════════════════════════
# Run these on each controller after certificate installation
# Wait up to 2–3 minutes for DTLS sessions to fully establish
# ════════════════════════════════════════════════════════════════════

# ── From vManage ──────────────────────────────────────────────────
show control connections

# EXPECTED OUTPUT (all states = "up"):
# PEER    PROT  SYSTEM-IP    SITE   DOMAIN  STATE   UPTIME
# vbond   dtls  10.10.10.3   1000   0       up      0:00:04:22  ✅
# vsmart  dtls  10.10.10.2   1000   1       up      0:00:04:18  ✅

show orchestrator connections
# Should show vBond connection as "up"

show control connections-history
# Shows a log of all connection attempts — look for "state=up" transitions
# Useful for troubleshooting if connections fail

# ── From vBond ────────────────────────────────────────────────────
show orchestrator connections

# EXPECTED OUTPUT:
# SYSTEM-IP    SITE-ID   STATE   UPTIME
# 10.10.10.1   1000      up      0:00:05:10  ← vManage ✅
# 10.10.10.2   1000      up      0:00:05:06  ← vSmart  ✅
# 10.10.10.11  10        up      0:00:02:15  ← vEdge (after serial upload) ✅

show control connections

# ── From vSmart ───────────────────────────────────────────────────
show control connections

# EXPECTED OUTPUT:
# PEER      PROT  SYSTEM-IP    SITE   STATE   UPTIME
# vmanage   dtls  10.10.10.1   1000   up      0:00:04:30  ✅
# vbond     dtls  10.10.10.3   1000   up      0:00:04:28  ✅

show omp peers
# PEER         TYPE     DOMAIN  STATE   UPTIME
# 10.10.10.11  vedge    1       up      0:00:02:10  ← vEdge-Site10 ✅

show omp summary
# Shows total routes, TLOCs, services being exchanged
```

</details>

---

<details>
<summary>📋 Step 7 — Upload WAN Edge Authorized Serial File</summary>

```bash
# ════════════════════════════════════════════════════════════════════
# Before a WAN Edge can join the fabric, its serial number must be
# uploaded to vManage. This is the "authorized device list".
# ════════════════════════════════════════════════════════════════════

# ── Get vEdge chassis and serial numbers ─────────────────────────
# SSH or console into each vEdge:
ssh admin@192.168.1.21

show certificate serial
# OUTPUT:
# Chassis Number  :  VEDGE-SITE10-AAAA-1234
# Certificate serial: ABCDEF1234567890

# ── Create the WAN Edge serial CSV file ──────────────────────────
# On your Linux machine, create the CSV:
cat > ~/sdwan-certs/wan-edge-serial.csv << 'EOF'
<chassis-number>,<serial-number>,NetworkCoder,valid
EOF

# Example with real values:
# VEDGE-SITE10-AAAA-1234,ABCDEF1234567890,NetworkCoder,valid

# For multiple edges, one line per edge:
# VEDGE-SITE10-AAAA,ABC123,NetworkCoder,valid
# VEDGE-SITE20-BBBB,DEF456,NetworkCoder,valid
# VEDGE-SITE30-CCCC,GHI789,NetworkCoder,valid

# ── Upload to vManage GUI ─────────────────────────────────────────
# 1. Navigate: Configuration → Devices → WAN Edge List
# 2. Click "Upload WAN Edge List"
# 3. Browse to wan-edge-serial.csv
# 4. Check: "Validate the uploaded WAN Edge List"
# 5. Click "Upload"
# 6. Status should show: "Upload Successful" ✅

# ── Verify upload ─────────────────────────────────────────────────
# Configuration → Devices → WAN Edge List
# All uploaded chassis numbers should show status: "Valid"

# ── After upload: trigger vEdge to connect ────────────────────────
# The vEdge will automatically retry vBond connection periodically.
# To force immediate retry, on vEdge CLI:
request sdwan control local-properties update-vbond
# OR simply wait — it retries every 30 seconds
```

</details>

---

<details>
<summary>🔍 Step 8 — Full Verification Walkthrough</summary>

```bash
# ════════════════════════════════════════════════════════════════════
# Run this full verification sequence on each device.
# Use as your end-of-module checklist.
# ════════════════════════════════════════════════════════════════════

# ── On vManage ────────────────────────────────────────────────────

# 1. Verify certificate is installed correctly
show certificate installed
# Should show: Valid, Subject includes O=NetworkCoder

# 2. Verify Root CA is loaded
show certificate root-ca-cert
# Should show: Issuer: CN=NetworkCoder-Root-CA ✅

# 3. Verify all control connections
show control connections
# All controllers show state = up ✅

# 4. Check connection history (no CHALLENGE errors?)
show control connections-history
# Look for: "state=up" events, NOT "CHALLENGE" or "NOENTSVR"

# ── On vSmart ─────────────────────────────────────────────────────

# 1. Certificate
show certificate installed

# 2. Control connections (vManage + vBond should be up)
show control connections

# 3. OMP peers (vEdge should appear after serial upload)
show omp peers
# PEER         TYPE   DOMAIN  STATE   UPTIME
# 10.10.10.11  vedge  1       up      ✅

# 4. OMP summary
show omp summary
# Shows: num_vsmart_peers, num_vmanage_peers, routes-received

# ── On vBond ──────────────────────────────────────────────────────

# 1. Certificate
show certificate installed

# 2. All orchestrator connections (vManage, vSmart, vEdge)
show orchestrator connections
# All rows should show state = up ✅

# ── On vEdge-Site10 ───────────────────────────────────────────────

# 1. Certificate
show certificate installed
show certificate serial

# 2. Control connections (should see vBond, vSmart, vManage)
show control connections
# PEER      PROT  SYSTEM-IP    STATE
# vbond     dtls  10.10.10.3   up  ✅
# vsmart    dtls  10.10.10.2   up  ✅
# vmanage   dtls  10.10.10.1   up  ✅

# 3. Verify OMP session with vSmart
show omp peers
# 10.10.10.2  vsmart  1  up ✅

# 4. Verify system status
show system status
# Personality: vedge
# Model:       vedge-cloud
# State:       green  ✅

# ── From vManage GUI ──────────────────────────────────────────────
# Monitor → Overview
#   Controllers section: vManage, vSmart, vBond all green ✅
#   WAN Edge Devices: vEdge-Site10 green ✅
#
# Configuration → Certificates → Controllers
#   All show: "Certificate Install" = Installed ✅
#   All show: "Control Status" = up ✅
```

</details>

---

## 3.3 Troubleshooting Control Connections

### Error Reference Table

| Error in `connections-history` | Meaning | Fix |
|---------------------------------|---------|-----|
| `CHALLENGE` | org-name mismatch between device config and certificate | Verify `organization-name` matches exactly in all device system configs AND in the cert subject `O=` field |
| `NOENTSVR` | Device serial not in authorized list | Upload WAN Edge serial CSV via vManage |
| `DCONFAIL` | DTLS handshake failed — cert not trusted | Verify Root CA is installed on vManage; re-sign and re-install certs |
| `TIMEOUT` | vBond unreachable | Check VPN 0 IP, default route, and vBond IP in system config |
| `connect` (stuck, not "up") | Port blocked or NAT issue | Check UDP 12346 is open from vEdge to vBond/vSmart |

### Quick Troubleshooting Commands

<details>
<summary>Troubleshooting Commands Reference</summary>

```bash
# ── Certificate status ────────────────────────────────────────────
show certificate installed        # show installed cert details
show certificate root-ca-cert     # confirm Root CA is loaded
show certificate serial           # chassis + serial numbers
show certificate validity         # NOT BEFORE / NOT AFTER dates

# ── Control plane ─────────────────────────────────────────────────
show control connections          # current connection state
show control connections-history  # full history with error codes
show control local-properties     # local DTLS parameters

# ── OMP ───────────────────────────────────────────────────────────
show omp peers                    # OMP session state
show omp summary                  # route counts, session summary

# ── System ────────────────────────────────────────────────────────
show system status                # overall device health
show clock                        # ⚠️  Clock skew can cause cert failures!
request nms all status            # vManage NMS service health

# ── Connectivity test ────────────────────────────────────────────
ping vpn 0 192.168.1.11           # can we reach vBond in VPN 0?
traceroute vpn 0 192.168.1.12     # path to vSmart?
```

</details>

---

## ✅ Module 03 Completion Checklist

- [ ] Root CA key and certificate generated (`CA.key`, `CA.pem`)
- [ ] Organization field in CA cert matches device `organization-name`
- [ ] Root CA installed on vManage — `show certificate root-ca-cert` confirms it
- [ ] CSRs generated from vManage, vSmart, and vBond
- [ ] All 3 CSRs signed with Root CA — `vManage.pem`, `vSmart.pem`, `vBond.pem` created
- [ ] Signed certificates installed on all 3 controllers
- [ ] `show control connections` on vManage shows vBond + vSmart = **up**
- [ ] `show control connections` on vSmart shows vManage + vBond = **up**
- [ ] `show orchestrator connections` on vBond shows vManage + vSmart = **up**
- [ ] WAN Edge serial CSV uploaded to vManage → status = Valid
- [ ] `show control connections` on vEdge shows vBond + vSmart + vManage = **up**
- [ ] `show omp peers` on vSmart shows vEdge-Site10 = **up**
- [ ] vManage GUI → Monitor → Overview → all devices **green**

---

[← EVE-NG Setup](../02-eve-ng-setup/README.md) | [Back to Main](../../README.md) | [Next: Edge Onboarding →](../04-edge-onboarding/README.md)
