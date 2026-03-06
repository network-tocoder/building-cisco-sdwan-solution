# 🏗️ Module 01 — Architecture Overview

[← Back to Main](../../README.md) | [Next: EVE-NG Setup →](../02-eve-ng-setup/README.md)

---

## 🎯 Learning Objectives

- Explain the Cisco SD-WAN architecture and fabric model
- Identify the role of each SD-WAN component
- Describe control vs data plane separation
- Understand OMP as the SD-WAN routing protocol
- Articulate common SD-WAN use cases

---

## 1.1 What Is Cisco SD-WAN?

Cisco SD-WAN (formerly Viptela, now **Cisco Catalyst WAN**) is a cloud-delivered overlay WAN that separates the **control plane** from the **data plane**, providing:

- Centralized management via **vManage**
- Policy-driven, automated routing
- Transport-agnostic connectivity (MPLS, Internet, LTE, Satellite)
- Application-aware path selection
- Native security integration

---

## 1.2 SD-WAN vs Traditional WAN

| Feature | Traditional WAN | Cisco SD-WAN |
|---------|----------------|--------------|
| Management | Per-device CLI | Centralized vManage |
| Routing | Static/OSPF/BGP per device | OMP centrally controlled |
| Transport | Single (MPLS) | Multi-transport |
| Provisioning | Weeks | Hours (ZTP) |
| Visibility | Per-device SNMP | Real-time analytics |
| Security | Bolt-on | Integrated |

---

## 1.3 Key Components

```
┌──────────────────────── MANAGEMENT PLANE ────────────────────────┐
│                    vManage (NMS / GUI / API)                      │
│       NETCONF · REST API · Template Engine · Policy Builder       │
└──────────────────────────────────────────────────────────────────┘
┌───────────────────────── CONTROL PLANE ──────────────────────────┐
│   vSmart (OMP Controller)          vBond (Orchestrator)           │
│   · Route & Policy distribution    · Auth & NAT traversal         │
│   · DTLS/TLS to WAN edges          · First point of contact       │
└──────────────────────────────────────────────────────────────────┘
┌────────────────────────── DATA PLANE ────────────────────────────┐
│   vEdge Cloud   ·   vEdge Hardware   ·   cEdge (IOS-XE)          │
│   (VM)              (Physical)            CSR1000v / ISR / ASR    │
└──────────────────────────────────────────────────────────────────┘
```

---

## 1.4 OMP — The SD-WAN Routing Protocol

| Route Type | Description |
|------------|-------------|
| **OMP Routes** | Service-side prefixes (LAN routes) |
| **TLOC Routes** | Transport endpoints (tunnel addresses) |
| **Service Routes** | Service chain info (FW, IDS) |

---

## 1.5 Key Concepts

- **TLOC:** `System-IP + Color + Encapsulation` — uniquely identifies a tunnel endpoint
- **OMP:** Runs between WAN Edges and vSmart — carries routes, TLOCs, services
- **IPSec tunnels:** Built directly between WAN Edges — vSmart is NOT in the data path
- **BFD:** Monitors each tunnel for liveliness and path quality metrics

---

## 1.6 Common Use Cases

| Use Case | Description |
|----------|-------------|
| Branch WAN Modernization | Replace MPLS with cheaper internet + SD-WAN |
| Multi-Cloud Connectivity | Direct breakout to AWS/Azure/GCP |
| SaaS Optimization | Best path to O365, Salesforce, WebEx |
| Network Segmentation | Multi-VRF across the WAN |
| Secure DIA | Branch internet breakout with integrated FW/Umbrella |

---

[← Back to Main](../../README.md) | [Next: EVE-NG Setup →](../02-eve-ng-setup/README.md)
