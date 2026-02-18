# Lab 02: VLAN Configuration & Trunk Links

## Metadata

| Field | Value |
|-------|-------|
| **CCNA Topic** | 2.1 – Configure and verify VLANs (normal range) |
| **Exam Objective** | Configure and verify interswitch connectivity |
| **Difficulty** | Junior |
| **Tools** | Cisco Packet Tracer 8.x |
| **Date Completed** | 2026-02-18 |
| **Author** | Your Name |

---

## Lab Objective

VLANs are how every enterprise network is segmented. Without VLANs, every device shares one broadcast domain — a security and performance problem. In this lab we created VLANs 10, 20, and 99 on two switches, assigned access ports to each VLAN, configured an 802.1Q trunk link between the switches, and verified that traffic is correctly isolated between VLANs. VLANs are the most heavily tested topic on the CCNA exam.

---

## Topology

```
   [PC-A]             [PC-C]
 VLAN 10              VLAN 10
192.168.10.10       192.168.10.11
    |                    |
   Fa0/1               Fa0/1
    |                    |
  [SW1] ====TRUNK==== [SW2]
   Gi0/1   802.1Q     Gi0/1
   Native VLAN 99
    |                    |
   Fa0/2               Fa0/2
    |                    |
 VLAN 20              VLAN 20
192.168.20.10       192.168.20.11
   [PC-B]             [PC-D]
```

**Cabling:**
- PC-A → SW1 Fa0/1 (straight-through)
- PC-B → SW1 Fa0/2 (straight-through)
- SW1 Gi0/1 → SW2 Gi0/1 (straight-through — trunk link)
- PC-C → SW2 Fa0/1 (straight-through)
- PC-D → SW2 Fa0/2 (straight-through)

---

## VLAN Table

| VLAN ID | Name | Purpose | Subnet |
|---------|------|---------|--------|
| 10 | SALES | PC-A and PC-C traffic | 192.168.10.0/24 |
| 20 | ENGINEERING | PC-B and PC-D traffic | 192.168.20.0/24 |
| 99 | MANAGEMENT | Native VLAN on trunk (security) | N/A |

---

## IP Addressing Table

| Device | IP Address | Subnet Mask | Default Gateway | VLAN |
|--------|-----------|-------------|-----------------|------|
| PC-A | 192.168.10.10 | 255.255.255.0 | N/A | 10 (SALES) |
| PC-B | 192.168.20.10 | 255.255.255.0 | N/A | 20 (ENGINEERING) |
| PC-C | 192.168.10.11 | 255.255.255.0 | N/A | 10 (SALES) |
| PC-D | 192.168.20.11 | 255.255.255.0 | N/A | 20 (ENGINEERING) |

> **Note:** No default gateway is configured — there is no router in this lab. Inter-VLAN routing is covered in Lab 03.

---

## Configuration

### SW1 — VLAN Creation

```
enable
configure terminal
hostname SW1
vlan 10
 name SALES
exit
vlan 20
 name ENGINEERING
exit
vlan 99
 name MANAGEMENT
exit
wr
```

### SW1 — Access Port Assignment

```
configure terminal
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 10
 no shutdown
 exit
interface FastEthernet0/2
 switchport mode access
 switchport access vlan 20
 no shutdown
 exit
wr
```

### SW1 — Trunk Port Configuration

```
configure terminal
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,99
 no shutdown
 exit
wr
```

### SW2 — VLAN Creation

```
enable
configure terminal
hostname SW2
vlan 10
 name SALES
exit
vlan 20
 name ENGINEERING
exit
vlan 99
 name MANAGEMENT
exit
wr
```

### SW2 — Access Port Assignment

```
configure terminal
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 10
 no shutdown
 exit
interface FastEthernet0/2
 switchport mode access
 switchport access vlan 20
 no shutdown
 exit
wr
```

### SW2 — Trunk Port Configuration

```
configure terminal
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,99
 no shutdown
 exit
wr
```

---

## Verification

### show vlan brief — SW1

```
VLAN  Name          Status    Ports
----  ------------- --------- ----------------------------
1     default       active    Fa0/3-24, Gi0/2
10    SALES         active    Fa0/1
20    ENGINEERING   active    Fa0/2
99    MANAGEMENT    active
```

### show interfaces trunk — SW1

```
Port     Mode   Encapsulation  Status    Native vlan
Gi0/1    on     802.1q         trunking  99

Port     Vlans allowed on trunk
Gi0/1    10,20,99

Port     Vlans allowed and active in management domain
Gi0/1    10,20,99

Port     Vlans in spanning tree forwarding state and not pruned
Gi0/1    10,20,99
```

### show interfaces Fa0/1 switchport — SW1

```
Name: Fa0/1
Administrative Mode: static access
Operational Mode: static access
Access Mode VLAN: 10 (SALES)
```

### VLAN Isolation Test — PC-A (VLAN 10)

