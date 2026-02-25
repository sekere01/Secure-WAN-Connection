# 🔐 Secure WAN Connection — Ghana & Cape Verde Branch Offices

> A network simulation project built with **Cisco Packet Tracer** demonstrating secure WAN connectivity between two geographically separated branch offices using ACL-based traffic filtering.

---

## 📌 Project Overview

This project simulates a real-world enterprise WAN deployment connecting a Ghana office and a Cape Verde office over a dedicated WAN link. Security is enforced through **Extended Access Control Lists (ACLs)** that permit only HTTP (port 80) and ICMP traffic, blocking all other protocols including FTP and Telnet.

| Property | Details |
|---|---|
| **Tool** | Cisco Packet Tracer |
| **Topology** | Star Topology (per branch) |
| **Routing** | Static Routing |
| **Security** | Extended ACL (access-list 100) |
| **Allowed Traffic** | HTTP (TCP port 80), ICMP (Ping) |
| **Blocked Traffic** | FTP, Telnet, and all other IP traffic |

---

## 🗺️ Network Topology

```
                    GHANA OFFICE                              CAPE VERDE OFFICE
                    ─────────────                             ─────────────────
  [Server 192.168.1.6]                                           [Server 192.168.2.6]
  [PC1    192.168.1.2]                                           [PC1    192.168.2.2]
  [PC2    192.168.1.3]  ──► [GHANA SWITCH] ──► [GHANA ROUTER]══WAN══[CV ROUTER] ◄── [CV SWITCH] ◄──  [PC2    192.168.2.3]
  [PC3    192.168.1.4]                        192.168.1.1  10.1.1.1──10.1.1.2  192.168.2.1          [PC3    192.168.2.4]
  [PC4    192.168.1.5]                                                                               [PC4    192.168.2.5]
```

Each branch uses a **Star Topology** where all end devices connect to a central switch, which uplinks to the branch router. The two routers are connected over a `/30` WAN link.

---

## 📋 IP Addressing Scheme

### Ghana Office — Subnet: `192.168.1.0/29`

| Device | IP Address | Subnet Mask | Default Gateway |
|---|---|---|---|
| Ghana Router (LAN) | 192.168.1.1 | 255.255.255.248 | — |
| Ghana PC1 | 192.168.1.2 | 255.255.255.248 | 192.168.1.1 |
| Ghana PC2 | 192.168.1.3 | 255.255.255.248 | 192.168.1.1 |
| Ghana PC3 | 192.168.1.4 | 255.255.255.248 | 192.168.1.1 |
| Ghana PC4 | 192.168.1.5 | 255.255.255.248 | 192.168.1.1 |
| Ghana Server | 192.168.1.6 | 255.255.255.248 | 192.168.1.1 |

### Cape Verde Office — Subnet: `192.168.2.0/29`

| Device | IP Address | Subnet Mask | Default Gateway |
|---|---|---|---|
| Cape Verde Router (LAN) | 192.168.2.1 | 255.255.255.248 | — |
| Cape Verde PC1 | 192.168.2.2 | 255.255.255.248 | 192.168.2.1 |
| Cape Verde PC2 | 192.168.2.3 | 255.255.255.248 | 192.168.2.1 |
| Cape Verde PC3 | 192.168.2.4 | 255.255.255.248 | 192.168.2.1 |
| Cape Verde PC4 | 192.168.2.5 | 255.255.255.248 | 192.168.2.1 |
| Cape Verde Server | 192.168.2.6 | 255.255.255.248 | 192.168.2.1 |

### WAN Link — Subnet: `10.1.1.0/30`

| Device | IP Address | Subnet Mask |
|---|---|---|
| Ghana Router (WAN) | 10.1.1.1 | 255.255.255.252 |
| Cape Verde Router (WAN) | 10.1.1.2 | 255.255.255.252 |

---

## ⚙️ Router Configuration

### Interface Setup

**Ghana Router**
```cisco
interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.248
 no shutdown

interface Serial0/0/0
 ip address 10.1.1.1 255.255.255.252
 no shutdown
```

**Cape Verde Router**
```cisco
interface GigabitEthernet0/0
 ip address 192.168.2.1 255.255.255.248
 no shutdown

interface Serial0/0/0
 ip address 10.1.1.2 255.255.255.252
 no shutdown
```

