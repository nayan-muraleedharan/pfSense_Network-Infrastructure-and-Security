# 06 Test Results

All 20 tests were planned in advance. Fill in **Actual result** and **Result** honestly after running each test. Failures are kept and the retest is recorded: that is more convincing than a page of perfect passes.

Result key: ✅ Pass · ❌ Fail · 🔁 Fixed and retested · ⬜ Not yet run

| ID | Test | How to test | Expected | Actual result | Result |
|---|---|---|---|---|---|
| T01 | DHCP assignment | `ip a` on each client | Address in correct subnet | | ⬜ |
| T02 | DHCP reservation | Reboot DMZ-Web01 | Always 192.168.30.10 | | ⬜ |
| T03 | DNS resolution | `nslookup example.com` | Resolves via pfSense | | ⬜ |
| T04 | Local DNS name | `dig +short www.lab.internal` | 192.168.30.10 | | ⬜ |
| T05 | Routing and NAT | `curl -sI https://example.com` from Admin-WS | HTTP response | | ⬜ |
| T06 | Guest → Admin blocked | `nc -zv -w3 192.168.10.50 22` | Fails | | ⬜ |
| T07 | Guest → firewall GUI blocked | `nc -zv -w3 192.168.10.1 443` | Fails | | ⬜ |
| T08 | Guest → web server port 80 | `curl -sI http://192.168.30.10` | 200 OK | | ⬜ |
| T09 | Guest → web server SSH blocked | `nc -zv -w3 192.168.30.10 22` | Fails | | ⬜ |
| T10 | DMZ → Admin blocked | From web server: `nc -zv -w3 192.168.10.50 22` | Fails | | ⬜ |
| T11 | Admin → DMZ allowed | Browser to www.lab.internal; SSH | Works | | ⬜ |
| T12 | Rule logging | Run T06, check Status → System Logs → Firewall | Block entry visible | | ⬜ |
| T13 | Port forward | `curl -sI http://<WAN-IP>:8080` from External-Client | 200 OK | | ⬜ |
| T14 | No other WAN ports open | `nc -zv -w3 <WAN-IP> 22` (and 443) | Fails | | ⬜ |
| T15 | VPN authentication | Connect with certificate + password | `tun0` has 10.0.8.x | | ⬜ |
| T16 | VPN allowed resource | `curl -sI http://192.168.30.10` over VPN | 200 OK | | ⬜ |
| T17 | VPN blocked resources | Admin LAN and firewall GUI over VPN | Fail | | ⬜ |
| T18 | IDS detection | `nmap -sS` from Guest | Suricata alert | | ⬜ |
| T19 | Backup and restore | Restore baseline XML | Services work afterwards | | ⬜ |
| T20 | Fault diagnosis (3) | Phase 8 faults | Found, fixed, verified | | ⬜ |

## Summary

| Passed first time | Failed then fixed | Not run |
|---|---|---|
| _ / 20 | _ / 20 | _ / 20 |

## Notes on failures and retests

*(Record each failure here: test ID, what happened, cause, fix, retest result.)*

## Test method note

Restrictions were tested with specific TCP ports (`nc -zv`, `curl`), not ping, and each block was confirmed in the firewall log so I could be sure it was **my rule** that blocked the traffic and not a different problem.
