# Small Network Setup Guide

A practical guide to planning, configuring and troubleshooting a small office/home network: IP addressing, DHCP, Wi-Fi, basic security and a step-by-step troubleshooting process.

Built from hands-on experience in an IT support role and my CompTIA Network+ and A+ studies.

---

## Contents

1. [Context – Action – Result](#context--action--result)
2. [Network overview](#network-overview)
3. [Network diagram](#network-diagram)
4. [IP addressing scheme](#ip-addressing-scheme)
5. [DHCP configuration](#dhcp-configuration)
6. [Wi-Fi and security](#wi-fi-and-security)
7. [Setup steps](#setup-steps)
8. [Troubleshooting guide](#troubleshooting-guide)
9. [Skills demonstrated](#skills-demonstrated)

---

## Context – Action – Result

**Context**
A small office needs reliable internet access and internal communication for three desktop PCs, a shared network printer and Wi-Fi devices, and it needs to be easy to maintain.

**Action**
- Planned the network layout, device placement and cabling.
- Designed an IP addressing scheme that separates static devices from DHCP clients.
- Configured the router (DHCP, Wi-Fi, security) and connected wired devices through a switch.
- Tested connectivity between devices and to the internet.
- Documented a repeatable troubleshooting process for common network faults.

**Result**
- A clear, step-by-step guide someone else can follow to set up and maintain the network.
- Stronger practical networking skills in configuration, addressing and fault-finding.
- Experience explaining technical processes clearly, a key skill for IT support.

---

## Network overview

| Device | Quantity | Connection | Addressing |
|---|---|---|---|
| Router / internet gateway (with Wi-Fi) | 1 | ISP line | Static (gateway) |
| Unmanaged switch (8-port) | 1 | Router LAN port | N/A |
| Desktop PCs | 3 | Wired via switch | Static |
| Network printer | 1 | Wired via switch | Static |
| Laptops and phones | Varies | Wi-Fi | DHCP |

**Objectives**
- Every device can reach the internet and each other.
- Fixed addresses for devices that others rely on (PCs, printer).
- Automatic addressing for guests and mobile devices.
- A simple, documented way to troubleshoot problems.

---

## Network diagram

```
                 Internet
                    |
             [ ISP / Modem ]
                    |
        +-----------------------+
        |  Router 192.168.1.1   |   ))) Wi-Fi (WPA2/WPA3)
        |  DHCP: .100 - .200    |        |
        +-----------------------+   Laptops / Phones
                    |                (DHCP clients)
          +-------------------+
          |  8-port Switch    |
          +-------------------+
           |      |      |     |
         PC1    PC2    PC3   Printer
         .10    .11    .12    .50
```

---

## IP addressing scheme

Network: **192.168.1.0/24** (subnet mask 255.255.255.0, 254 usable addresses)

| Device | IP address | Subnet mask | Default gateway | DNS |
|---|---|---|---|---|
| Router (LAN) | 192.168.1.1 | 255.255.255.0 | N/A | ISP / 1.1.1.1 |
| PC1 | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 | 192.168.1.1 |
| PC2 | 192.168.1.11 | 255.255.255.0 | 192.168.1.1 | 192.168.1.1 |
| PC3 | 192.168.1.12 | 255.255.255.0 | 192.168.1.1 | 192.168.1.1 |
| Network printer | 192.168.1.50 | 255.255.255.0 | 192.168.1.1 | 192.168.1.1 |

**Address plan**

| Range | Use |
|---|---|
| .1 | Router / gateway |
| .2 – .49 | Static: PCs and servers |
| .50 – .99 | Static: printers and network devices |
| .100 – .200 | DHCP pool: laptops, phones, guests |
| .201 – .254 | Reserved for future use |

Keeping static addresses **outside** the DHCP pool prevents IP conflicts.

---

## DHCP configuration

Configured on the router:

| Setting | Value |
|---|---|
| DHCP server | Enabled |
| Start address | 192.168.1.100 |
| End address | 192.168.1.200 |
| Lease time | 24 hours |
| Default gateway | 192.168.1.1 |
| DNS servers | 192.168.1.1 (router forwards to ISP / 1.1.1.1) |

**Alternative:** instead of setting static IPs on each PC, use **DHCP reservations** on the router (bind each PC's MAC address to a fixed IP). This keeps all addressing managed in one place.

---

## Wi-Fi and security

- **Encryption:** WPA3 (or WPA2-AES if older devices need it). Never WEP or open networks.
- **Strong passphrase:** at least 12 characters.
- **Change default router admin credentials** and disable remote admin access.
- **Disable WPS**, which is a known weak point.
- **Guest network:** separate SSID for visitors, isolated from office devices.
- **Firmware:** keep the router updated.
- **Physical security:** keep the router and switch in a locked or low-traffic area.

---

## Setup steps

1. **Connect the router** to the ISP modem/line and power it on.
2. **Log in to the router** (e.g. `http://192.168.1.1`) and change the default admin password.
3. **Set the LAN IP** to 192.168.1.1 / 255.255.255.0.
4. **Configure DHCP** with the pool 192.168.1.100 – .200.
5. **Configure Wi-Fi:** SSID, WPA3/WPA2-AES, strong passphrase, and disable WPS.
6. **Connect the switch** to a router LAN port, then connect the PCs and printer to the switch.
7. **Assign static IPs** to PC1–PC3 and the printer (or create DHCP reservations).
8. **Test** each device (see below) and record the results.

**Verification tests**

```cmd
ipconfig /all            :: confirm IP, mask, gateway and DNS
ping 192.168.1.1         :: can reach the router
ping 192.168.1.50        :: can reach the printer
ping 1.1.1.1             :: can reach the internet by IP
nslookup google.com      :: DNS is resolving names
```

---

## Troubleshooting guide

I follow the **CompTIA troubleshooting methodology**: identify the problem → establish a theory → test the theory → plan and implement a fix → verify → document.

### Work from the bottom up

| Step | Check | Command / action | If it fails |
|---|---|---|---|
| 1 | Physical | Cables seated? Link lights on? | Reseat or swap cable, try another switch port |
| 2 | IP config | `ipconfig /all` | See the "Common faults" table below |
| 3 | Local stack | `ping 127.0.0.1` | Reset the TCP/IP stack: `netsh int ip reset` |
| 4 | Gateway | `ping 192.168.1.1` | Check cabling, switch and router |
| 5 | Internet (IP) | `ping 1.1.1.1` | Check the router WAN connection or ISP |
| 6 | DNS | `nslookup google.com` | Check DNS settings, try `ipconfig /flushdns` |

### Common faults

| Symptom | Likely cause | Fix |
|---|---|---|
| IP address is **169.254.x.x** (APIPA) | Device can't reach the DHCP server | Check the cable or Wi-Fi, then `ipconfig /release` and `ipconfig /renew` |
| "IP address conflict" warning | Static IP inside the DHCP pool, or a duplicate | Move statics outside .100–.200 or use reservations |
| Internet works by IP but not by name | DNS issue | Set correct DNS, run `ipconfig /flushdns` |
| Can't print | Printer IP changed or offline | Give the printer a static IP or reservation, then re-add it on PCs |
| Slow or dropping Wi-Fi | Interference or poor signal | Change channel, move the router, prefer 5 GHz |
| One PC offline, others fine | That device's cable, port or config | Swap cable or port, check its IP settings |

---

## Skills demonstrated

- IPv4 addressing and subnetting (/24)
- DHCP configuration, scopes and reservations
- Router, switch and Wi-Fi setup
- Network security fundamentals
- Structured troubleshooting with command-line tools (`ipconfig`, `ping`, `nslookup`, `netsh`)
- Clear technical documentation

---

*Author: [nachtlos1](https://github.com/nachtlos1) · CompTIA A+ · CompTIA Network+*
