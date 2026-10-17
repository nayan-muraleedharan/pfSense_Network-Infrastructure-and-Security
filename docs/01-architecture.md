# 01 Architecture

## Goal

Simulate a small office behind one firewall and show that I can design, build, test, monitor and recover it. One pfSense firewall sits between a simulated internet and three zones. Each zone is a separate virtual network, so traffic between zones must pass through pfSense, where firewall rules decide what is allowed.

## Why separate zones?

**Segmentation** splits a network into zones so a problem in one cannot spread to the others.

- **Admin LAN** is trusted. It holds the administrator workstation that manages the firewall.
- **Guest** is untrusted. Guest devices must not reach admin systems or firewall management.
- **DMZ** (demilitarised zone) holds servers that less-trusted users must reach. If the web server were compromised, the attacker would still be confined to the DMZ and could not move to the Admin LAN.

The design follows the **principle of least privilege**: every device and rule gets only the access it needs. For example, the VPN user can reach one web server, not the whole network.

## Topology

![Network topology](../diagrams/topology.png)

*(If the image is not yet added, see the Mermaid diagram in the [README](../README.md).)*

## Virtual networking

| Network type | Used for | Notes |
|---|---|---|
| NAT Network `LabWAN` (10.0.2.0/24) | pfSense WAN and External-Client | Simulated internet that VMs use to reach the real internet |
| Internal Network `lan-admin` | Admin zone | Private virtual switch, invisible to the host |
| Internal Network `lan-guest` | Guest zone | As above |
| Internal Network `lan-dmz` | DMZ zone | As above |

Internal Networks were chosen over Host-Only adapters because Host-Only connects the host PC to the lab network, which weakens isolation. The Admin-WS VM handles management instead.

### pfSense adapter mapping

| VirtualBox adapter | pfSense interface | Role | VirtualBox network |
|---|---|---|---|
| Adapter 1 | em0 | WAN | NAT Network `LabWAN` |
| Adapter 2 | em1 | LAN (Admin) | Internal `lan-admin` |
| Adapter 3 | em2 | OPT1 (Guest) | Internal `lan-guest` |
| Adapter 4 | em3 | OPT2 (DMZ) | Internal `lan-dmz` |

All adapters use the Intel PRO/1000 MT Desktop type. VLANs were deliberately not used; separate virtual networks achieve the same isolation with fewer moving parts.

## Virtual machines

| VM | OS | RAM / CPU | Disk | Network |
|---|---|---|---|---|
| pfSense-FW | pfSense CE | 2 GB (3 GB with Suricata) / 2 CPU | 20 GB | 4 adapters (above) |
| Admin-WS | Xubuntu 24.04 Desktop | 2 GB / 2 CPU | 20 GB | `lan-admin` |
| Guest-Client | Ubuntu Server 24.04 | 1 GB / 1 CPU | 10 GB | `lan-guest` |
| DMZ-Web01 | Ubuntu Server 24.04 | 1 GB / 1 CPU | 10 GB | `lan-dmz` |
| External-Client | Ubuntu Server 24.04 | 1 GB / 1 CPU | 10 GB | `LabWAN` |

Tools installed on the clients while temporarily attached to LabWAN (before isolation):

| VM | Packages |
|---|---|
| Guest-Client | `nmap curl dnsutils netcat-openbsd` |
| DMZ-Web01 | `nginx curl dnsutils` |
| External-Client | `openvpn curl dnsutils netcat-openbsd` |

## Isolation measures

- No lab adapter is bridged to a real network.
- A VirtualBox snapshot (`clean-install`, later `baseline-working`) was taken of each VM.
- The port forward and VPN exist only on the virtual WAN. Nothing is forwarded from a real router.

## Evidence

- `screenshots/01-vm-list.png`: VirtualBox VM list and pfSense adapter settings
- `screenshots/02-dashboard.png`, `screenshots/03-interface-assignments.png`
