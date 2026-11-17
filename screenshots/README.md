# Screenshot evidence index

All images are original captures from the lab. They are indexed here so a reviewer can match each claim in the [README](../README.md) to its evidence.

| Screenshot | What it proves |
|:--|:--|
| [wazuh-dashboard.png](wazuh-dashboard.png) | Custom SOC overview: failed Windows logons, Windows account changes (4720/4722/4724/4732/4738), Linux SSH auth from `alpine-lab` |
| [wazuh-discover-events.png](wazuh-discover-events.png) | Raw `wazuh-alerts-*` events, ~3,962 hits over 7 days; Sysmon fields such as image loads, registry targets, hashes and signatures |
| [windows-agents-connected.png](windows-agents-connected.png) | Agent inventory: 2 active, 0 disconnected, 0 pending; `soc-windows` v4.14.10 and `alpine-lab` |
| [fim-alerts-overview.png](fim-alerts-overview.png) | FIM overview: alerts by action (added/deleted/modified), rule distribution, top agent |
| [fim-alert-events.png](fim-alert-events.png) | 8 `syscheck` events with path, event type, rule description, level and ID |
| [system-inventory-hygiene.png](system-inventory-hygiene.png) | OS families, installed packages, running processes (including 3 `nginx` processes), CPUs and destination ports |
| [linux-ssh-auth.png](linux-ssh-auth.png) | "Failed password" events from `alpine-lab` in `wazuh-archives-*`; source IP/user not extracted |
| [mitre-attack-framework.png](mitre-attack-framework.png) | MITRE tactics and techniques mapped per agent |
| [threat-hunting-dashboard.png](threat-hunting-dashboard.png) | Alert-level evolution, level-12+ alerts, authentication success/failure, top MITRE techniques |
| [vulnerability-detection.png](vulnerability-detection.png) | CVEs by severity and affected packages on `soc-windows` |
| [alpine-juice-shop.png](alpine-juice-shop.png) | OWASP Juice Shop reachable over HTTP through Nginx on the Alpine host |
| [vmware-lab-environment.png](vmware-lab-environment.png) | VMware Workstation VM layout (Ubuntu, Windows 10, Linux) |

## Evidence still to capture

The following screenshots were planned but no matching capture was available, so they are intentionally absent rather than substituted:

- **SCA results page** — policy name/version with total, passed, failed and not-applicable checks. The agent log confirms the CIS Win10 policy loaded and the scan finished in 28 seconds, but no score screenshot exists.
- **Nginx log ingestion into Wazuh** — a request-to-alert sequence proving `/var/log/nginx/access.log` is a working Wazuh source. Logs were generated locally, but end-to-end alert evidence was not captured.
- **Alpine agent connection log** — a close-up of the Alpine agent reporting a successful connection to `192.168.188.132:1514/tcp` (the agent inventory does show it as active).

Add these before using the repository to claim those specific achievements in an application.
