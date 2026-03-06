# 🗺️ Module 08 — Routing Protocols

[← Service VPN](../07-service-vpn/README.md) | [Back to Main](../../README.md) | [Next: Centralized Policies →](../09-centralized-policies/README.md)

---

## 🎯 Learning Objectives

- Master OMP route types and redistribution
- Configure BGP and OSPF integration with SD-WAN
- Implement static routes and inter-VPN leaking
- Verify end-to-end routing across the fabric

---

## 8.1 OMP Route Types

| Type | CLI | Source |
|------|-----|--------|
| OMP routes | `show sdwan omp routes` | Connected, Static, BGP, OSPF |
| TLOC routes | `show sdwan omp tlocs` | Tunnel interfaces |
| Service routes | `show sdwan omp services` | Service VPN configs |

---

## 8.2 OMP Configuration

```bash
vEdge# config
vEdge(config)# omp
vEdge(config-omp)#  send-path-limit  4
vEdge(config-omp)#  ecmp-limit       4
vEdge(config-omp)#  graceful-restart
vEdge(config-omp)#  advertise connected
vEdge(config-omp)#  advertise static
vEdge(config-omp)#  commit
```

---

## 8.3 BGP Integration

```bash
vEdge-BR1# config
vEdge-BR1(config)# vpn 1
vEdge-BR1(config-vpn-1)#  router bgp 65002
vEdge-BR1(config-router-bgp-65002)#  router-id 10.2.0.1
vEdge-BR1(config-router-bgp-65002)#  address-family ipv4-unicast
vEdge-BR1(config-address-family-ipv4-unicast)#   redistribute omp
vEdge-BR1(config-router-bgp-65002)#  neighbor 172.16.2.254
vEdge-BR1(config-neighbor-172.16.2.254)#   remote-as 65200
vEdge-BR1(config-neighbor-172.16.2.254)#   address-family ipv4-unicast
vEdge-BR1(config-address-family-ipv4-unicast)#    activate

# Redistribute BGP into OMP
vEdge-BR1(config)# omp
vEdge-BR1(config-omp)#  advertise bgp
vEdge-BR1(config-omp)#  commit
```

### BGP Verification

```bash
show sdwan bgp summary vpn 1
show sdwan bgp routes vpn 1
```

---

## 8.4 OSPF Integration

```bash
vEdge-HQ# config
vEdge-HQ(config)# vpn 1
vEdge-HQ(config-vpn-1)#  router ospf
vEdge-HQ(config-router-ospf)#   router-id 10.1.0.1
vEdge-HQ(config-router-ospf)#   redistribute omp
vEdge-HQ(config-router-ospf)#   area 0
vEdge-HQ(config-area-0)#         interface ge0/2

# Redistribute OSPF into OMP
vEdge-HQ(config)# omp
vEdge-HQ(config-omp)#  advertise ospf intra
vEdge-HQ(config-omp)#  advertise ospf external
vEdge-HQ(config-omp)#  commit
```

---

## 8.5 Static Routes & Route Leaking

```bash
# Static route in service VPN
vEdge(config)# vpn 1
vEdge(config-vpn-1)#  ip route 192.168.50.0/24 172.16.1.254

# Inter-VPN route leaking (VPN 2 → VPN 1 shared service)
vEdge(config)# vpn 2
vEdge(config-vpn-2)#  ip route 10.100.0.0/24 vpn 1
vEdge(config-vpn-2)#  commit
```

---

## 8.6 Routing Verification

```bash
show sdwan omp routes
show sdwan omp tlocs
show sdwan ip route vpn 1
show sdwan omp routes 10.1.0.0/24
show sdwan bgp summary vpn 1
show sdwan ospf neighbor vpn 1
```

---

## ✅ Checklist

- [ ] OMP routes visible on all vEdges
- [ ] BGP peering established with LAN routers
- [ ] OSPF routes redistributed into OMP
- [ ] End-to-end ping succeeds between all sites
- [ ] Routing table shows correct next-hops (TLOC)

---

[← Service VPN](../07-service-vpn/README.md) | [Back to Main](../../README.md) | [Next: Centralized Policies →](../09-centralized-policies/README.md)
