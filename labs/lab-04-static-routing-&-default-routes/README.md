# Lab 4 — Static Routing & Default Routes

**CCNA Lab Series | Cisco Packet Tracer | Junior Level**

---

## Overview

This lab builds a three-router network from scratch and configures static routing to connect three separate office LANs. It covers four core skills that form the foundation of enterprise routing: manual static routes, default routes, floating static routes for failover, and route verification using Cisco IOS commands.

All configuration was performed in Cisco Packet Tracer using Cisco 2911 routers and 2960 switches. The full lab report with annotated CLI screenshots is included as a PDF in this repository.

---

## What This Lab Demonstrates

**Static Routing** — Manually telling each router how to reach networks it isn't directly connected to by specifying a destination network and the next-hop IP address. This is the most basic and predictable form of routing, commonly used in small networks and stub connections.

**Default Routes** — Replacing multiple individual static routes with a single catch-all entry (0.0.0.0/0) that matches any destination not already in the routing table. This simplifies configuration on edge routers that only have one upstream path, and acts as the "gateway of last resort."

**Floating Static Routes** — Creating a backup route with a higher Administrative Distance so it stays hidden during normal operation. If the primary path fails, the backup automatically activates without any manual intervention — providing basic path redundancy.

**Route Verification** — Using `show ip route`, `ping`, and `tracert` to confirm that routes are installed correctly, traffic flows end-to-end, and packets traverse the expected path through the network.

---

## Network Design

The topology consists of three routers connected in a chain. Each router serves one LAN segment through a switch. The WAN links between routers use /30 subnets (2 usable addresses each), while each LAN uses a /24 subnet (254 usable addresses).

**Equipment used:** 3× Cisco 2911 Routers, 3× Cisco 2960-24TT Switches, 3× PCs

### Addressing Scheme

| Device | Interface | IP Address | Subnet | Purpose |
|--------|-----------|------------|--------|---------|
| R1 | G0/0 | 192.168.12.1 | /30 | WAN link to R2 |
| R1 | G0/1 | 192.168.1.1 | /24 | LAN-A gateway |
| R2 | G0/0 | 192.168.12.2 | /30 | WAN link to R1 |
| R2 | G0/1 | 192.168.23.1 | /30 | WAN link to R3 |
| R2 | G0/2 | 192.168.2.1 | /24 | LAN-B gateway |
| R3 | G0/0 | 192.168.23.2 | /30 | WAN link to R2 |
| R3 | G0/1 | 192.168.3.1 | /24 | LAN-C gateway |
| PC1 | NIC | 192.168.1.10 | /24 | Default gateway: 192.168.1.1 |
| PC2 | NIC | 192.168.2.10 | /24 | Default gateway: 192.168.2.1 |
| PC3 | NIC | 192.168.3.10 | /24 | Default gateway: 192.168.3.1 |

---

## Configuration Summary

### Step 1 — Interface Configuration

Each router was assigned a hostname, had its interfaces configured with IP addresses and subnet masks, and all interfaces were enabled. Configurations were saved to NVRAM using `write memory`.

### Step 2 — Static Routes

Static routes were added on all three routers so each one knows how to reach every remote network. R1 forwards all remote traffic toward R2. R2 has routes in both directions — toward R1 for LAN-A and toward R3 for LAN-C. R3 sends all return traffic toward R2.

**R1 routes:**
```
ip route 192.168.2.0 255.255.255.0 192.168.12.2
ip route 192.168.3.0 255.255.255.0 192.168.12.2
ip route 192.168.23.0 255.255.255.0 192.168.12.2
```

**R2 routes:**
```
ip route 192.168.1.0 255.255.255.0 192.168.12.1
ip route 192.168.3.0 255.255.255.0 192.168.23.2
```

**R3 routes:**
```
ip route 192.168.1.0 255.255.255.0 192.168.23.1
ip route 192.168.2.0 255.255.255.0 192.168.23.1
ip route 192.168.12.0 255.255.255.252 192.168.23.1
```

### Step 3 — Verification

Connectivity was tested from multiple endpoints. PC1 successfully pinged both PC2 and PC3. PC2 confirmed bidirectional reachability to PC1 and PC3. A traceroute from PC1 to PC3 confirmed the correct path: R1 → R2 → R3 → PC3 (four hops). The routing table on R1 was inspected with `show ip route` and showed all expected static (S) and connected (C) entries.

### Step 4 — Default Route

The three individual static routes on R1 were removed and replaced with a single default route. This tells R1 to forward all unknown traffic to R2, which then makes more specific forwarding decisions.

```
ip route 0.0.0.0 0.0.0.0 192.168.12.2
```

After this change, `show ip route` displayed `S* 0.0.0.0/0 [1/0] via 192.168.12.2` and confirmed the gateway of last resort was set. All connectivity tests still passed.

### Step 5 — Floating Static Route

A direct backup link was added between R1 and R3 (192.168.13.0/30). Two routes to LAN-C were then configured on R1: a primary route through R2 with the default Administrative Distance of 1, and a floating backup route through the direct R3 link with an AD of 10.

```
ip route 192.168.3.0 255.255.255.0 192.168.12.2          ! Primary — AD 1
ip route 192.168.3.0 255.255.255.0 192.168.13.2 10       ! Backup  — AD 10
```

Under normal conditions, only the primary route appears in the routing table. If the R1–R2 link goes down, the floating route automatically activates and traffic reroutes through the direct R1–R3 path.

---

## Key Concepts

**Administrative Distance (AD)** is the trustworthiness metric routers use to choose between route sources. Lower AD means more preferred. Directly connected routes have AD 0, static routes have AD 1, OSPF has AD 110, and RIP has AD 120. Floating static routes work by intentionally setting a higher AD so they only become active when the preferred route disappears.

**Bidirectional routing** is essential — if Router A has a route to reach Router C's network, but Router C has no return route back to Router A, the reply packets will be dropped. Every static route needs a corresponding return path.

**Next-hop vs exit interface** are two ways to specify where a router should forward traffic. Next-hop (used in this lab) specifies the IP address of the next router. Exit interface specifies the local interface to send packets out of. Next-hop is generally preferred because it provides more precise forwarding behavior.

---

## Results

All lab objectives were completed successfully:

- All router interfaces configured with correct IP addresses and enabled
- Static routes established full end-to-end connectivity across all three LANs
- Ping tests confirmed reachability between all PCs in both directions
- Traceroute verified the correct forwarding path through R1, R2, and R3
- Default route replaced individual statics on R1 and functioned correctly
- Floating static route configured with AD 10 as backup to the primary AD 1 path
- All configurations saved to NVRAM

---

## Files in This Repository

| File | Description |
|------|-------------|
| `README.md` | This document — lab summary and configuration details |
| `Lab_4_Report.pdf` | Full lab report with annotated Packet Tracer screenshots |
| `screenshots/` | All CLI and topology screenshots from the lab |

---

## Tools & Prerequisites

- Cisco Packet Tracer 8.x
- Completion of Lab 1 (Basic Device Configuration)
- Understanding of IP addressing and subnetting

---

*Lab 4 complete. Static routing is foundational — dynamic routing protocols like OSPF build directly on these concepts.*
