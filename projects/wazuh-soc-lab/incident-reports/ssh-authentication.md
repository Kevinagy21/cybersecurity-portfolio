# LAB-SSH-001 — Controlled SSH Authentication Failure

**Date:** October 11, 2026  
**Endpoint:** \`kali-soc-agent\` · **Tools:** Wazuh 4.14.8, OpenSSH, journald  
**Disposition:** Authorized training test; detection confirmed, no compromise demonstrated

## Objective

Test whether a failed SSH login to the Kali VM was recorded locally and detected centrally by Wazuh.

## Procedure

1. Confirmed that SSH was active: \`systemctl is-active ssh\`.
2. Investigated the missing \`/var/log/auth.log\` and confirmed that \`journalctl\` showed service events.
3. Verified Wazuh's \`journald\` collection block in \`/var/ossec/etc/ossec.conf\`.
4. Attempted a local SSH password login, intentionally entering one incorrect password:

\`\`\`bash
ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no -o NumberOfPasswordPrompts=1 kev@127.0.0.1
\`\`\`

5. Checked \`journalctl\` and found \`Failed password for kev from 127.0.0.1\`.
6. Opened **Wazuh → Threat Hunting → Events** and investigated the matching event.

## Observed alert

| Field | Value |
|---|---|
| Agent | \`kali-soc-agent\` |
| Source IP | \`127.0.0.1\` (loopback) |
| Target user | \`kev\` |
| Log collection | \`journald\` |
| Decoder | \`sshd\` |
| Wazuh rule | **5760** |
| Severity | **Level 5** |
| Rule description | \`sshd: authentication failed.\` |
| ATT&CK mappings | **T1110.001** (Password Guessing), **T1021.004** (SSH) |

The interface showed **three related authentication-failure records**, not evidence of three separate adversaries or successful intrusions.

## Conclusion

The endpoint journal and central Wazuh event matched by username, source address, time and message. **Expected detection of intentionally generated lab activity.** No containment was warranted.

## Evidence

Screenshot files were captured and sanitized, and will be placed under [screenshots/](../screenshots/) after upload.