# Nginx and OWASP Juice Shop

## 1. Purpose

OWASP Juice Shop is an intentionally vulnerable web application. In this lab it provided realistic HTTP traffic and web logs without exposing a vulnerable service to the internet. Nginx sat in front of it as a reverse proxy and generated access and error logs for potential forwarding to Wazuh.

## 2. Deployment

The web stack ran on the Alpine Linux VM:

| Item | Lab value |
|:--|:--|
| Host | `alpine-lab`, `192.168.188.130` |
| Web server | Nginx (OpenRC managed) |
| Application | OWASP Juice Shop, reached through the proxy |
| Config directory | `/etc/nginx/http.d/` |
| Config file | `default.conf` |
| Logs | `/var/log/nginx/access.log`, `/var/log/nginx/error.log` |

Juice Shop was reachable in a browser over plain HTTP on the lab network. The browser showed the "Not secure" warning expected for HTTP, confirming the service was live. Evidence: [`../screenshots/alpine-juice-shop.png`](../screenshots/alpine-juice-shop.png).

## 3. Configuration

The sanitized server block is in [`../configurations/nginx-config.conf`](../configurations/nginx-config.conf). Key points:

```nginx
server {
    listen 80;
    server_name _;

    access_log /var/log/nginx/access.log;
    error_log  /var/log/nginx/error.log;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Confirm the real Juice Shop port on your host (`ss -tlnp` / `netstat -tlnp`) and adjust `proxy_pass` accordingly.

## 4. Validation and troubleshooting

```bash
# Syntax check before any reload
nginx -t

# Reload if the test passes
nginx -s reload

# Inspect generated logs
tail -n 50 /var/log/nginx/access.log
tail -n 50 /var/log/nginx/error.log
```

**Encountered error:** `nginx: [emerg] invalid url prefix`.

Cause: the `proxy_pass` directive was missing a URL scheme:

```nginx
proxy_pass 127.0.0.1:3000;          # invalid
proxy_pass http://127.0.0.1:3000;   # correct
```

Workflow: run `nginx -t`, read the exact file and line from the error, correct the directive, retest, then reload.

The IT Hygiene dashboard confirmed three running `nginx` processes on the Alpine endpoint. Evidence: [`../screenshots/system-inventory-hygiene.png`](../screenshots/system-inventory-hygiene.png).

## 5. Log forwarding status

Nginx access and error logs were generated locally. Adding them as Wazuh sources requires `<localfile>` entries on the Alpine agent — see [`../configurations/syslog-config.txt`](../configurations/syslog-config.txt).

Status in this lab:

- Log files generated on Alpine: **confirmed**.
- Wazuh alerts generated from Nginx log events: **not fully verified**. Capture a request → alert sequence in Wazuh before claiming complete ingestion.

## 6. Safety

- Keep Juice Shop on an isolated lab network; never port-forward it to the public internet.
- Use harmless requests for testing.
- Review access logs for personal data before publishing screenshots or samples.
