# 🏗️ Module 01 — Architecture Overview

[Back to Main](../../README.md) | [Next: EVE-NG Setup →](../02-eve-ng-setup/README.md)

---

## 🎯 Learning Objectives

- Explain the Cisco SD-WAN architecture and four-plane separation model
- Identify the role of each component — vManage, vBond, vSmart, vEdge, cEdge
- Understand OMP as the SD-WAN routing protocol and its three route types
- Articulate the VPN model — VPN 0, VPN 512, and Service VPNs
- Understand the certificate trust model and Organization Name binding
- Describe DTLS/TLS control connection establishment sequence
- Identify common SD-WAN deployment use cases

---

## 📺 Video Timestamps

| Topic | Time |
|-------|------|
| SD-WAN architecture overview | 0:00 |
| Four-plane separation explained | 2:30 |
| vManage deep-dive | 4:00 |
| vBond deep-dive | 5:30 |
| vSmart deep-dive | 7:00 |
| vEdge / cEdge deep-dive | 8:30 |
| OMP and route types | 10:00 |
| VPN model | 12:00 |
| Certificate trust model | 14:00 |
| Control connection sequence | 16:00 |
| Use cases | 18:00 |

---

## 1.1 What Is Cisco SD-WAN?

Cisco SD-WAN (formerly Viptela, now **Cisco Catalyst WAN**) is a cloud-delivered overlay WAN architecture that fundamentally separates the **control plane** from the **data plane**, enabling:

- Centralized management via a single pane of glass (**vManage**)
- Policy-driven, automated routing across any transport (MPLS, Internet, LTE, Satellite)
- Transport-agnostic overlay using IPSec tunnels
- Application-aware, SLA-based path selection per application
- End-to-end network segmentation — multiple VRFs over the same WAN fabric
- Native security integration (Umbrella DNS, IPS, URL filtering, DIA)

---

## 1.2 SD-WAN vs Traditional WAN

| Feature | Traditional WAN | Cisco SD-WAN |
|---------|----------------|--------------|
| Management | Per-device CLI | Centralized vManage GUI + API |
| Routing | Static / OSPF / BGP per device | OMP centrally controlled by vSmart |
| Transport | Typically single (MPLS) | Multi-transport (MPLS + Internet + LTE) |
| Provisioning | Weeks — manual per-device | Hours — ZTP + templates |
| Visibility | Per-device SNMP polling | Real-time analytics + DPI |
| Security | Bolt-on, separate appliances | Integrated into the WAN Edge |
| Scalability | Complex per-device config changes | Single policy → all sites |
| Cost | High (MPLS-heavy) | Optimized (internet augmentation) |

---

## 1.3 Four-Plane Architecture

SD-WAN separates the network into four distinct planes. This separation is the foundation of the entire architecture:

