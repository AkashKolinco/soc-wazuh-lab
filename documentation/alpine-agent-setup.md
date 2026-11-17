# Alpine Linux Agent Setup

The Alpine VM served two roles: a monitored Linux endpoint and the host for Nginx + OWASP Juice Shop.

## 1. Endpoint details

| Item | Lab value |
|:--|:--|
| Agent name | `alpine-lab` (agent ID `002`) |
| Address | `192.168.188.130` |
| OS | Alpine Linux (dashboard showed `3.24.4`) |
| Agent version | v4.14.x — confirm the exact build in your dashboard |
| Manager | `192.168.188.132` |
| Workloads | Nginx reverse proxy, OWASP Juice Shop, syslog source |

The agent inventory confirmed `alpine-lab` as **active** alongside `soc-windows`. Evidence: [`../screenshots/windows-agents-connected.png`](../screenshots/windows-agents-connected.png).

## 2. Prepare the system

```bash
# Check the release and current address
cat /etc/alpine-release
ip addr

# Set a static address via the Alpine network configuration, then verify
ping -c 3 192.168.188.132
```

## 3. Fix APK repositories

During package installation, `apk` reported that packages could not be found because the repository configuration still pointed at the installation media:

```bash
cat /etc/apk/repositories
# Observed problem entry: /media/cdrom/apk
```

Replace the CD-ROM entry with reachable community/main repositories for your Alpine release, then refresh the indexes:

```bash
vi /etc/apk/repositories
apk update
apk add nginx docker
```

> Do not copy mirror URLs from this document blindly — use the repositories matching your exact Alpine version.

## 4. Install and register the Wazuh agent

Add the Wazuh repository and install `wazuh-agent`, then register the agent with the manager (enrollment via agent name or key):

```bash
apk add --no-cache curl
# Follow the current official Wazuh docs for the Alpine agent package/repo
apk add wazuh-agent

/var/ossec/bin/agent-auth -m 192.168.188.132 -A alpine-lab
```

## 5. Manage the service with OpenRC

Alpine uses OpenRC rather than systemd. Confirm the real init script name before starting or enabling anything:

```bash
ls -l /etc/init.d/
rc-service <wazuh-service-name> status
rc-service <wazuh-service-name> start
rc-update add <wazuh-service-name> default
```

Verify in the log that the agent connected to the manager, and confirm its status in **Agents → Endpoints**. Do not claim a successful connection without that evidence.

## 6. Log sources

Add `<localfile>` entries for system logs and Nginx logs so the agent forwards them to Wazuh:

```xml
<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/messages</location>
</localfile>

<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/nginx/access.log</location>
</localfile>

<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/nginx/error.log</location>
</localfile>
```

See [`../configurations/syslog-config.txt`](../configurations/syslog-config.txt).

In the lab, "Failed password" events for `alpine-lab` were found in the `wazuh-archives-*` index using the Linux SSH Auth view. Evidence: [`../screenshots/linux-ssh-auth.png`](../screenshots/linux-ssh-auth.png).

**Known gap:** `data.srcip` and `data.srcuser` were not extracted in that view. Decoder/field-extraction tuning is still needed for clean source attribution, and Nginx log ingestion into Wazuh alerts was not fully verified.

## 7. Web workload

Nginx and OWASP Juice Shop were deployed on this host and are documented separately in [nginx-juice-shop.md](nginx-juice-shop.md). The system inventory view confirmed three running `nginx` processes on the Alpine endpoint. Evidence: [`../screenshots/system-inventory-hygiene.png`](../screenshots/system-inventory-hygiene.png).

## 8. Verification checklist

- `alpine-lab` visible as **active** in the Wazuh agent inventory.
- Agent log reports a connection to `192.168.188.132:1514/tcp`.
- `nginx -t` succeeds and Juice Shop is reachable through the proxy.
- SSH failures appear in Wazuh with parsed source IP/user (tuning pending).
- Nginx access/error events appear as Wazuh alerts (validation pending).
