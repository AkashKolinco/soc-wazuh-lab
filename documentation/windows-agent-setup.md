# Windows 10 Agent Setup

How the Windows 10 endpoint was enrolled in the Wazuh manager and what telemetry was verified.

## 1. Endpoint details

| Item | Lab value |
|:--|:--|
| Agent name | `soc-windows` (agent ID `001`) |
| Address | `192.168.188.129` |
| OS | Microsoft Windows 10 Home, build `10.0.18362.30` |
| Hostname in logs | `DESKTOP-Q8RRDQ5` |
| Agent version | `v4.14.10` |
| Manager | `192.168.188.132:1514/tcp` |
| User monitored by FIM | `dex` |

## 2. Install and enroll

1. In the Wazuh dashboard, open **Endpoints → Deploy new agent** and choose Windows.
2. Set the manager address to `192.168.188.132` and the agent name to `soc-windows`.
3. Run the generated installer on the Windows host (Administrator PowerShell):

   ```powershell
   .\wazuh-agent-4.x.msi /q WAZUH_MANAGER="192.168.188.132" WAZUH_AGENT_NAME="soc-windows"
   NET START WazuhSvc
   ```

   The agent configuration is written to
   `C:\Program Files (x86)\ossec-agent\ossec.conf`.

## 3. Service management

The service display name is **Wazuh**; the internal service name is **WazuhSvc**.

```powershell
# Inspect state
Get-Service -Name WazuhSvc
sc.exe queryex WazuhSvc
sc.exe qc WazuhSvc

# Start / restart (Administrator)
Start-Service -Name WazuhSvc
Restart-Service -Name WazuhSvc
```

**Observed issue:** `Restart-Service` initially failed with *"Cannot open WazuhSvc service on computer ."*, and a later attempt failed with *"Cannot stop WazuhSvc"*. Checking `sc.exe queryex WazuhSvc` showed the service `STOPPED` with PID 0. `Start-Service` brought it back, and the log confirmed a fresh process and a new manager connection. The exact cause of the failed stop was not proven — record it as a service-state/service-control troubleshooting issue, not a confirmed permissions fault.

## 4. Log verification

```powershell
Get-Content "C:\Program Files (x86)\ossec-agent\ossec.log" -Tail 50
```

Successful startup evidence from the lab:

```text
2026/11/17 04:45:12 wazuh-agent: INFO: Started (pid: 956).
2026/11/17 04:45:12 wazuh-agent: INFO: Windows version is 6.0 or newer.
(Microsoft Windows 10 Home [Ver: 10.0.18362.30] | DESKTOP-Q8RRDQ5 | x86_64 - Wazuh v4.14.10)
2026/11/17 04:45:12 wazuh-agent: INFO: Analyzing event log: 'Application'.
2026/11/17 04:45:12 wazuh-agent: INFO: Analyzing event log: 'Security'.
2026/11/17 04:45:12 wazuh-agent: INFO: Analyzing event log: 'System'.
2026/11/17 04:45:12 wazuh-agent: INFO: Trying to connect to server ([192.168.188.132]:1514/tcp).
2026/11/17 04:45:12 wazuh-agent: INFO: (4102): Connected to the server ([192.168.188.132]:1514/tcp).
2026/11/17 04:45:12 wazuh-agent: INFO: File integrity monitoring scan started.
2026/11/17 04:45:12 wazuh-agent: INFO: Real-time file integrity monitoring started.
```

`(4102): Connected to the server` is the key line proving agent-to-manager communication.

## 5. Windows event log collection

The agent analyzed the **Application**, **Security** and **System** channels. In the dashboard, the custom SOC overview showed failed logons and account-management events: **4720** (account created), **4722** (account enabled), **4724** (password reset), **4732** (member added to security-enabled group) and **4738** (account changed).

Raw events were reviewed in Discover with fields such as `agent.name`, `data.win.eventdata.image`, `processId`, `hashes` and `targetObject`. Evidence: [`../screenshots/wazuh-discover-events.png`](../screenshots/wazuh-discover-events.png).

## 6. Sysmon telemetry

Sysmon was installed with a configuration file:

```powershell
.\sysmon64.exe -accepteula -i .\sysmons.xml
Get-Service -Name Sysmon64 -ErrorAction SilentlyContinue
Get-Service -Name Sysmon  -ErrorAction SilentlyContinue
```

Confirm the configuration filename (`sysmons.xml` vs `sysmon.xml`) on your own host before running the command. Ingestion was verified in Discover: Sysmon fields including `data.win.eventdata.originalFileName`, `imageLoaded`, `hashes` (SHA1/MD5/SHA256), `signature`, `signed` and `targetObject` were searchable for `soc-windows`.

## 7. Security Configuration Assessment (SCA)

The agent log showed the CIS policy being loaded and evaluated:

```text
sca: INFO: Module started.
sca: INFO: Loaded policy 'C:\Program Files (x86)\ossec-agent\ruleset\sca\cis_win10_enterprise.yml'
sca: INFO: Starting Security Configuration Assessment scan.
sca: INFO: Evaluation finished for policy '...\cis_win10_enterprise.yml'
sca: INFO: Security Configuration Assessment scan finished.
Duration: 28 seconds.
```

The scan completion is verified. A screenshot of the SCA results page with pass/fail counts was **not** captured in this lab — add it to `../screenshots/` before presenting a compliance percentage.

## 8. Recommended follow-up

- Confirm the dashboard shows `soc-windows` as **active** after a clean restart.
- Capture the SCA policy results with total/passed/failed checks.
- Keep the agent and manager versions aligned, and record both in your notes.

See also: [fim-configuration.md](fim-configuration.md) and the [FIM investigation](../investigation/fim-investigation.md).
