# 08 Limitations and Next Steps

A lab proves some things and not others. Being clear about the difference matters.

## What this lab demonstrates

- Designing a multi-zone network with a written security policy
- Implementing that policy with firewall rules, aliases, NAT and DHCP/DNS
- Building certificate-based remote access with least privilege
- Detecting and documenting a simulated scan using two log sources
- Diagnosing faults methodically, and backing up and restoring a firewall

## Limitations

| Limitation | Why it matters |
|---|---|
| Runs in a virtual environment, not on production hardware | No real traffic volume, hardware faults or physical switching |
| Single firewall, no high availability | A real office would want failover |
| Self-signed certificates from an internal CA | Not a public or enterprise PKI |
| IDS in alert-only mode | Detects but does not block; tuning and false-positive handling were not exercised at scale |
| No centralised SIEM | Logs were analysed by hand; at most a single syslog target |
| VLANs not used | Separate virtual networks replaced them; real switches and 802.1Q were not exercised |
| "Block private networks" disabled on WAN | Needed for the virtual WAN; would stay enabled on a real internet link |
| One simulated attacker, one scan type | Does not represent realistic threat activity |

## Next steps

1. **VLANs** on a managed switch, replacing separate virtual networks
2. **Central SIEM** (Wazuh or Security Onion) receiving pfSense and Suricata logs
3. **Active Directory lab** (users, groups, Group Policy, domain DNS) integrated with the network
4. **Automated configuration backups** with versioning
5. **High availability** with a second firewall (CARP)
6. Suricata tuning and an optional move from IDS to IPS once alerts are understood
