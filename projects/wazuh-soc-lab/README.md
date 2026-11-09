# Wazuh SIEM — SOC Monitoring & Threat Detection Lab

**Status:** Core lab complete (October 2026); additional SSH detection validation performed November 9, 2026  
**Environment:** Ubuntu Server 24.04.5 ARM64, Kali Linux ARM64, VMware Fusion  
**Stack:** Wazuh 4.14.8 · OpenSSH · Linux journald · File Integrity Monitoring

> All tests were intentionally generated in an authorized home lab. These detections are not evidence of a real attacker or a compromise.

## Project overview

Built a two-VM SOC monitoring environment. An Ubuntu Server VM hosted the centralized Wazuh manager, indexer, and dashboard. A Kali Linux endpoint sent security telemetry using an installed Wazuh agent (`kali-soc-agent`). Initially verified two detection scenarios: an intentionally failed SSH login and a real-time modification of a monitored test file. On November 9, 2026, I repeated the SSH test with multiple intentionally incorrect passwords to check that Wazuh would recognize a pattern instead of just a single failure.

## Architecture

```mermaid
flowchart LR
  Kali["Kali Linux ARM64 · OpenSSH, journald, Wazuh agent"] -->|Endpoint telemetry| SIEM["Ubuntu ARM64 · Wazuh Manager, Indexer, Dashboard"]
  SIEM -->|HTTPS| Mac["Mac · Safari"]
```

Ubuntu VM: 4 virtual CPUs, approximately 8 GB RAM, 65 GB virtual disk with an approximately 61 GB root filesystem. VMs communicate on a private virtual network.

## Verified milestones

| Milestone | Evidence |
|---|---|
| Wazuh installed | Indexer, manager, Filebeat and dashboard initialized |
| Agent enrolled | Dashboard displayed one active agent |
| SSH failure detected | Wazuh rule **5760**, level **5**, after a controlled incorrect password |
| Repeated SSH attempts detected (Nov 9) | **12 hits** in the filtered Wazuh view, including repeated rule **5760** authentication failures and rule **5763** brute-force detection, level **10** |
| Linux log collection | Authentication failure confirmed in Kali's `journald` |
| FIM configured | Monitored `/home/kev/soc-fim-lab` using `realtime="yes"` |
| File modification detected | `test.txt` produced a real-time `modified` event and changed hashes |

## Follow-up: November 9, 2026 — repeated SSH password test

I wanted to check whether the original authentication alert was repeatable and whether several incorrect passwords would produce a higher-severity detection. On the Kali lab endpoint, I ran a short, controlled Hydra password-guessing test against `ssh://127.0.0.1` with twelve intentionally wrong candidates. Hydra reported **0 valid passwords found**. I then used `journalctl -u ssh` to inspect the failed-password records and opened Wazuh Threat Hunting to compare the corresponding alerts.

The filtered Wazuh view showed **12 hits**. Most visible entries were **rule 5760 / level 5** (`sshd: authentication failed`); a higher-severity entry was **rule 5763 / level 10**, describing a brute-force attempt. A Wazuh document-details view matched the monitored agent (`kali-soc-agent`), SSH decoder (`sshd`), user (`kev`), and loopback address (`127.0.0.1`). This shows the detection pipeline responding to repeated attempts—not a successful login or proof of a real attack. The test was conducted against my own VM.

**Evidence captures:** (1) local SSH test and journal output, (2) individual Wazuh alert details, and (3) Wazuh Threat Hunting event summary. Screenshots are preserved without altering event values; only unrelated browser/desktop areas are cropped. See the [updated SSH investigation](incident-reports/ssh-authentication.md) for interpretation.

## Investigation reports

- [LAB-SSH-001 — Controlled SSH Authentication Failure](incident-reports/ssh-authentication.md)
- [LAB-FIM-001 — Real-Time File Integrity Monitoring](incident-reports/file-integrity-monitoring.md)
- [Lab setup, configuration and troubleshooting notes](lab-configuration.md)

## Screenshot evidence

**New November 9 screenshots have been cropped for privacy, with original logs and counts unchanged. The image files are prepared for separate upload.** Evidence captured includes Wazuh installation, 0-to-1 agent connection, SSH terminal/journal and alert details, FIM agent configuration, test file baseline, FIM monitoring overview, and a matching real-time `test.txt` modification alert.

The screenshots are supplied in the companion evidence package. The following image files are intended for `screenshots/`: `ssh-test-terminal.png`, `wazuh-alert-details.png`, and `wazuh-detection-summary.png`.

## Skills demonstrated

SIEM deployment · Linux endpoint monitoring · log analysis · SSH authentication event triage · MITRE ATT&CK mapping · Wazuh FIM · file hash verification · Linux VM administration · incident reporting.

## Key takeaways

A missing `/var/log/auth.log` does not mean authentication logs are absent; `journald` contained the SSH records. The SSH alert was correlated against the endpoint journal rather than assuming every alert was an attack. For FIM, the matching test-file path and before/after hashes confirmed actual content modification.

**Security:** No passwords, generated keys, or certificate files are included. Any previously disclosed Wazuh administrator credential should be rotated before the lab is reused.