# Setup and Troubleshooting Notes

## Environment

- macOS host using VMware Fusion, with Ubuntu Server 24.04.5 ARM64 and Kali Linux ARM64 VMs.
- Ubuntu host: Wazuh 4.14.8 manager, indexer, dashboard and Filebeat.
- Kali endpoint: Wazuh ARM64 agent, SSH server and `journald`.
- Dashboard accessed through Safari on the host over HTTPS on the private virtual network.

## Setup notes

- The Ubuntu Server installer ISO was disconnected from the virtual CD/DVD drive after installation to avoid booting into the setup again.
- The VMware virtual disk was resized to 65 GB; its Linux LVM root filesystem was expanded to approximately 61 GB.
- Verified Wazuh agent active status using `sudo systemctl status wazuh-agent --no-pager`.
- Verified agent monitoring of Linux `journald` via `<log_format>journald</log_format>` and `<location>journald</location>`.
- For FIM, configured `/home/kev/soc-fim-lab` with `check_all="yes"` and `realtime="yes"`, then restarted the agent.

## Lessons learned

1. Troubleshooting a missing auth log requires checking Linux systemd journals instead of assuming there is no audit trail.
2. File-change dashboards can include unrelated scheduled alerts. Confirm the exact `syscheck.path` before drawing conclusions.
3. Use matching timestamps, users, log text, and rule descriptions when correlating endpoint events with central alerts.
4. Keep snapshots of before-and-after system state, but never publish live credentials.

## Safety

All activity was performed in an authorized local lab. Screenshots showing passwords were redacted. Rotate any leaked Wazuh administrative password before reusing this lab.