# Lab 01: Device Configuration Fundamentals & IOS Navigation

## Metadata

| Field | Value |
|-------|-------|
| **CCNA Topic** | 1.1 – Cisco IOS Navigation, Device Hardening, Remote Management |
| **Exam Objective** | Configure device access security using local passwords |
| **Difficulty** | Junior |
| **Tools** | Cisco Packet Tracer 8.x |
| **Date Completed** | 2026-02-18 |
| **Author** | Michael Chileshe|

---

## Lab Objective

Configure a Cisco 2911 router and 2960 switch from scratch in Packet Tracer. This lab covers IOS mode navigation, hostname and MOTD banner configuration, password security (console, VTY, enable secret), SSH remote access setup, IP addressing, and essential verification show commands. These foundational skills underpin every subsequent CCNA lab.

---

## Topology

```
           [R1]  — 2911 Router
          192.168.1.1/24
              |
              | G0/0 ↔ G0/1
              |
           [SW1] — 2960 Switch
          VLAN1: 192.168.1.2/24
          GW: 192.168.1.1
           /        \
    Fa0/1           Fa0/2
       |               |
    [PC 1]          [PC 2]
  192.168.1.10   192.168.1.11
  /24 GW:.1.1    /24 GW:.1.1
```

**Cable Types used:**
- R1 G0/0 → SW1 G0/1 : Copper Straight-Through
- PC1 Fa0 → SW1 Fa0/1 : Copper Straight-Through
- PC2 Fa0 → SW1 Fa0/2 : Copper Straight-Through

---

## IP Addressing Table

| Device | Interface | IP Address | Subnet Mask | Default Gateway |
|--------|-----------|------------|-------------|-----------------|
| R1 | GigabitEthernet0/0 | 192.168.1.1 | 255.255.255.0 | N/A |
| SW1 | VLAN 1 | 192.168.1.2 | 255.255.255.0 | 192.168.1.1 |
| PC 1 | FastEthernet0 | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 |
| PC 2 | FastEthernet0 | 192.168.1.11 | 255.255.255.0 | 192.168.1.1 |

---

## Configuration

### Router R1

```
enable
configure terminal
hostname R1
banner motd # Authorized Access Only. Violators Will Be Prosecuted. #

! Enable password (encrypted)
enable secret Cisco2020

! Console line security
line console 0
 password Console2020
 login
 logging synchronous
 exec-timeout 5 0
 exit

! VTY lines — SSH only
line vty 0 4
 login local
 transport input ssh
 exit

! Encrypt plaintext passwords
service password-encryption

! Local admin account
username admin privilege 15 secret Admin2020

! Interface IP
interface GigabitEthernet0/0
 ip address 192.168.1.1 255.255.255.0
 description ## Link to SW1 ##
 no shutdown
 exit

! SSH setup
ip domain-name lab.local
crypto key generate rsa general-keys modulus 2048
ip ssh version 2
ip ssh time-out 60
ip ssh authentication-retries 3

end
wr
```

### Switch SW1

```
enable
configure terminal
hostname SW1

! Management VLAN IP
interface vlan 1
 ip address 192.168.1.2 255.255.255.0
 no shutdown
 exit

ip default-gateway 192.168.1.1

enable secret Cisco2020

line console 0
 password Console2020
 login
 logging synchronous
 exit

line vty 0 15
 login local
 transport input ssh
 exit

username admin privilege 15 secret Admin2020
service password-encryption

ip domain-name lab.local
crypto key generate rsa general-keys modulus 2048
ip ssh version 2

end
wr
```

---

## Verification

### Connectivity Tests — from PC 1

```
C:\>ping 192.168.1.1
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Reply from 192.168.1.1: bytes=32 time<1ms TTL=255
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)

C:\>ping 192.168.1.2
Reply from 192.168.1.2: bytes=32 time<1ms TTL=255
Packets: Sent = 4, Received = 3, Lost = 1 (25% loss)
[Note: First packet timeout is normal — ARP resolution delay]

C:\>ping 192.168.1.11
Reply from 192.168.1.11: bytes=32 time=7ms TTL=128
Reply from 192.168.1.11: bytes=32 time<1ms TTL=128
Reply from 192.168.1.11: bytes=32 time<1ms TTL=128
Reply from 192.168.1.11: bytes=32 time<1ms TTL=128
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

### Key Show Commands — on R1

```
R1# show ip interface brief
Interface         IP-Address      OK? Method Status   Protocol
GigabitEthernet0/0 192.168.1.1   YES manual up       up

R1# show ip route
C   192.168.1.0/24 is directly connected, GigabitEthernet0/0

R1# show version
Cisco IOS Software, Version 15.4 ...
Uptime: X hours Y minutes

R1# show running-config
[full configuration output]

R1# show arp
Protocol Address     Age   Hardware Addr    Type  Interface
Internet 192.168.1.10  -   0060.9A09.636D  ARPA  GigabitEthernet0/0
Internet 192.168.1.11  -   0001.C7FF.3011  ARPA  GigabitEthernet0/0
```

---

## Troubleshooting Notes

**Issue encountered:** `logging synchronous` was initially typed incorrectly as `logging synchronus` — IOS returned `% Invalid input detected at '^' marker.`

**Fix:** Retyped the command correctly. Important lesson — IOS will not accept partial or misspelled commands even when they closely resemble valid commands. Always check spelling and use `?` for tab completion guidance.

**Note:** When first pinging SW1 (192.168.1.2), 1 out of 4 packets was lost (25% loss). This is expected behaviour — the first packet triggers ARP resolution which causes a timeout. Subsequent pings succeed.

---

## Key Takeaways

- IOS has five core modes: User EXEC → Privileged EXEC → Global Config → Interface/Line Config. `Ctrl+Z` always returns to Privileged EXEC.
- `enable secret` uses MD5 encryption and always overrides `enable password` — always use `secret`.
- `service password-encryption` encrypts all plaintext passwords in the running-config (Type 7 — weak, but better than plaintext).
- SSH requires: domain name, RSA key generation (2048-bit minimum), `ip ssh version 2`, and local user account with `login local` on VTY lines.
- `copy running-config startup-config` (or `wr`) saves config to NVRAM — without this, a reload wipes all changes.
- The first ping to a new host often fails due to ARP resolution — this is normal and expected behaviour.
- `show ip interface brief` is always the first command to run when troubleshooting — it shows interface status at a glance.

---

## Lab Validation Checklist

- [x] R1 hostname set to R1
- [x] MOTD banner configured with legal warning
- [x] Enable secret encrypted password configured on R1 and SW1
- [x] Console line password and `logging synchronous` configured
- [x] VTY lines restricted to SSH only (`transport input ssh`)
- [x] `service password-encryption` applied to both devices
- [x] R1 G0/0 IP address 192.168.1.1/24 configured and up/up
- [x] SW1 VLAN 1 IP address 192.168.1.2/24 configured
- [x] SW1 default gateway set to 192.168.1.1
- [x] PC 1 IP: 192.168.1.10/24, Gateway: 192.168.1.1
- [x] PC 2 IP: 192.168.1.11/24, Gateway: 192.168.1.1
- [x] SSH configured on both R1 and SW1 (RSA 2048-bit, version 2)
- [x] PC 1 can ping R1 (192.168.1.1) — 0% loss
- [x] PC 1 can ping SW1 (192.168.1.2) — ARP delay on first packet only
- [x] PC 1 can ping PC 2 (192.168.1.11) — 0% loss
- [x] All IOS modes navigated successfully
- [x] All essential show commands executed and understood

---

*Lab 01 Complete — Foundation skills for all subsequent CCNA labs.*
