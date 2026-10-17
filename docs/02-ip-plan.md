# 02 IP Plan, DHCP and DNS

## Addressing plan

| Zone | Subnet | Gateway / DNS | DHCP pool | Static devices |
|---|---|---|---|---|
| WAN | 10.0.2.0/24 (VirtualBox) | VirtualBox | DHCP client | pfSense WAN: `10.0.2.__` *(fill in)* |
| Admin LAN | 192.168.10.0/24 | 192.168.10.1 | .100 – .199 | Admin-WS `.50` (reservation) |
| Guest | 192.168.20.0/24 | 192.168.20.1 | .100 – .199 | none |
| DMZ | 192.168.30.0/24 | 192.168.30.1 | .100 – .199 | DMZ-Web01 `.10` (reservation) |
| VPN tunnel | 10.0.8.0/24 | 10.0.8.1 | assigned by OpenVPN | n/a |

`/24` means the first 24 bits are the network part: 254 usable hosts (mask 255.255.255.0). Addresses `.1–.99` sit outside each DHCP pool and are kept for fixed devices such as gateways and servers.

## DHCP

DHCP is enabled on LAN, GUEST and DMZ (Services → DHCP Server). pfSense creates hidden rules allowing DHCP on interfaces where the DHCP server runs, so no explicit rule is needed.

### Static mappings (reservations)

| Host | Zone | Reserved IP | Reason |
|---|---|---|---|
| admin-ws | Admin LAN | 192.168.10.50 | Stable address for testing and rules |
| web01 | DMZ | 192.168.30.10 | Firewall rules, NAT and the `WEB_SERVER` alias depend on this address |

A reservation tells the DHCP server to always give a specific MAC address the same IP, which keeps firewall rules reliable.

## DNS

pfSense runs the **DNS Resolver (Unbound)** on all interfaces. Upstream servers: `1.1.1.1` and `9.9.9.9`. Domain: `lab.internal` (`.internal` is reserved for private use; `.local` was avoided because it clashes with mDNS).

| Host override | Domain | IP | Purpose |
|---|---|---|---|
| www | lab.internal | 192.168.30.10 | DMZ web server |

## NAT

Outbound NAT is in **Automatic** mode: pfSense rewrites the source address of outgoing traffic from each internal subnet to its WAN address and tracks the connection so replies return to the right host (Firewall → NAT → Outbound).

## WAN setting specific to this lab

"Block RFC1918 Private Networks" is **unticked** on WAN. In VirtualBox, the simulated internet and External-Client both use private 10.0.2.x addresses, so the default block would silently drop the VPN and port-forward tests. On a real WAN this option should stay enabled. "Block bogon networks" stays ticked.

## Verification commands

| Command (on a client) | What it proves |
|---|---|
| `ip a` | Interface has an IP in the correct subnet |
| `ip route` | Default gateway is the zone's pfSense address |
| `resolvectl status` | DNS server from DHCP is pfSense |
| `ping -c 3 192.168.10.1` | Client reaches its gateway (layer 3) |
| `ping -c 3 1.1.1.1` | Routing and NAT to the internet work |
| `nslookup example.com 192.168.10.1` | pfSense DNS answers queries |
| `dig +short www.lab.internal` | Local host override works |

## Evidence

- `screenshots/04-dhcp-static-mappings.png`
- `screenshots/05-dns-resolver-host-override.png`
- `screenshots/14-terminal-ip-nslookup.png`
