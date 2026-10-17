# 04 Remote Access VPN (OpenVPN)

## Goal

Give a remote user encrypted access to **one resource only**, the DMZ web server, and prove that the Admin LAN and firewall management stay unreachable.

## Design

| Item | Choice | Reason |
|---|---|---|
| Server mode | Local User Access | Users and certificates managed in pfSense |
| Authentication | Certificate **and** username/password | Two factors: something you have + something you know. A stolen client file alone is not enough |
| Protocol / port | UDP 1194 on WAN | OpenVPN default |
| Encryption / digest | AES-256-GCM / SHA256 | Modern, strong defaults |
| TLS authentication | Enabled | Drops unauthenticated packets early |
| Tunnel network | 10.0.8.0/24 | Separate from every zone, so rules can identify VPN clients clearly |
| Tunnel type | **Split tunnel** (Redirect Gateway off) | Only lab traffic uses the VPN |
| Pushed route | 192.168.30.0/24 only | Clients are told about the DMZ and nothing else |

## Certificate design

| Object | Name | Details |
|---|---|---|
| Certificate Authority | `LabVPN-CA` | Internal CA, RSA 2048, SHA256, lifetime 3650 days, CN `labvpn-ca` |
| Server certificate | `vpn-server-cert` | Signed by LabVPN-CA, type Server Certificate, CN `pfsense-vpn-server` |
| User | `vpnuser1` | Strong password, user certificate `vpnuser1-cert` signed by LabVPN-CA |

Client export used the **openvpn-client-export** package (Inline Configuration → Most Clients), with host name resolution set to the pfSense WAN IP.

## Firewall rules on the OpenVPN tab

The wizard's "OpenVPN rule" (allow everything) was **unticked**; stricter rules were written by hand. Only the "Firewall Rule" option (opens UDP 1194 on WAN) was kept.

| # | Action | Proto | Source | Destination | Port | Description |
|---|---|---|---|---|---|---|
| 1 | PASS | TCP | 10.0.8.0/24 | WEB_SERVER | WEB_ADMIN_PORTS (22, 80, 443) | VPN: web server 22/80/443 |
| 2 | PASS | ICMP (echo) | 10.0.8.0/24 | WEB_SERVER | n/a | VPN: ping web server (diagnostics) |
| 3 | BLOCK (log) | Any | 10.0.8.0/24 | any | any | VPN: explicit deny for evidence |

Rule 3 is technically redundant because of the default deny, but it is logged, which creates evidence in the firewall log.

## Connecting

On External-Client:

```
sudo openvpn --config ~/pfsense-udp-1194-vpnuser1-config.ovpn
```

Enter the VPN username and password. Success shows `Initialization Sequence Completed`, and `ip a show tun0` shows a `10.0.8.x` address.

> The `.ovpn` file contains private key material. It is **not** stored in this repository (blocked by `.gitignore`).

## Proving the restrictions work

| Test (External-Client, VPN up) | Expected | What it proves |
|---|---|---|
| `curl -sI -m5 http://192.168.30.10` | 200 OK | Authorised resource reachable |
| `nc -zv -w3 192.168.30.10 22` | succeeds | SSH allowed to the web server |
| `sudo ip route add 192.168.10.0/24 dev tun0` then `nc -zv -w3 192.168.10.50 22` | timeout | The **firewall** (not merely a missing route) blocks the Admin LAN |
| `curl -sk -m5 https://192.168.10.1` | timeout | VPN users cannot reach firewall management |

The manual route test matters: without it, a failing connection could be explained by "there is no route". Adding the route by hand forces the traffic into the tunnel so only the firewall rules can stop it. The test route is removed afterwards (`sudo ip route del 192.168.10.0/24`).

Verification points: **Status → OpenVPN** (connected client) and **Status → System Logs → Firewall** (blocks from `10.0.8.x`).

## If the VPN will not connect

1. WAN IP in the `.ovpn` `remote` line is correct
2. "Block private networks" is off on WAN (lab only)
3. A WAN rule for UDP 1194 exists
4. Time and date roughly match on both VMs (certificates are time-sensitive)
5. **Status → System Logs → OpenVPN** for the exact error

## Evidence

`screenshots/09-openvpn-server-rules.png`, `screenshots/10-status-openvpn-connected.png`, `screenshots/11-vpn-allowed-blocked-tests.png`