```
C:\>ping 192.168.10.11   ! PC-C — same VLAN
Reply from 192.168.10.11: bytes=32 time=1ms TTL=128
Reply from 192.168.10.11: bytes=32 time<1ms TTL=128
Reply from 192.168.10.11: bytes=32 time<1ms TTL=128
Reply from 192.168.10.11: bytes=32 time<1ms TTL=128
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss) ✅ EXPECTED

C:\>ping 192.168.20.10   ! PC-B — different VLAN
Request timed out.
Request timed out.
Request timed out.
Request timed out.
Packets: Sent = 4, Received = 0, Lost = 4 (100% loss) ✅ EXPECTED
```

### VLAN Isolation Test — PC-B (VLAN 20)

```
C:\>ping 192.168.10.11   ! PC-C — different VLAN
Request timed out.
Request timed out.
Request timed out.
Request timed out.
Packets: Sent = 4, Received = 0, Lost = 4 (100% loss) ✅ EXPECTED

C:\>ping 192.168.20.11   ! PC-D — same VLAN
Reply from 192.168.20.11: bytes=32 time<1ms TTL=128
Reply from 192.168.20.11: bytes=32 time<1ms TTL=128
Reply from 192.168.20.11: bytes=32 time<1ms TTL=128
Reply from 192.168.20.11: bytes=32 time=7ms TTL=128
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss) ✅ EXPECTED
```

---

## Troubleshooting Notes

### Issue 1 — SW2 VLAN Name Typo
**Problem:** VLAN 10 on SW2 was initially named `SALEs` (incorrect capitalisation).  
**Symptom:** None immediately visible — IOS accepted the name since names are case-sensitive strings, not validated commands.  
**Fix:** Returned to global config, re-entered `vlan 10` → `name SALES` with correct capitalisation.  
**Lesson:** VLAN names must match exactly across switches for consistency in documentation and management tools.

### Issue 2 — Native VLAN Mismatch (CDP Error)
**Problem:** SW2's G0/1 was configured with native VLAN 99 while SW1's side still had native VLAN 1 (default).  
**Symptom:**
```
%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on GigabitEthernet0/1 (99), with SW1 FastEthernet0/3 (1).
%SPANTREE-2-BLOCK_PVID_LOCAL: Blocking GigabitEthernet0/1 on VLAN0099. Inconsistent local vlan.
```
**Fix:** Configured matching native VLAN 99 on SW1's trunk port.  
**Lesson:** The native VLAN **must match on both ends** of every trunk link. STP will block the port until resolved.

> **Note:** During PC-A isolation testing, 192.168.20.1 was pinged instead of 192.168.20.10 (PC-B). The ping timed out, but this was because the IP doesn't exist — not solely due to VLAN isolation. PC-B's tests correctly confirm isolation behaviour.

---

## Key Takeaways

- VLANs provide Layer 2 segmentation — without a router, devices in different VLANs cannot communicate at all.
- VLANs must be manually created on every switch that needs to carry that traffic (unless VTP is used — covered later).
- Access ports carry one untagged VLAN. Trunk ports carry multiple VLANs using 802.1Q tags.
- The native VLAN is sent across a trunk **untagged**. Both ends must agree or STP blocks the port.
- Setting native VLAN to 99 (unused) is a security best practice — it prevents VLAN hopping attacks that exploit the default native VLAN 1.
- `show interfaces trunk` is the most important verification command for trunk links.
- `show vlan brief` instantly shows which ports belong to which VLAN.

---

## Lab Validation Checklist

- [x] VLANs 10 (SALES), 20 (ENGINEERING), and 99 (MANAGEMENT) exist on SW1
- [x] VLANs 10 (SALES), 20 (ENGINEERING), and 99 (MANAGEMENT) exist on SW2
- [x] SW1 Fa0/1 → access VLAN 10 | Fa0/2 → access VLAN 20
- [x] SW2 Fa0/1 → access VLAN 10 | Fa0/2 → access VLAN 20
- [x] Trunk link up: SW1 Gi0/1 ↔ SW2 Gi0/1, 802.1Q, native VLAN 99
- [x] Trunk allows VLANs 10, 20, and 99 only
- [x] PC-A (VLAN 10) → PC-C (VLAN 10): 0% loss across trunk ✅
- [x] PC-A (VLAN 10) → PC-B (VLAN 20): 100% loss — VLAN isolation confirmed ✅
- [x] PC-B (VLAN 20) → PC-D (VLAN 20): 0% loss across trunk ✅
- [x] PC-B (VLAN 20) → PC-C (VLAN 10): 100% loss — VLAN isolation confirmed ✅
- [x] Native VLAN mismatch resolved — no CDP errors remaining
- [x] `show vlan brief` confirms port assignments
- [x] `show interfaces trunk` confirms 802.1Q, native 99, allowed 10,20,99

---

*Lab 02 Complete — VLANs are the #1 most tested CCNA topic. Master them.*
