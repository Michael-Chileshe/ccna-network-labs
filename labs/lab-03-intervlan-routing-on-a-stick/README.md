# Lab 03: Inter-VLAN Routing (Router-on-a-Stick)

## Metadata

| Field | Value |
|-------|-------|
| **CCNA Topic** | 3.5 – Configure and verify Layer 3 routing between VLANs |
| **Exam Objective** | Configure router-on-a-stick inter-VLAN routing |
| **Difficulty** | Junior |
| **Tools** | Cisco Packet Tracer 8.x |
| **Date Completed** | 2026-02-19 |
| **Author** | Michael Chileshe |

---

## Lab Objective

VLANs isolate traffic — but users in different VLANs still need to communicate. In this lab we added a Cisco 2911 router to the Lab 02 topology, configured Router-on-a-Stick (ROAS) using subinterfaces with 802.1Q encapsulation, updated all PC default gateways, and verified that traffic now flows between VLANs through the router. The Layer 3 switch SVI alternative was also implemented and demonstrated. Both methods are tested on the CCNA exam.

---

## Topology

```
              [R1] — 2911 Router
           G0/0.10 → 192.168.10.1
           G0/0.20 → 192.168.20.1
           G0/0.99 → 192.168.99.1
                |
                | G0/0 (no IP — trunk link)
                |
             [SW1] ========TRUNK (Gi0/1)======== [SW2]
          Gi0/2 trunk                           /       \
          native 99                          Fa0/1     Fa0/2
          /       \                            |           |
       Fa0/1     Fa0/2                      [PC-C]      [PC-D]
         |           |                     VLAN 10     VLAN 20
      [PC-A]       [PC-B]              192.168.10.11  192.168.20.11
      VLAN 10      VLAN 20             GW: 10.1       GW: 20.1
   192.168.10.10 192.168.20.10
   GW: 10.1      GW: 20.1
```

**The "one stick":** R1 uses a single physical link (G0/0) to SW1. Subinterfaces G0/0.10, G0/0.20, and G0/0.99 each handle one VLAN — that is what "Router-on-a-Stick" means.

---

## Cabling

| From | To | Cable | Notes |
|------|----|-------|-------|
| R1 G0/0 | SW1 Gi0/2 | Straight-through | New trunk link for router |
| SW1 Gi0/1 | SW2 Gi0/1 | Straight-through | Inter-switch trunk (from Lab 02) |
| SW1 Fa0/1 | PC-A | Straight-through | Access VLAN 10 |
| SW1 Fa0/2 | PC-B | Straight-through | Access VLAN 20 |
| SW2 Fa0/1 | PC-C | Straight-through | Access VLAN 10 |
| SW2 Fa0/2 | PC-D | Straight-through | Access VLAN 20 |

---

## IP Addressing Table

| Device | Interface | IP Address | Subnet Mask | Default Gateway | VLAN |
|--------|-----------|------------|-------------|-----------------|------|
| R1 | G0/0 | unassigned | — | N/A | Trunk |
| R1 | G0/0.10 | 192.168.10.1 | 255.255.255.0 | N/A | 10 |
| R1 | G0/0.20 | 192.168.20.1 | 255.255.255.0 | N/A | 20 |
| R1 | G0/0.99 | 192.168.99.1 | 255.255.255.0 | N/A | 99 |
| PC-A | Fa0 | 192.168.10.10 | 255.255.255.0 | 192.168.10.1 | 10 |
| PC-B | Fa0 | 192.168.20.10 | 255.255.255.0 | 192.168.20.1 | 20 |
| PC-C | Fa0 | 192.168.10.11 | 255.255.255.0 | 192.168.10.1 | 10 |
| PC-D | Fa0 | 192.168.20.11 | 255.255.255.0 | 192.168.20.1 | 20 |

> **Key change from Lab 02:** PCs now have a default gateway — the router subinterface IP for their VLAN. Without a gateway, inter-VLAN traffic has nowhere to go.

---

## Configuration

### SW1 — Add Trunk Port for Router (Gi0/2)

```
enable
configure terminal
interface GigabitEthernet0/2
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,99
 no shutdown
 exit
wr
```

### R1 — Router-on-a-Stick Subinterface Configuration

```
enable
configure terminal
hostname R1

! Physical interface — no IP, just bring it up
interface GigabitEthernet0/0
 no shutdown
 exit

! VLAN 10 — SALES gateway
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 exit

! VLAN 20 — ENGINEERING gateway
interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 exit

! VLAN 99 — MANAGEMENT (native keyword required)
interface GigabitEthernet0/0.99
 encapsulation dot1Q 99 native
 ip address 192.168.99.1 255.255.255.0
 exit

end
wr
```

---

## Verification

### show ip interface brief — R1

```
Interface              IP-Address      OK? Method Status    Protocol
GigabitEthernet0/0     unassigned      YES unset  up        up
GigabitEthernet0/0.10  192.168.10.1   YES manual up        up
GigabitEthernet0/0.20  192.168.20.1   YES manual up        up
GigabitEthernet0/0.99  192.168.99.1   YES manual up        up
```

### show ip route — R1

```
C   192.168.10.0/24 is directly connected, GigabitEthernet0/0.10
L   192.168.10.1/32 is directly connected, GigabitEthernet0/0.10
C   192.168.20.0/24 is directly connected, GigabitEthernet0/0.20
L   192.168.20.1/32 is directly connected, GigabitEthernet0/0.20
C   192.168.99.0/24 is directly connected, GigabitEthernet0/0.99
L   192.168.99.1/32 is directly connected, GigabitEthernet0/0.99
```

### show interfaces G0/0.10 — R1