```
┌──────────────────────────────────────────────────────────────────────┐
│                       MANAGEMENT PLANE                                │
│   ┌──────────────────────────────────────────────────────────────┐   │
│   │                       vManage (NMS)                           │   │
│   │  Single pane of glass: monitoring, configuration,             │   │
│   │  software upgrades, troubleshooting, and REST API             │   │
│   └──────────────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────────┘
         ↕ NETCONF / REST API / DTLS
┌──────────────────────────────────────────────────────────────────────┐
│                     ORCHESTRATION PLANE                               │
│   ┌──────────────────────────────────────────────────────────────┐   │
│   │                        vBond                                  │   │
│   │  Authenticates ALL SD-WAN components joining the fabric       │   │
│   │  Distributes vManage + vSmart IPs to WAN Edges               │   │
│   │  Facilitates NAT traversal (STUN-like mechanism)              │   │
│   │  ⚠️  Must be publicly reachable (or NAT-reachable)            │   │
│   └──────────────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────────┘
         ↕ DTLS (UDP 12346) / TLS (TCP 23456)
┌──────────────────────────────────────────────────────────────────────┐
│                        CONTROL PLANE                                  │
│   ┌──────────────────────────────────────────────────────────────┐   │
│   │                        vSmart                                 │   │
│   │  OMP route reflector — distributes routes, TLOCs, services    │   │
│   │  Enforces centralized control + data policies                 │   │
│   │  NOT in the data path — only influences routing decisions     │   │
│   └──────────────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────────┘
         ↕ IPSec Data Plane Tunnels (direct edge-to-edge)
┌──────────────────────────────────────────────────────────────────────┐
│                         DATA PLANE                                    │
│   ┌───────────────┐  ┌────────────────┐  ┌────────────────────────┐ │
│   │  vEdge Cloud  │  │  vEdge HW      │  │  cEdge (IOS-XE)        │ │
│   │  (VM-based)   │  │  100/1000/     │  │  CSR1000v / ISR /      │ │
│   │               │  │  2000/5000     │  │  ASR / Catalyst 8000   │ │
│   └───────────────┘  └────────────────┘  └────────────────────────┘ │
│   IPSec tunnels · BFD path monitoring · Data policies · QoS/shaping  │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 1.4 Component Deep Dive

<details>
<summary>vManage — Management Plane</summary>

**Role:** Single pane of glass for ALL SD-WAN operations

**Key Functions:**
- Web GUI + REST API (Swagger available at `/apidocs`)
- **Device Templates** — assemble feature templates into complete device configurations
- **Feature Templates** — modular config blocks (System, VPN 0, VPN 1, BGP, OSPF, AAR…)
- Software image management and zero-touch upgrades
- Real-time monitoring, alarms, and telemetry dashboards
- Certificate management — CSR generation, cert signing, installation
- NETCONF/YANG for programmatic device configuration

**Protocols / Ports:**
- NETCONF: TCP 830
- REST API: TCP 8443
- DTLS to controllers: UDP 12346

**In the Lab:**
```
System-IP:  10.10.10.1
Site-ID:    1000
VPN 0 IP:   192.168.1.10/24  (eth1)
GUI:        https://192.168.1.10:8443
Creds:      admin / Admin@123
```

**Important:** `eth0` is hardcoded to VPN 512 on vManage. Always configure `eth1` as VPN 0.

</details>

<details>
<summary>vBond — Orchestration Plane</summary>

**Role:** The "front door" — the FIRST point of contact for every SD-WAN component

**Key Functions:**
- Authenticates every device joining the SD-WAN fabric (via certificates + org-name match)
- Distributes vManage and vSmart IP addresses to WAN Edges (so edges know who to call)
- NAT traversal helper — acts like a STUN server so edges behind NAT can reach each other
- Validates that the `organization-name` in the device config matches the certificate subject

**Critical Rule:** vBond **MUST** be reachable from the public internet (or from all WAN Edges via the underlay). Control connections ONLY flow over **VPN 0**.

**Protocols / Ports:** DTLS (UDP 12346)

**In the Lab:**
```
System-IP:  10.10.10.3
Site-ID:    1000
VPN 0 IP:   192.168.1.11/24  (ge0/1)
Config:     vbond 192.168.1.11 local vbond-only
```

**Note:** `ge0/1` is the VPN 0 interface on vBond. `eth0` is VPN 512 (hardcoded). Do not wire eth0 to the transport switch.

</details>

<details>
<summary>vSmart — Control Plane</summary>

**Role:** The "brain" — controls ALL routing decisions across the fabric

**Key Functions:**
- OMP route reflector — learns routes from all WAN Edges, applies policies, distributes to all peers
- Enforces **centralized control policies** — route filtering, hub-spoke topology, path preference
- Pushes **centralized data policies** down to WAN Edges — traffic steering, AAR, QoS
- All routing policy logic runs here — WAN Edges simply execute it in the data plane

**Critical:** vSmart is **NOT** in the data path. Actual user packets flow **directly between WAN Edges** via IPSec tunnels. vSmart only influences routing decisions.

**Protocols / Ports:** OMP over DTLS (UDP 12346) or TLS (TCP 23456)

**In the Lab:**
```
System-IP:  10.10.10.2
Site-ID:    1000
VPN 0 IP:   192.168.1.12/24  (eth1)
```

</details>

<details>
<summary>vEdge / cEdge — Data Plane (WAN Edges)</summary>

**vEdge (Viptela OS):**
- **vEdge Cloud** — VM-based (EVE-NG, VMware, KVM) — used in this lab
- **vEdge 100/1000/2000/5000** — Physical hardware appliances
- OS: Viptela OS (CLI syntax different from IOS)

**cEdge (IOS-XE SD-WAN Mode):**
- **CSR1000v** — VM-based (functionally equivalent to vEdge Cloud)
- **ISR 1000/4000** — Physical branch routers
- **ASR 1000** — Physical aggregation / DC routers
- **Catalyst 8000v / 8300 / 8500** — Latest generation
- OS: IOS-XE with SD-WAN mode enabled

**Key Functions (both types):**
- Build **IPSec tunnels** directly to all other WAN Edges using TLOC info from vSmart
- Run **BFD** on every tunnel — measures latency, jitter, packet loss continuously
- Enforce **data policies** pushed by vSmart (AAR, QoS, traffic steering)
- Originate **OMP routes** — LAN prefixes + TLOCs advertised to vSmart
- Run service-side routing protocols (OSPF, BGP) inside VPN 1+

**In the Lab:**
```
vEdge-Site10:  System-IP 10.10.10.11, Site-ID 10
               VPN 0 IP: 192.168.1.21/24 (ge0/1)
```

</details>

---

## 1.5 OMP — Overlay Management Protocol

OMP is the SD-WAN control plane protocol. It runs **between WAN Edges and vSmart only** — never directly between two WAN Edges.

```
How OMP flows:

