# Configuration examples

Sanitized configuration files used in the lab. They are templates: replace IP addresses, hostnames and user paths with your own before reuse. No credentials, keys or enrollment tokens are included.

| File | What it shows |
|:--|:--|
| [windows-fim-config.xml](windows-fim-config.xml) | `<syscheck>` block for real-time FIM on the Windows Desktop and Startup folder |
| [nginx-config.conf](nginx-config.conf) | Nginx reverse-proxy server block in front of OWASP Juice Shop, with access/error logging |
| [syslog-config.txt](syslog-config.txt) | Wazuh `<localfile>` and manager `<remote>` syslog templates, plus Alpine OpenRC notes |

Full build documentation is in [`../documentation/`](../documentation/), and the FIM alert analysis is in [`../investigation/fim-investigation.md`](../investigation/fim-investigation.md).