```
GigabitEthernet0/0.10 is up, line protocol is up (connected)
  Encapsulation 802.1Q Virtual LAN, Vlan ID 10
  Internet address is 192.168.10.1/24
```

### Ping Tests — PC-A (VLAN 10)

```
C:\>ping 192.168.10.11    ! PC-C — same VLAN 10 (switched, not routed)
Reply from 192.168.10.11: TTL=128
Packets: Sent=4, Received=4, Lost=0 (0% loss) ✅

C:\>ping 192.168.20.10    ! PC-B — cross VLAN via router
Reply from 192.168.20.10: TTL=127
Packets: Sent=4, Received=3, Lost=1 (25% loss) ✅ First packet = ARP delay

C:\>ping 192.168.20.11    ! PC-D — cross VLAN via router
Reply from 192.168.20.11: TTL=127
Packets: Sent=4, Received=3, Lost=1 (25% loss) ✅ TTL=127 confirms routing

C:\>tracert 192.168.20.10
  1  192.168.10.1   (R1 — the router)
  2  192.168.20.10  (PC-B)
Trace complete. ✅
```

> **TTL diagnostic:** Same-VLAN pings return TTL=128 (Layer 2 switch, no router hop). Cross-VLAN pings return TTL=127 (one router hop reduces TTL by 1). This is a fast, reliable confirmation that inter-VLAN routing is working.

---

## Bonus: Layer 3 Switch SVI Alternative

A Cisco 3560-24PS Multilayer Switch was also configured to demonstrate the SVI routing method — going beyond the lab requirements.

```
! On a Cisco 3560 Multilayer Switch:
enable
configure terminal
ip routing

interface vlan 10
 ip address 192.168.10.1 255.255.255.0
 no shutdown
 exit

interface vlan 20
 ip address 192.168.20.1 255.255.255.0
 no shutdown
 exit
wr
```

### ROAS vs Layer 3 Switch Comparison

| Feature | Router-on-a-Stick | Layer 3 Switch (SVI) |
|---------|------------------|----------------------|
| Hardware | External 2911 router | 3560 multilayer switch |
| Routing speed | Software / CPU | Hardware ASIC (faster) |
| Bandwidth | Single uplink bottleneck | Full switch backplane |
| Config | Subinterfaces + dot1Q | `ip routing` + `interface vlan` |
| Best use | Small networks / labs | Enterprise production |
| CCNA exam | ✅ Tested | ✅ Tested |

---

## Troubleshooting Notes

### Issue 1 — Interface Command at Privileged EXEC
**Problem:** Typed `SW1#int g0/2` at privileged EXEC without entering `configure terminal` first.
**Symptom:** `% Invalid input detected at '^' marker.`
**Fix:** Entered `conf t` first, then `int g0/2`. Interface configuration commands only work inside global config mode.

### Issue 2 — `wr` Inside Global Config Mode
**Problem:** Typed `R1(config)#wr` while still inside `configure terminal`.
**Symptom:** `% Invalid input detected at '^' marker.`
**Fix:** Used `end` to return to privileged EXEC (`R1#`), then ran `wr`. The save command must be run from privileged EXEC.

### Issue 3 — `sh ip rt` Abbreviation Not Recognised
**Problem:** Used `sh ip rt` as shorthand for `show ip route`.
**Symptom:** `% Invalid input detected at '^' marker.`
**Fix:** Used full command `show ip route`. IOS abbreviations only work when the partial string is unambiguous — `rt` was ambiguous here.

---

## Key Takeaways

- Router-on-a-Stick uses one physical interface with multiple logical subinterfaces — one per VLAN — each acting as that VLAN's default gateway.
- The physical interface (`G0/0`) has **no IP address**. Only the subinterfaces have IPs. The physical interface just needs `no shutdown`.
- `encapsulation dot1Q [vlan-id]` is mandatory on every subinterface — without it the router does not know which VLAN tag to process.
- The native VLAN subinterface requires the `native` keyword: `encapsulation dot1Q 99 native`.
- TTL=127 on cross-VLAN pings confirms routing occurred. TTL=128 means it was switched at Layer 2.
- First cross-VLAN ping often shows 25% loss due to ARP resolution — this is normal.
- `wr` and `copy run start` must always be run from privileged EXEC (`R1#`), not from inside config mode.
- For production networks, Layer 3 switches with SVIs are preferred over ROAS — routing happens in dedicated hardware, eliminating the single-link bottleneck.

---

## Lab Validation Checklist

- [x] R1 added and cabled to SW1 Gi0/2
- [x] SW1 Gi0/2: trunk mode, native 99, allowed 10,20,99
- [x] R1 G0/0: no shutdown, no IP address
- [x] R1 G0/0.10: encapsulation dot1Q 10, IP 192.168.10.1/24
- [x] R1 G0/0.20: encapsulation dot1Q 20, IP 192.168.20.1/24
- [x] R1 G0/0.99: encapsulation dot1Q 99 native, IP 192.168.99.1/24
- [x] All 4 PC gateways updated to their respective subinterface IPs
- [x] PC-A → PC-C (same VLAN): 0% loss, TTL=128 ✅
- [x] PC-A → PC-B (cross VLAN): success, TTL=127 confirms routing ✅
- [x] PC-A → PC-D (cross VLAN): success, TTL=127 confirms routing ✅
- [x] tracert confirms R1 as hop 1 for cross-VLAN traffic ✅
- [x] show ip route: all 3 subnets present via subinterfaces ✅
- [x] show interfaces G0/0.10: 802.1Q VLAN ID 10 confirmed ✅
- [x] Layer 3 switch SVI method demonstrated on Cisco 3560 (bonus) ✅

---

*Lab 03 Complete — Inter-VLAN routing is tested on every CCNA exam. Next: Lab 04 — Static Routing.*