1. WAN Edge boots → contacts vBond (auth + get controller list)
2. WAN Edge establishes OMP session with vSmart (over DTLS/TLS)
3. WAN Edge advertises to vSmart:
     ├── OMP Routes    = LAN prefixes from service VPNs (e.g. 172.16.2.0/24)
     ├── TLOC Routes   = tunnel endpoint identifiers (System-IP + Color + Encap)
     └── Service Routes = service chain info (FW, IDS, NAT locations)
4. vSmart applies control policy → distributes routes to all OMP peers
5. WAN Edges receive TLOC info → build IPSec tunnels peer-to-peer
6. BFD starts on each IPSec tunnel → measures path quality
```

### OMP Route Types

| Route Type | What It Carries | Example |
|------------|----------------|---------|
| **OMP Routes** (vRoutes) | Service-side LAN prefixes | `172.16.2.0/24` from Site 10 |
| **TLOC Routes** | Tunnel endpoint identifiers | `10.10.10.11 + mpls + ipsec` |
| **Service Routes** | Service chain information | Firewall at Site 200 |

### OMP Route Attributes

```
Each OMP route advertisement carries:
  Prefix         :  172.16.2.0/24
  TLOC           :  10.10.10.11 + mpls + ipsec    ← next-hop for data plane
  Originator     :  10.10.10.11                    ← which edge owns this prefix
  Preference     :  0 (default, higher = preferred)
  Tag            :  optional (for policy matching)
  Origin         :  Connected / Static / BGP / OSPF
  Site-ID        :  10
  VPN            :  1
```

---

## 1.6 TLOC — Transport Location

A **TLOC** uniquely identifies a WAN tunnel endpoint. Always a **3-tuple**:

```
TLOC = System-IP  +  Color  +  Encapsulation

Examples:
  10.10.10.11 + mpls          + ipsec    ← Site10 on MPLS transport
  10.10.10.11 + biz-internet  + ipsec    ← Site10 on Internet transport
  10.10.10.20 + mpls          + ipsec    ← HQ on MPLS transport
```

### TLOC Colors Reference

| Color | Typical Transport |
|-------|-----------------|
| `mpls` | MPLS private WAN |
| `biz-internet` | Business broadband |
| `public-internet` | Consumer internet |
| `lte` | 4G/5G cellular backup |
| `metro-ethernet` | Metro-E / carrier ethernet |
| `private1`–`private6` | Generic private WAN labels |

---

## 1.7 VPN Model

```
┌─────────────────────────────────────────────────────────────────┐
│                    SD-WAN VPN MODEL                              │
│                                                                  │
│  VPN 0   — Transport VPN                                        │
│            ├── WAN-facing interfaces + tunnel-interfaces        │
│            ├── All OMP / DTLS / IPSec control & data tunnels    │
│            └── ⚠️  Control connections ONLY work over VPN 0     │
│                                                                  │
│  VPN 512 — Management VPN (Out-of-Band)                         │
│            ├── eth0 is HARDCODED to VPN 512 on vBond/vEdge      │
│            ├── SSH, SNMP, NTP device management access          │
│            └── Never carries user application traffic           │
│                                                                  │
│  VPN 1–511 — Service VPNs (User / Application Traffic)          │
│            ├── VPN 1 = Corporate LAN                            │
│            ├── VPN 2 = Guest WiFi                               │
│            ├── VPN 3 = IoT / OT                                 │
│            ├── VPN 4 = Voice / UC                               │
│            └── Each VPN = isolated VRF — zero cross-leakage     │
└─────────────────────────────────────────────────────────────────┘
```

> ⚠️ **Critical:** `eth0` is **hardcoded to VPN 512** on vBond and vEdge QEMU images.
> Always wire your **VPN 0 interface** (`eth1` on vManage/vSmart, `ge0/x` on vBond/vEdge) to the transport network.
> Wiring `eth0` to the transport switch = control connections **will never form**.

---

## 1.8 Certificate Trust Model

The SD-WAN fabric uses a **PKI trust model** — every component must have a certificate signed by the same Root CA.

```
┌─────────────────────────────────────────────────────────────────┐
│              SD-WAN CERTIFICATE TRUST CHAIN                      │
│                                                                  │
│  Root CA (Enterprise OpenSSL or Cisco/Symantec PKI)              │
│     │                                                            │
│     ├── Signs → vManage Certificate                             │
│     ├── Signs → vSmart Certificate                              │
│     ├── Signs → vBond Certificate                               │
│     └── Signs → WAN Edge Certificate                            │
│                                                                  │
│  All devices share the same Root CA → they trust each other     │
│                                                                  │
│  Organization Name MUST match on every device + every cert      │
│  Mismatch → "CHALLENGE" error → control connections REFUSED     │
└─────────────────────────────────────────────────────────────────┘
```

**Key points:**
- **Enterprise Root CA** — Generate your own with OpenSSL (used in this lab / private deployments)
- **Cisco PKI / Symantec** — Used in production (auto-signed, no OpenSSL needed)
- **Organization Name** — e.g. `NetworkCoder` — globally unique string, **must be identical** in every device system config AND in every certificate subject field
- **Certificate flow:** Each controller generates a CSR → you sign it with Root CA → install signed cert on controller

---

## 1.9 Control Connection Bring-Up Sequence

Understanding this sequence is critical for troubleshooting:

```
Full bring-up after certificates are installed:

