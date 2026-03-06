# 🌐 Building Cisco SD-WAN Solution — Design, Build & Automate

<div align="center">

![Cisco SD-WAN](https://img.shields.io/badge/Cisco-SD--WAN-blue?style=for-the-badge&logo=cisco&logoColor=white)
![EVE-NG](https://img.shields.io/badge/EVE--NG-Lab-orange?style=for-the-badge)
![Ansible](https://img.shields.io/badge/Ansible-Automation-red?style=for-the-badge&logo=ansible&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.10+-green?style=for-the-badge&logo=python&logoColor=white)
![Terraform](https://img.shields.io/badge/Terraform-IaC-purple?style=for-the-badge&logo=terraform&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=for-the-badge)

**A complete end-to-end guide to designing, building, and automating Cisco SD-WAN (Catalyst WAN) in an EVE-NG lab environment.**

*Companion repository for the video series — "Building Cisco SD-WAN Solution: Design, Build & Automate"*

[▶️ Watch the Video Series](#) · [📖 Read the Docs](#-module-index) · [🐛 Report an Issue](../../issues) · [💬 Discussions](../../discussions)

</div>

---

## 📋 Table of Contents

- [About This Lab](#-about-this-lab)
- [Lab Topology](#-lab-topology)
- [Prerequisites](#-prerequisites)
- [Quick Start](#-quick-start)
- [Module Index](#-module-index)
- [Automation Modules](#-automation-modules)
- [Repo Structure](#-repository-structure)
- [SD-WAN Component Summary](#-cisco-sd-wan-component-summary)
- [Contributing](#-contributing)
- [License](#-license)

---

## 🎯 About This Lab

This repository is the **complete companion guide** for the video series *"Building Cisco SD-WAN Solution — Design, Build & Automate"*. It covers everything from designing the SD-WAN architecture and spinning up EVE-NG, to advanced routing, policy configuration, and full automation using **Ansible**, **Python**, **Postman**, and **Terraform**.

### What You Will Learn

| Area | Topics |
|------|--------|
| 🏗️ **Design** | SD-WAN fabric, control vs data plane, component roles, topology design |
| 🖥️ **Build** | EVE-NG image import, control plane bring-up, edge onboarding |
| ⚙️ **Configure** | Templates, routing (OMP/BGP/OSPF), centralized policies |
| 🚀 **Advanced** | AAR, QoS, Cloud OnRamp, service chaining, security |
| 🤖 **Automate** | REST API, Ansible, Python, Terraform |
| 🔍 **Operate** | Monitoring dashboards, CLI troubleshooting, telemetry |

---

## 🗺️ Lab Topology

```
                        ┌─────────────────────────────────────────┐
                        │           vManage (Management)           │
                        │         192.168.100.10/24                │
                        └──────────────┬──────────────────────────┘
                                       │ OMP / NETCONF
                        ┌──────────────┼──────────────────────────┐
                        │             │                            │
               ┌────────▼──────┐  ┌───▼────────┐          ┌──────▼──────┐
               │   vSmart-1    │  │   vBond    │          │   vSmart-2  │
               │ 10.0.0.11/24  │  │ 10.0.0.12  │          │ 10.0.0.13   │
               └───────────────┘  └────────────┘          └─────────────┘
                        │                │
          ┌─────────────┼────────────────┼──────────────────┐
          │             │                │                   │
   ┌──────▼──────┐  ┌───▼──────────┐  ┌─▼────────────┐  ┌──▼──────────┐
   │  vEdge-HQ   │  │  vEdge-BR1   │  │  vEdge-BR2   │  │ cEdge-DC1   │
   │ (Head Qtr)  │  │  (Branch 1)  │  │  (Branch 2)  │  │ (Data Ctr)  │
   │ 10.1.0.1    │  │  10.2.0.1    │  │  10.3.0.1    │  │ 10.4.0.1    │
   └──────┬──────┘  └───────┬──────┘  └──────┬───────┘  └──────┬──────┘
          │                 │                 │                  │
    ┌─────▼─────┐     ┌─────▼─────┐    ┌─────▼─────┐    ┌──────▼──────┐
    │  LAN HQ   │     │  LAN BR1  │    │  LAN BR2  │    │   LAN DC    │
    │172.16.1.0 │     │172.16.2.0 │    │172.16.3.0 │    │172.16.4.0   │
    └───────────┘     └───────────┘    └───────────┘    └─────────────┘

          ════════════════ IPSec Tunnels (BFD monitored) ════════════════
          ─ ─ ─ ─ ─ ─ ─ ─ OMP Control Plane (via vSmart)  ─ ─ ─ ─ ─ ─ ─
```

### Transport Colors

| Color | Transport | Interface |
|-------|-----------|-----------|
| `mpls` | MPLS Leased Line | ge0/0 |
| `biz-internet` | Business Internet | ge0/1 |
| `public-internet` | Broadband Internet | ge0/2 |

---

## ✅ Prerequisites

### Software

| Component | Version |
|-----------|---------|
| EVE-NG | Pro 5.x or Community 2.x |
| vManage | 20.9+ |
| vSmart | 20.9+ |
| vBond | 20.9+ |
| vEdge Cloud | 20.9+ |
| CSR1000v (cEdge) | IOS-XE 17.x |

### Host Requirements

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| RAM | 32 GB | 64 GB |
| CPU | 8 cores | 16 cores |
| Disk | 200 GB SSD | 500 GB NVMe |

### Automation Tools

```bash
pip3 install requests netmiko pandas tabulate python-dotenv
ansible-galaxy collection install cisco.catalystwan
terraform --version  # 1.5+
```

---

## ⚡ Quick Start

```bash
git clone https://github.com/YOUR_USERNAME/building-cisco-sdwan-solution.git
cd building-cisco-sdwan-solution
pip3 install -r automation/python/requirements.txt
cp automation/.env.example automation/.env
# Edit .env with your vManage IP and credentials
python3 automation/python/sdwan_api/auth.py
```

---

## 📚 Module Index

| # | Module | Duration | Topics |
|---|--------|----------|--------|
| 01 | [🏗️ Architecture Overview](docs/01-architecture/README.md) | ~10 min | SD-WAN components, fabric, use cases |
| 02 | [🖥️ EVE-NG Lab Setup](docs/02-eve-ng-setup/README.md) | ~15 min | Image import, topology, mgmt networking |
| 03 | [⚙️ Control Plane Bring-Up](docs/03-control-plane/README.md) | ~20 min | vBond, vManage, vSmart, OMP, certificates |
| 04 | [📦 vEdge / cEdge Onboarding](docs/04-edge-onboarding/README.md) | ~15 min | Bootstrap, ZTP, TLOC, BFD |
| 05 | [🗂️ vManage Templates](docs/05-vmanage-templates/README.md) | ~15 min | Feature templates, device templates |
| 06 | [🔌 Underlay & Transport](docs/06-underlay-transport/README.md) | ~15 min | VPN 0, TLOC, colors, IPSec tunnels |
| 07 | [🔀 Service VPN & Segmentation](docs/07-service-vpn/README.md) | ~10 min | VPN 1+, multi-VRF, DHCP, OMP routes |
| 08 | [🗺️ Routing Protocols](docs/08-routing/README.md) | ~20 min | OMP, BGP, OSPF, static, redistribution |
| 09 | [📜 Centralized Policies](docs/09-centralized-policies/README.md) | ~15 min | Control policy, data policy, hub-spoke |
| 10 | [🚀 Advanced Features](docs/10-advanced-features/README.md) | ~15 min | AAR, QoS, NAT/DIA, Cloud OnRamp |
| 11 | [📊 Monitoring & Operations](docs/11-monitoring/README.md) | ~10 min | Dashboards, telemetry, CLI troubleshooting |
| 12 | [🤖 Automation Layer](docs/12-automation/README.md) | ~25 min | REST API, Python, Ansible, Terraform |
| 13 | [🔍 Troubleshooting Guide](docs/13-troubleshooting/README.md) | Reference | Issues, debug commands, fixes |

---

## 🤖 Automation Modules

| Tool | What It Automates |
|------|-------------------|
| [🔌 Postman / REST API](automation/postman/README.md) | API auth, device ops, template & policy endpoints |
| [🐍 Python](automation/python/README.md) | vManage API wrapper, onboarding, stats, Netmiko CLI |
| [📘 Ansible](automation/ansible/README.md) | Onboard devices, push templates, deploy policies |
| [🟪 Terraform](automation/terraform/README.md) | Declarative SD-WAN template & policy management |

---

## 📁 Repository Structure

```
building-cisco-sdwan-solution/
├── README.md
├── docs/
│   ├── 01-architecture/README.md
│   ├── 02-eve-ng-setup/README.md
│   ├── 03-control-plane/README.md
│   ├── 04-edge-onboarding/README.md
│   ├── 05-vmanage-templates/README.md
│   ├── 06-underlay-transport/README.md
│   ├── 07-service-vpn/README.md
│   ├── 08-routing/README.md
│   ├── 09-centralized-policies/README.md
│   ├── 10-advanced-features/README.md
│   ├── 11-monitoring/README.md
│   ├── 12-automation/README.md
│   └── 13-troubleshooting/README.md
├── automation/
│   ├── .env.example
│   ├── ansible/
│   ├── python/
│   ├── postman/
│   └── terraform/
├── configs/
│   ├── day0-bootstrap/
│   ├── feature-templates/
│   └── policy-templates/
└── topology/
```

---

## 🧩 Cisco SD-WAN Component Summary

| Component | Role | Protocol | Port |
|-----------|------|----------|------|
| **vManage** | NMS / GUI / API | NETCONF, REST | 8443 |
| **vSmart** | Control Plane Controller | OMP, DTLS/TLS | 12346 |
| **vBond** | Orchestrator / NAT helper | DTLS | 12346 |
| **vEdge** | Data Plane (Viptela OS) | IPSec, BFD | various |
| **cEdge** | Data Plane (IOS-XE) | IPSec, BFD | various |

---

## 🤝 Contributing

1. Fork the repo
2. Create a branch: `git checkout -b fix/module-name`
3. Commit: `git commit -m "Description of fix"`
4. Open a Pull Request

---

## 📄 License

MIT License — see [LICENSE](LICENSE) for details.

> **Disclaimer:** For educational purposes only. All Cisco SD-WAN images require a valid Cisco software license.

---

<div align="center">

Made with ❤️ for the networking community · ⭐ Star this repo if it helped you!

</div>
