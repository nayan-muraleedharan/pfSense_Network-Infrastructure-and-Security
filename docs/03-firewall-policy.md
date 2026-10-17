# 03 Firewall Policy

The policy was written **before** any rules were created. Every rule below comes from this matrix.

## Policy matrix

| Source ↓ / Destination → | Admin LAN | Guest | DMZ web server | Firewall management | Internet |
|---|---|---|---|---|---|
| Admin LAN | own zone | allowed | 22, 80, 443 | HTTPS 443 | allowed |
| Guest | BLOCKED | own zone | port 80 only | DNS (53) only | allowed |
| DMZ | BLOCKED | BLOCKED | own zone | DNS (53) only | 80/443 (updates) |
| VPN client | BLOCKED | BLOCKED | 22, 80, 443 | BLOCKED | BLOCKED (split tunnel) |
| Internet / WAN | BLOCKED | BLOCKED | only via port forward 8080 | BLOCKED | n/a |

## How pfSense evaluates rules

- Rules are attached to the **inbound** interface: a GUEST rule controls traffic coming from the Guest network into pfSense.
- Evaluated **top to bottom; the first match wins**. Specific rules go above general ones.
- Traffic matching no rule is **denied by default**.
- The firewall is **stateful**: replies to allowed connections are allowed automatically.
- **Block** silently drops packets; **Reject** returns an error. Block is used on untrusted sides.

## Aliases

An alias is a named list of IPs, networks or ports reused in rules, so rules stay readable and one change applies everywhere.

| Name | Type | Contents |
|---|---|---|
| `RFC1918` | Network(s) | 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16 |
| `WEB_SERVER` | Host(s) | 192.168.30.10 |
| `WEB_PORTS` | Port(s) | 80, 443 |
| `WEB_ADMIN_PORTS` | Port(s) | 22, 80, 443 |

## GUEST rules

| # | Action | Proto | Source | Destination | Port | Description |
|---|---|---|---|---|---|---|
| 1 | PASS | TCP/UDP | GUEST subnets | This Firewall | 53 | GUEST: allow DNS to pfSense |
| 2 | PASS | TCP | GUEST subnets | WEB_SERVER | 80 | GUEST: web server HTTP only |
| 3 | BLOCK (log) | Any | GUEST subnets | This Firewall | any | GUEST: block firewall management |
| 4 | BLOCK (log) | Any | GUEST subnets | RFC1918 | any | GUEST: block all internal networks |
| 5 | PASS | Any | GUEST subnets | any | any | GUEST: internet access |

**Reasoning.** Rules 1–2 are the only exceptions: guests need DNS and may view the public website. Rule 3 stops guests managing the firewall. Rule 4 stops them reaching Admin, DMZ and the WAN-side private range. Rule 5 effectively applies only to non-private (internet) destinations because rule 4 already caught everything private.

## DMZ rules

| # | Action | Proto | Source | Destination | Port | Description |
|---|---|---|---|---|---|---|
| 1 | PASS | TCP/UDP | DMZ subnets | This Firewall | 53 | DMZ: allow DNS to pfSense |
| 2 | BLOCK (log) | Any | DMZ subnets | This Firewall | any | DMZ: block firewall management |
| 3 | BLOCK (log) | Any | DMZ subnets | RFC1918 | any | DMZ: never initiate to internal nets |
| 4 | PASS | TCP | DMZ subnets | any | WEB_PORTS | DMZ: outbound web (apt updates) |

**Reasoning.** A server that outsiders can reach must never be able to start connections into internal networks, so a compromised server cannot pivot. Outbound web access is allowed only for package updates. Anything else (for example DMZ → internet on port 25) falls to the default deny.

## ADMIN (LAN) rules

pfSense's built-in **Default allow LAN to any** and **Anti-Lockout** rules are kept: Admin is the trusted zone. The anti-lockout rule guarantees the administrator can always reach the web interface and must never be deleted.

## WAN rules

| Action | Proto | Source | Destination | Port | Description |
|---|---|---|---|---|---|
| PASS | TCP | any | WAN address (NAT to WEB_SERVER:80) | 8080 | Auto-created by port forward "Publish DMZ web server" |
| PASS | UDP | any | WAN address | 1194 | Auto-created by the OpenVPN wizard |

No other inbound WAN ports are open, and the pfSense web interface is **not** exposed on WAN.

## Port forward (destination NAT)

| Setting | Value |
|---|---|
| Interface | WAN |
| Protocol | TCP |
| Destination | WAN address, port 8080 |
| Redirect target | `WEB_SERVER`, port 80 (HTTP) |
| Filter rule association | Add associated filter rule |

Only one port is opened, which is how organisations publish a server behind a single public IP.

## OpenVPN rules

See [04-vpn.md](04-vpn.md).

## Design note: why not "block a site by IP"?

Blocking a website by IP address is a good exercise for learning rule order, but it is not a reliable way to block sites: many sites share IP addresses through CDNs and change addresses often. Domain-level blocking belongs in DNS (optional exercise in [05-monitoring.md](05-monitoring.md)).

## Testing

Segmentation is verified with specific TCP-port tests (`nc -zv`, `curl`) rather than ping, because a failed ping proves nothing: ICMP can be blocked while other protocols stay open, and vice versa. Block entries are confirmed in **Status → System Logs → Firewall**. Results are in [06-test-results.md](06-test-results.md).

## Evidence

`screenshots/06-aliases.png`, `screenshots/07-guest-dmz-rules.png`, `screenshots/08-port-forward-wan-rules.png`
