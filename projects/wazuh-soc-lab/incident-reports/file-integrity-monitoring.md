# LAB-FIM-001 — Real-Time File Integrity Monitoring

**Test period:** October 13–15, 2026  
**Endpoint:** `kali-soc-agent` · **Tools:** Wazuh FIM, Kali Linux, Linux shell  
**Disposition:** Authorized file modification; real-time detection confirmed

## Objective

Configure Wazuh to watch a safe test folder, change a test file, and validate the resulting file-integrity event in the SIEM.

## Setup

Created a test folder, backed up the agent configuration and edited `/var/ossec/etc/ossec.conf` in the existing `<syscheck>` section:

```bash
mkdir -p /home/kev/soc-fim-lab
sudo cp /var/ossec/etc/ossec.conf /var/ossec/etc/ossec.conf.bak
```

```xml
<directories check_all="yes" realtime="yes">/home/kev/soc-fim-lab</directories>
```

Restarted the agent and verified `active (running)` using:

```bash
sudo systemctl restart wazuh-agent
sudo systemctl status wazuh-agent --no-pager
```

## Controlled test

```bash
echo "SOC FIM test - original file" > /home/kev/soc-fim-lab/test.txt
cat /home/kev/soc-fim-lab/test.txt

echo "SOC FIM test - file modified" >> /home/kev/soc-fim-lab/test.txt
cat /home/kev/soc-fim-lab/test.txt
```

## Verified event

In **Wazuh → File Integrity Monitoring → Events**, the expanded event showed:

| Field | Observed value |
|---|---|
| `syscheck.path` | `/home/kev/soc-fim-lab/test.txt` |
| `syscheck.event` | `modified` |
| `syscheck.mode` | `realtime` |
| `rule.level` | **7** |
| `syscheck.changed_attributes` | size, mtime, md5, sha1, sha256 |
| ATT&CK | **T1565.001** (Stored Data Manipulation) |

Before/after cryptographic hashes and size values differed, supporting the finding that file contents changed.

We also observed scheduled system-file alerts for `/etc/resolv.conf` and an EFI file; they are **not** claimed as the purpose-built file test.

## Conclusion

The dedicated file-change test produced a matching **real-time** Wazuh event. This confirms endpoint FIM detection and alert investigation, not an actual compromise.

## Evidence

The cleaned installation, service-status, baseline-file, FIM dashboard and expanded-event screenshots are staged for addition under [screenshots/](../screenshots/).