# LAB-SSH-001 — Controlled SSH Authentication Failure

**Date:** October 11, 2026  
**Endpoint:** `kali-soc-agent` · **Tools:** Wazuh 4.14.8, OpenSSH, journald  
**Disposition:** Authorized training test; detection confirmed, no compromise demonstrated

## Objective

Test whether a failed SSH login to the Kali VM was recorded locally and detected centrally by Wazuh.

## Procedure

1. Confirmed that SSH was active: `systemctl is-active ssh`.
2. Investigated the missing `/var/log/auth.log` and confirmed that `journalctl` showed service events.
3. Verified Wazuh's `journald` collection block in `/var/ossec/etc/ossec.conf`.
4. Attempted a local SSH password login, intentionally entering one incorrect password:

```bash
ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no -o NumberOfPasswordPrompts=1 kev@127.0.0.1
```

5. Checked `journalctl` and found `Failed password for kev from 127.0.0.1`.
6. Opened **Wazuh → Threat Hunting → Events** and investigated the matching event.

## Observed alert

| Field | Value |
|---|---|
| Agent | `kali-soc-agent` |
| Source IP | `127.0.0.1` (loopback) |
| Target user | `kev` |
| Log collection | `journald` |
| Decoder | `sshd` |
| Wazuh rule | **5760** |
| Severity | **Level 5** |
| Rule description | `sshd: authentication failed.` |
| ATT&CK mappings | **T1110.001** (Password Guessing), **T1021.004** (SSH) |

The interface showed **three related authentication-failure records**, not evidence of three separate adversaries or successful intrusions.

## Conclusion

The endpoint journal and central Wazuh event matched by username, source address, time and message. **Expected detection of intentionally generated lab activity.** No containment was warranted.

## Evidence

Screenshot files were captured and sanitized, and will be placed under [screenshots/](../screenshots/) after upload.

## Follow-up validation — November 9, 2026

**Why I repeated the test:** One incorrect-password event confirmed that logging worked, but it did not show how the monitoring setup handled repeated activity. I ran a second controlled test so the portfolio would include both single-event detection and a sequence of failed logins.

**What changed:** From the Kali VM, I ran a short Hydra password-guessing test with 12 deliberately invalid passwords against local SSH (`127.0.0.1:22`). Hydra reported **12 login tries and 0 valid passwords**. I compared the SSH service records using `sudo journalctl -u ssh --since "10 minutes ago" | grep "Failed password"` with the Wazuh Threat Hunting view.

**What Wazuh showed:** The filtered view contained **12 hits**. Multiple events were classified as **rule 5760, level 5** (authentication failed), while at least one was **rule 5763, level 10** (brute-force attempt). Document details showed the monitored `kali-soc-agent`, `sshd` decoder, user `kev`, and the loopback source `127.0.0.1`.

**Interpretation:** The additional test demonstrated that the endpoint journal and Wazuh alerts reflected the same intentionally generated failed-login activity. It also demonstrated a more severe brute-force classification during repeated attempts. **No successful SSH login, real-world adversary, or compromise was demonstrated.** The 12 hits are filtered alert records, not 12 distinct attackers.

**Screenshot captions (November 9):**

1. `ssh-test-terminal.png` — Hydra test result (no valid password) and Kali `journalctl` failed-password entries.
2. `wazuh-alert-details.png` — Detail view showing SSH decoder, monitored agent, source, and brute-force rule description.
3. `wazuh-detection-summary.png` — Wazuh Threat Hunting table with 12 hits, level-5 authentication failures and a level-10 brute-force alert.

The screenshots were cropped to remove unrelated browser tabs and desktop content; the event details were not modified. Image files are prepared for upload under `../screenshots/`.
