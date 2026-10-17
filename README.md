# pfSense Network Security Home Lab

A virtual small-office network built with **pfSense CE** and **VirtualBox** to practise firewall administration, network segmentation, secure remote access, security monitoring and troubleshooting. Everything runs on one computer inside isolated virtual networks; nothing is exposed to the internet.

**Author:** Nayan Marathothu Muraleedharan

## Network topology

<img width="3200" height="2700" alt="topology" src="https://github.com/user-attachments/assets/f486504a-d00e-44d4-a508-25c408ad8892" />
<br>
## Highlights

- Three isolated zones: **Admin LAN, Guest and DMZ** (`192.168.10/20/30.0/24`), each on its own VirtualBox internal network
- **Least-privilege firewall policy** written as a matrix first, then implemented with aliases and logged block rules
- Linux **nginx web server in the DMZ**, published through a NAT port forward (WAN 8080 → DMZ 80)
- **Certificate-based OpenVPN** (certificate + username/password), split tunnel, limited to one DMZ web server
- **Suricata IDS** in alert-only mode, used to detect a simulated port scan and correlate it with firewall logs
- **20-case test plan**, three injected-fault troubleshooting reports, and a tested backup/restore

## Technologies

pfSense CE · VirtualBox 7 · OpenVPN · Suricata (ET Open rules) · Ubuntu Server · Xubuntu · nginx · DHCP · DNS (Unbound) · NAT · TCP/IP

## Security policy summary

| Source ↓ / Destination → | Admin LAN | Guest | DMZ web server | Firewall management | Internet |
|---|---|---|---|---|---|
| **Admin LAN** | own zone | allowed | 22, 80, 443 | HTTPS 443 | allowed |
| **Guest** | BLOCKED | own zone | port 80 only | DNS (53) only | allowed |
| **DMZ** | BLOCKED | BLOCKED | own zone | DNS (53) only | 80/443 (updates) |
| **VPN client** | BLOCKED | BLOCKED | 22, 80, 443 | BLOCKED | BLOCKED (split tunnel) |
| **Internet / WAN** | BLOCKED | BLOCKED | only via port forward 8080 | BLOCKED | n/a |

Replies to allowed connections are allowed automatically (stateful firewall). Anything not explicitly allowed is denied by default.

## Documentation

| Doc | What it covers |
|---|---|
| [01 Architecture](docs/01-architecture.md) | Design goals, zones, virtual networking, VM list |
| [02 IP plan](docs/02-ip-plan.md) | Subnets, DHCP pools, reservations, DNS |
| [03 Firewall policy](docs/03-firewall-policy.md) | Policy matrix, aliases, every rule and why it exists |
| [04 VPN](docs/04-vpn.md) | CA and certificate design, tunnel settings, restrictions, tests |
| [05 Monitoring](docs/05-monitoring.md) | Log reading, Suricata setup, incident note |
| [06 Test results](docs/06-test-results.md) | 20-case test table with expected and actual results |
| [07 Troubleshooting](docs/07-troubleshooting.md) | Troubleshooting method and three fault reports |
| [08 Limitations and next steps](docs/08-limitations-and-next-steps.md) | What this lab does and does not prove |

## Build summary

| Phase | Work | Evidence |
|---|---|---|
| 1 | VirtualBox networks, pfSense VM, client VMs | VM list, adapter settings |
| 2 | Interface assignment, setup wizard | Dashboard, interface assignments |
| 3 | DHCP, DNS, IP plan | Static mapping, host override, `nslookup` output |
| 4 | Guest/DMZ interfaces, aliases, rules | Rule tables, block log entries |
| 5 | nginx in DMZ, port forward | `curl` from External-Client |
| 6 | OpenVPN, restricted rules | Connection status, allowed/blocked tests |
| 7 | Suricata, port-scan investigation | Alert, firewall log, incident note |
| 8 | Backup, 3 injected faults, restore | Reports, restore test |
| 9 | Documentation | This repository |

## What I learned

<!-- Replace with 3-5 honest sentences in your own words after finishing the lab. Examples of what to cover:
- the most useful thing you learned (e.g. first-match-wins rule order)
- one problem that took you longest to diagnose and what it taught you
- what surprised you (e.g. "ping fails" proving nothing, the RFC1918 WAN block) -->

## Limitations and next steps

Virtual lab only (not production hardware); single firewall with no high availability; self-signed certificates; IDS in alert-only mode; no centralised SIEM; VLANs not used. See [docs/08](docs/08-limitations-and-next-steps.md).

Next: VLANs on a managed switch, central SIEM (Wazuh or Security Onion), automated config backups, Active Directory integration.

## Safety note

No passwords, private keys, VPN profiles or unredacted configuration backups are included. All addresses are private lab addresses. The lab is isolated from any real network.
