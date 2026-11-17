# File Integrity Monitoring (FIM)

## 1. Objective

Monitor two meaningful Windows locations so that changes to a persistence-sensitive path and a user-activity path generate alerts:

- `%PROGRAMDATA%\Microsoft\Windows\Start Menu\Programs\Startup` — a common persistence location.
- `C:\Users\dex\Desktop` — a user-writable area where test files can be created safely.

## 2. Configuration

Configuration file on the endpoint:

```text
C:\Program Files (x86)\ossec-agent\ossec.conf
```

Add the `<syscheck>` block from [`../configurations/windows-fim-config.xml`](../configurations/windows-fim-config.xml) inside `<ossec_config>`:

```xml
<syscheck>
  <disabled>no</disabled>
  <frequency>43200</frequency>
  <scan_on_start>yes</scan_on_start>
  <directories realtime="yes">%PROGRAMDATA%\Microsoft\Windows\Start Menu\Programs\Startup</directories>
  <directories realtime="yes" report_changes="yes" check_all="yes">C:\Users\dex\Desktop</directories>
</syscheck>
```

**Fix applied:** an earlier fragment contained `report_changes=""yes"`, which is invalid XML. It was corrected to `report_changes="yes"`.

### Option reference

| Option | Meaning |
|:--|:--|
| `realtime="yes"` | Real-time monitoring on supported filesystems |
| `report_changes="yes"` | Reports content changes for supported files (review privacy impact) |
| `check_all="yes"` | Enables the supported set of integrity checks |
| `frequency` | Periodic scan interval — 43,200 seconds = 12 hours |
| `scan_on_start` | Runs a baseline scan when the agent starts |

After editing, restart the agent and watch `ossec.log`:

```powershell
Restart-Service -Name WazuhSvc
Get-Content "C:\Program Files (x86)\ossec-agent\ossec.log" -Tail 50
```

## 3. Startup evidence

```text
INFO: File integrity monitoring scan frequency: 43200 seconds
INFO: File integrity monitoring scan started.
INFO: File integrity monitoring scan ended.
INFO: FIM sync module started.
INFO: Real-time file integrity monitoring started.
```

This confirms the FIM module started and the baseline scan completed.

## 4. Safe validation procedure

1. Create a harmless file on the monitored Desktop.
2. Write a short string to it.
3. Modify and save the file to trigger a checksum change.
4. Delete the file to exercise the deletion rule.
5. Search Wazuh for `rule.groups: syscheck`, then filter by `syscheck.path`.
6. Capture the alert list and save it as evidence.
7. Remove test files and document the cleanup.

```powershell
New-Item -Path "C:\Users\dex\Desktop\fim-test.txt" -ItemType File
Set-Content -Path "C:\Users\dex\Desktop\fim-test.txt" -Value "Wazuh FIM testing"
Set-Content -Path "C:\Users\dex\Desktop\fim-test.txt" -Value "Testing file modification detection"
Remove-Item "C:\Users\dex\Desktop\fim-test.txt"
```

## 5. Result observed in the lab

The FIM dashboard showed **8 alerts** for `soc-windows` over the test window:

| Timestamp (2026-11-17) | File | Action | Rule | Description |
|:--|:--|:--|:--|:--|
| 08:53:48 | `c:\users\dex\desktop\dip.txt` | added | 554 | File added to the system |
| 08:53:48 | `c:\users\dex\desktop\new text document.txt` | deleted | 553 | File deleted |
| 08:53:44 | `c:\users\dex\desktop\new text document.txt` | added | 554 | File added to the system |
| 08:41:04 | `c:\users\dex\desktop\hello.txt` | modified | 550 | Integrity checksum changed |
| 04:47:23 | `c:\users\dex\desktop\hello.txt` | modified | 550 | Integrity checksum changed |
| 04:47:13 | `c:\users\dex\desktop\hello.txt` | added | 554 | File added to the system |
| 04:47:13 | `c:\users\dex\desktop\new text document.txt` | deleted | 553 | File deleted |
| 04:47:08 | `c:\users\dex\desktop\new text document.txt` | added | 554 | File added to the system |

All events were attributed to user `dex`, agent `001 soc-windows`, rule group `syscheck`.

Evidence:

- [FIM overview dashboard](../screenshots/fim-alerts-overview.png)
- [FIM event list](../screenshots/fim-alert-events.png)

This is end-to-end validation: configuration → agent detection → manager alert → dashboard evidence.

## 6. Cleanup and recommendations

- Remove all test files after collecting evidence.
- Keep `report_changes` limited to directories that do not contain sensitive content.
- Tune `ignore` entries for noisy application-generated files if false positives appear.
- Use the same test procedure again after any agent or rule update.
- The full alert analysis is in [the FIM investigation](../investigation/fim-investigation.md).