### Static Routing

```cisco
! On Ghana Router
ip route 192.168.2.0 255.255.255.248 10.1.1.2

! On Cape Verde Router
ip route 192.168.1.0 255.255.255.248 10.1.1.1
```

---

## 🔒 Security — ACL Configuration

An **Extended ACL (access-list 100)** is applied **inbound** on the WAN-facing interface of both routers to control inter-office traffic.

### ACL Rules

```cisco
! Permit HTTP traffic (TCP port 80)
access-list 100 permit tcp any any eq 80

! Permit return/established TCP sessions (HTTP responses)
access-list 100 permit tcp any any established

! Permit ICMP (ping) for connectivity testing
access-list 100 permit icmp any any

! Deny all other IP traffic (implicit deny made explicit)
access-list 100 deny ip any any
```

### Applying the ACL

```cisco
! Applied on both routers — inbound on WAN interface
interface Serial0/0/0
 ip access-group 100 in
```

> **Why inbound on the WAN interface?**  
> Applying the ACL inbound on the WAN-facing serial interface means traffic is filtered *as it arrives* from the remote branch, before it is forwarded into the local LAN. This is the most efficient placement and follows the principle of filtering traffic as close to the source as possible.

---

## ✅ Testing & Verification

### ICMP — Ping Tests

All ping tests between Ghana and Cape Verde passed with **0% packet loss**.

| Source | Destination | Result |
|---|---|---|
| Ghana PC1 (192.168.1.2) | Cape Verde Router (192.168.2.1) | ✅ Success |
| Ghana PC1 | Cape Verde PC1–PC4 | ✅ Success |
| Ghana PC1 | Cape Verde Server (192.168.2.6) | ✅ Success |
| Cape Verde PC1 (192.168.2.2) | Ghana Router (192.168.1.1) | ✅ Success |
| Cape Verde PC1 | Ghana PC1–PC4 | ✅ Success |
| Cape Verde PC1 | Ghana Server (192.168.1.6) | ✅ Success |

### HTTP — Web Browser Test (Port 80)

| Source | Destination | URL | Result |
|---|---|---|---|
| Ghana PC1 | Cape Verde Server | http://192.168.2.6 | ✅ Page Loaded |
| Cape Verde PC1 | Ghana Server | http://192.168.1.6 | ✅ Page Loaded |

### Blocked Protocols — FTP & Telnet

| Source | Destination | Protocol | Result |
|---|---|---|---|
| Cape Verde PC | Ghana Server (192.168.1.6) | FTP | ❌ Timed Out |
| Ghana PC | Cape Verde Server (192.168.2.6) | FTP | ❌ Timed Out |
| Cape Verde PC | Ghana Router (192.168.1.1) | Telnet | ❌ Timed Out |
| Ghana PC | Cape Verde Router (192.168.2.1) | Telnet | ❌ Timed Out |

All blocked protocols confirmed **denied** by the ACL as expected. ✅

---

## 📁 Repository Structure

```
📦 secure-wan-ghana-capeverde/
 ┣ 📄 README.md                          ← This file
 ┣ 📦 Project_1_Altschool.pkt            ← Cisco Packet Tracer simulation file
 ┗ 📄 Project_Documentation.pdf          ← Full project documentation with screenshots
```

---

## 🧠 Key Concepts Demonstrated

- **Subnetting** with CIDR notation (`/29` for LANs, `/30` for WAN point-to-point link)
- **Static Routing** between two remote networks
- **Extended ACLs** for granular Layer 3 & Layer 4 traffic control
- **WAN link design** using serial interfaces
- **Network security principles** — least privilege access (allow only what is needed)
- **Verification methodology** — testing both permitted and denied traffic

---

## 🛠️ Tools Used

- [Cisco Packet Tracer](https://www.netacad.com/cisco-packet-tracer) — Network simulation
- Cisco IOS CLI — Router configuration

---

## 👤 Author

Oluwafisayomi
Cybersecurity Student | AltSchool Africa  

---

## 📄 License

This project is for educational purposes. Feel free to fork, reference, or build upon it.