① WAN Edge boots with system config
     (system-ip, site-id, org-name, vbond IP)
         ↓
② WAN Edge → UDP 12346 → vBond  (DTLS, presents certificate)
         ↓
③ vBond validates:
     ✅ org-name in device config matches cert subject
     ✅ certificate is signed by trusted Root CA
         ↓
④ vBond → sends vManage IP + vSmart IP list to WAN Edge
         ↓
⑤ WAN Edge → TCP 23456 → vManage  (registers, pulls config/template)
         ↓
⑥ WAN Edge → UDP 12346 → vSmart   (OMP session establishes)
         ↓
⑦ vSmart ↔ WAN Edge exchange OMP routes + TLOCs
         ↓
⑧ WAN Edge builds IPSec tunnels to peers using TLOC info
         ↓
⑨ BFD starts on each IPSec tunnel
     (measures latency, jitter, packet loss every BFD interval)
         ↓
✅ Device fully operational in SD-WAN fabric
```

---

## 1.10 Underlay Protocols

| Protocol | Purpose | Transport | Default Port |
|----------|---------|-----------|-------------|
| **DTLS** | Control connections (vBond, vSmart, vManage) | UDP | 12346 |
| **TLS** | Alternative to DTLS (when UDP is blocked) | TCP | 23456 |
| **OMP** | Routing protocol (runs inside DTLS/TLS) | DTLS/TLS | 12346 |
| **BFD** | Tunnel path monitoring (latency/jitter/loss) | UDP | 12346 |
| **IPSec** | Data plane encryption between WAN Edges | UDP | 500 / 4500 |
| **NETCONF** | vManage → device configuration | TCP | 830 |

---

## 1.11 Common SD-WAN Use Cases

| Use Case | How SD-WAN Solves It |
|----------|---------------------|
| **Branch WAN Modernization** | Replace expensive MPLS with cheaper broadband + SD-WAN overlay |
| **Multi-Cloud Connectivity** | Cloud OnRamp — direct, optimized path to AWS / Azure / GCP |
| **SaaS Optimization** | Per-branch measurement of best exit path to O365/Salesforce/WebEx |
| **Network Segmentation** | Multi-VRF over the same WAN fabric — zero cross-VPN leakage by default |
| **Automatic Failover** | MPLS → Internet → LTE failover in seconds — BFD-driven detection |
| **Secure DIA at Branch** | Direct Internet Access with Umbrella DNS + integrated firewall |
| **MPLS Cost Reduction** | Run MPLS + Internet in parallel, shift non-critical traffic to internet |

---

## 📝 Key Takeaways

> These are the must-know fundamentals before any lab work:

- SD-WAN **separates management, orchestration, control, and data planes** into distinct components
- **vBond is always first contact** — it MUST be publicly or NAT reachable from all WAN Edges
- **vSmart is the routing brain** — OMP routes flow through it, but it is **NOT** in the data path
- **IPSec tunnels are built directly between WAN Edges** — no hairpin through vSmart
- **Organization Name** must be **identical** on every device — it is the fabric's trust anchor
- **eth0 = VPN 512** (hardcoded on vBond/vEdge) — NEVER wire eth0 to the transport switch
- **VPN 0** is required for ALL control connections — VPN 512 is out-of-band management only
- **Certificates must be signed by the same Root CA** — this is what allows mutual DTLS authentication

---

## 🔗 Resources

- [Cisco SD-WAN Solution Overview](https://www.cisco.com/c/en/us/solutions/enterprise-networks/sd-wan/index.html)
- [Cisco SD-WAN Documentation Hub](https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/system-interface/vedge-20-x/systems-interfaces-book.html)
- [Cisco SD-WAN Certificate Management](https://www.cisco.com/c/en/us/td/docs/routers/sdwan/configuration/system-interface/vedge-20-x/systems-interfaces-book/certificates.html)
- [Cisco DevNet SD-WAN Learning Labs](https://developer.cisco.com/sdwan/)
- [ENSDWI 300-415 Exam Topics](https://learningnetwork.cisco.com/s/ensdwi-exam-topics)

---

[Back to Main](../../README.md) | [Next: EVE-NG Setup →](../02-eve-ng-setup/README.md)
