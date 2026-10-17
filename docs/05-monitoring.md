# 05 Monitoring and Log Analysis

## Goal

Practise the core of SOC analyst work: reading logs, investigating an alert, correlating two independent sources, and writing it up.

## Reading firewall logs

**Status → System Logs → Firewall.** Each line is one decision:

```
time      action interface source               destination          proto/flags
10:42:11  block  GUEST     192.168.20.101:51234 192.168.10.1:443     TCP:S
10:42:15  pass   GUEST     192.168.20.101:44872 192.168.30.10:80     TCP:S
```

| Field | Meaning | Question an analyst asks |
|---|---|---|
| action / rule | Pass or block, and which rule did it (ⓘ icon) | Which rule fired? Is it the one I expected? |
| interface | Where the packet entered | Did this come from the untrusted side? |
| source : port | Sender; source port is random | Who is it? Is it a known device? |
| destination : port | Target service (443 HTTPS, 22 SSH) | Why is a guest probing the firewall's HTTPS? |
| flags (S = SYN) | Start of a TCP connection attempt | Many SYNs to many ports = a scan |

## IDS vs IPS and Suricata

An **IDS** watches traffic and raises alerts when it matches known attack patterns (signatures). An **IPS** also blocks. **Suricata** can do both; here it runs in **alert-only (IDS) mode** so it can never accidentally cut off lab access. **Emerging Threats (ET) Open** is the free rule set.

## Suricata setup

pfSense-FW RAM was raised to 3 GB (Suricata is memory hungry).

1. System → Package Manager → Available Packages → install **suricata**
2. Services → Suricata → Global Settings: enable **Install ETOpen Emerging Threats rules**, update interval 12 hours → Save
3. Updates tab → Update, wait for "success"
4. Interfaces → Add: interface **GUEST**, description "Guest IDS", enable **Send Alerts to System Log**, leave **Block Offenders unticked** → Save
5. GUEST Categories tab: enable `emerging-scan`, `emerging-attack_response`, `emerging-exploit`, `emerging-malware`, `emerging-policy`; tick **Resolve Flowbits** → Save
6. Start Suricata on GUEST (green tick = running)

Suricata only sees traffic on the interface it is attached to, so the scan must come from Guest.

## Generating and investigating an alert

Run from Guest-Client (legal here because it is my own lab):

```
nmap -sS -p 1-1000 192.168.30.10
curl -s http://testmynids.org/uid/index.html
```

The second command fetches a harmless page crafted to trigger a known IDS test signature.

1. **Services → Suricata → Alerts** (GUEST): look for scan-type alerts (names starting `ET SCAN`) and an "ATTACK_RESPONSE id check" style alert. Exact names vary with rule version.
2. **Status → System Logs → Firewall**: find the matching block entries for the same time and source IP.
3. Correlate the two sources and write the incident note below.

```
Guest-Client (nmap) ──► GUEST interface ──► Suricata: IDS alert
                                        └─► Firewall: BLOCK log
                                                 └─► Analyst correlates both
```

## Incident note 01: Port scan from Guest network

> **Fill this in from your real run.** The values below show the format.

| Field | Entry |
|---|---|
| When | `YYYY-MM-DD HH:MM–HH:MM` (lab local time) |
| Who | Source `192.168.20.___` (Guest-Client) → Target `192.168.30.10` (web server) |
| What | TCP SYN scan across ports 1–1000; Suricata raised `ET SCAN` alerts: *(list actual alert names)* |
| Result | Firewall rule "GUEST: web server HTTP only" allowed port 80; all other ports blocked *(confirm from logs)* |
| Action | No compromise. Verified blocks in the firewall log. Recommend keeping guest IDS enabled. |

## Optional: DNS-based domain blocking

A better replacement for "block a site by IP", and it exposes a limitation worth discussing.

1. Services → DNS Resolver → Custom options:
   ```
   server:
   local-zone: "example.org" always_nxdomain
   ```
   `nslookup example.org` from Guest-Client now returns NXDOMAIN.
2. **Bypass:** `nslookup example.org 8.8.8.8` still works because the client chose a different DNS server.
3. **Fix:** on the GUEST tab, below "allow DNS to firewall" and above the internet rule, add Block TCP/UDP, GUEST subnets → any, port 53, "GUEST: no external DNS". The bypass now times out.

## Optional: central syslog

Sending logs to another host means they survive a compromise of the device. pfSense logs were sent to DMZ-Web01.

`/etc/rsyslog.d/10-pfsense.conf` on DMZ-Web01:

```
module(load="imudp")
input(type="imudp" port="514")
template(name="RemoteLogs" type="string" string="/var/log/remote/%HOSTNAME%.log")
if $fromhost-ip == '192.168.30.1' then ?RemoteLogs
& stop
```

Then `sudo mkdir -p /var/log/remote && sudo systemctl restart rsyslog` (and `sudo ufw allow from 192.168.30.1 to any port 514 proto udp` if UFW is enabled). In pfSense: Status → System Logs → Settings → enable Remote Logging, server `192.168.30.10:514`. Verify with `sudo tail -f /var/log/remote/*.log`.

## If no alerts appear

Check that Suricata is running on **GUEST** (not WAN), that the rules were updated, that `emerging-scan` is enabled, and that the scan came from the Guest VM.

## Evidence

`screenshots/12-suricata-alert-and-firewall-log.png`; incident note above.
