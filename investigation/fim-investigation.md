# Alert Investigation: Multiple File Integrity Events on `soc-windows`

**Investigation ID:** SOC-FIM-2026-1117-01
**Analyst:** Lab owner
**Date:** 2026-11-17
**Severity:** Low-to-medium (FIM rules 550/553, level 7; rule 554, level 5)
**Status:** Closed — planned activity, no malicious behaviour found

---

## 1. Alert summary

The Wazuh dashboard's File Integrity Monitoring view showed **8 `syscheck` alerts** for agent `001 soc-windows` (`192.168.188.129`) within a 7-day window, with a concentrated burst of activity on 2026-11-17. The events involved file creation, checksum changes and deletion under `C:\Users\dex\Desktop`, a directory explicitly monitored by the lab's FIM configuration.

Primary rules triggered:

| Rule | Level | Description |
|:--|:--|:--|
| 554 | 5 | File added to the system |
| 550 | 7 | Integrity checksum changed |
| 553 | 7 | File deleted |

Evidence: [FIM overview](../screenshots/fim-alerts-overview.png) · [FIM event list](../screenshots/fim-alert-events.png).

## 2. Initial triage questions

1. Are the affected paths inside a monitored, expected directory?
2. Is the activity consistent with authorized lab testing?
3. Did any change occur in the persistence-sensitive Startup folder?
4. Is there corroborating process or logon evidence of malicious behaviour?

## 3. Timeline (all times 2026-11-17, local lab time)

| Time | File | Event | Rule |
|:--|:--|:--|:--|
| 04:47:08 | `c:\users\dex\desktop\new text document.txt` | added | 554 |
| 04:47:13 | `c:\users\dex\desktop\new text document.txt` | deleted | 553 |
| 04:47:13 | `c:\users\dex\desktop\hello.txt` | added | 554 |
| 04:47:23 | `c:\users\dex\desktop\hello.txt` | modified (checksum changed) | 550 |
| 08:41:04 | `c:\users\dex\desktop\hello.txt` | modified (checksum changed) | 550 |
| 08:53:44 | `c:\users\dex\desktop\new text document.txt` | added | 554 |
| 08:53:48 | `c:\users\dex\desktop\new text document.txt` | deleted | 553 |
| 08:53:48 | `c:\users\dex\desktop\dip.txt` | added | 554 |

All events were attributed to user `dex` and agent `soc-windows`, rule group `syscheck`.

## 4. Evidence reviewed

- **FIM event list** — `syscheck.path`, `syscheck.event`, rule description, level and rule ID for all 8 events.
- **FIM overview dashboard** — action breakdown (added / modified / deleted), rule distribution and top agent (`soc-windows`, 8 events).
- **Agent startup log** — FIM baseline scan completed and real-time monitoring started, confirming the alerts came from a healthy, configured agent.
- **Discover / Sysmon data** — Windows process events (`svchost.exe`, `taskhostw.exe`, registry `targetObject`, image loads, valid Microsoft signatures) showed normal endpoint activity, with no anomalous unsigned process tied to the file changes in the available data.
- **SOC overview dashboard** — Windows account-change and failed logon panels showed no suspicious account creation correlated to the file events.
- **Configuration** — the monitored Desktop path was intentionally added in the lab FIM configuration, with `realtime`, `report_changes` and `check_all` enabled.

## 5. Analysis

- Every affected path is under `C:\Users\dex\Desktop`, the user-writable directory selected for FIM testing. **No events were recorded for the Windows Startup folder**, the persistence-sensitive location also monitored by the lab configuration.
- The sequence *create → modify → delete* matches the documented FIM validation procedure exactly: a harmless test file was created, its content was changed, and test files were removed after evidence collection.
- Two bursts (04:47 and 08:53) correspond to two rounds of testing, with a later single checksum change at 08:41.
- The file names (`hello.txt`, `new text document.txt`, `dip.txt`) and their location are consistent with manual test artifacts rather than application or malware behaviour.
- Correlating Windows event logs and Sysmon data did not reveal suspicious process execution, unusual account activity, or an external source tied to these changes.
- Rule 554 is informational (level 5); rules 550/553 are level 7 but are expected consequences of deliberately modifying and deleting monitored test files.

## 6. Conclusion and disposition

**Verdict:** Planned, authorized FIM validation activity — false positive in operational terms, but a **successful end-to-end detection test** for the lab.

**Disposition:** Closed. No escalation required. Test files were removed after screenshots were captured.

## 7. Actions taken

1. Preserved screenshot evidence in [`../screenshots/`](../screenshots/).
2. Documented the test procedure in [the FIM configuration guide](../documentation/fim-configuration.md).
3. Confirmed cleanup of the test files from the Desktop.
4. Recorded the validation result in the [README validation matrix](../README.md#validation-summary).

## 8. Recommendations / lessons learned

- Keep FIM focused on meaningful paths; add `ignore` entries for noisy, application-generated files to reduce alert volume.
- Preserve process context (Sysmon + Windows Security events) alongside FIM alerts — it made benign classification faster.
- Monitor the Startup folder specifically for persistence techniques such as `T1547.001` (Registry Run Keys / Startup Folder); the absence of events there was a key part of this verdict.
- Re-run the test after any agent upgrade or rule tuning to confirm detection still works.
- When writing this up for a portfolio, describe it as a *detection engineering and alert-triage exercise*, not as a real intrusion.

---

**Related files:** [FIM configuration](../configurations/windows-fim-config.xml) · [FIM setup guide](../documentation/fim-configuration.md) · [Windows agent setup](../documentation/windows-agent-setup.md)
