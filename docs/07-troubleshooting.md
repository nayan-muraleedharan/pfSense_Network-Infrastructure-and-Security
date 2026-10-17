# 07 Troubleshooting

## Method: the troubleshooting staircase

Work from the bottom up so you never guess. At each step ask "does this work?" and stop at the first "no". That step shows which layer holds the fault.

| Step | Layer | Check | Tool |
|---|---|---|---|
| 1 | Link / IP | Interface up, got an address? | `ip a` |
| 2 | Gateway | Reach the router? | `ping <gateway>` |
| 3 | Routing / NAT | Reach an outside IP? | `ping 1.1.1.1` |
| 4 | DNS | Do names resolve? | `nslookup`, `dig` |
| 5 | Port / service | Is the port open? | `nc -zv`, `curl` |
| 6 | Firewall logs | Which rule blocked it? | Status → System Logs → Firewall |

## Change-management loop

Baseline backup → Inject fault → Diagnose → Fix → Verify → Write report

## Faults injected

| # | Fault introduced | Expected symptoms | Diagnosis clue |
|---|---|---|---|
| 1 | DNS: Services → DNS Resolver → untick Enable → Save/Apply | Browsing fails on Admin-WS but `ping 1.1.1.1` works | Staircase passes steps 1–3, fails step 4; `nslookup` times out |
| 2 | Rule order: on GUEST, move the block-firewall rule above the allow-DNS rule | Guest has an IP and can ping out but no names resolve | Firewall log shows block from `192.168.20.x` to `192.168.20.1:53` |
| 3 | VPN: set OpenVPN Local Network to a wrong subnet, or edit the OpenVPN rule destination to a wrong IP | VPN connects but the web server is unreachable | `ip route` on the client lacks the DMZ route, or logs show OpenVPN blocks |

---

## Report 1: DNS service disabled

> **Complete from your real run.**

| Section | Entry |
|---|---|
| Symptom | |
| Impact | |
| Diagnosis | Staircase result per step; commands and output |
| Root cause | |
| Resolution | |
| Verification | |
| Prevention | |

## Report 2: Rule-order fault (GUEST)

> The entry below is the **worked example** from the lab plan, showing the expected format and findings. **Replace it with your own observations** (real IPs, timings, output) after you perform the fault.

| Section | Entry |
|---|---|
| Symptom | Guest-Client could ping 1.1.1.1 but `curl example.com` failed with "could not resolve host". |
| Impact | All guest users lose web browsing. |
| Diagnosis | Staircase: steps 1–3 passed, step 4 failed. `nslookup example.com 192.168.20.1` timed out. The firewall log showed blocks from `192.168.20.101` to `192.168.20.1:53` under rule "GUEST: block firewall management". |
| Root cause | The block rule sat above the "allow DNS" rule, so first-match-wins blocked DNS. |
| Resolution | Moved the DNS rule above the block rule and applied changes. |
| Verification | `nslookup` and `curl` succeed; management port 443 is still blocked. |
| Prevention | Keep rule descriptions clear, export a config backup before changes, test after each change. |

## Report 3: VPN route / rule fault

> **Complete from your real run.**

| Section | Entry |
|---|---|
| Symptom | |
| Impact | |
| Diagnosis | |
| Root cause | |
| Resolution | |
| Verification | |
| Prevention | |

---

## Backup, restore and lock-out recovery

### Configuration backup

Diagnostics → Backup & Restore → Backup Configuration, area **All**, **Encrypt this configuration file** with a password, download the XML. The file is stored **outside** the Git project folder (e.g. `C:\LabBackups\config-baseline.xml`) because it contains password hashes, private keys, VPN keys and certificates. A VirtualBox snapshot named `baseline-working` was also taken of every VM.

Snapshots protect the machine; configuration backups protect the configuration. Both are used.

### Restore test

A backup you have never restored is only a hope.

1. Make a deliberate bigger mistake (e.g. delete the GUEST rules)
2. Diagnostics → Backup & Restore → Restore Backup, choose the baseline XML, enter the password, Restore (pfSense reboots)
3. Re-run the Guest tests (T06–T09). They should pass again

Restore result: *(record here)*

### Locked out of the web interface?

Use the pfSense **VM console** menu:

| Option | Use |
|---|---|
| 2 Set interface IP address | Fix a wrong LAN address |
| 3 Reset webConfigurator password | Forgotten admin password |
| 8 Shell → `pfctl -d` | Temporarily switch the firewall **off** to regain access, fix the rule, then `pfctl -e` to switch it on |
| 11 Restart webConfigurator | GUI itself is hung |
| 15 Restore recent configuration | Roll back to an earlier automatic config revision |

Menu numbers can vary slightly by version; read the labels on screen.

## Evidence

`screenshots/13-backup-and-restore.png`, final snapshot list
