# HW 11 - Ansible Hardening Change Review

## Playbook Findings & Corrections

| Line | Issue and evidence | Risk | Suggested correction |
| :--- | :--- | :--- | :--- |
| **09** | Right now, `line: 'PermitRootLogin yes'` is actually turning root login on instead of off, which is the exact opposite of what we want. Also, if that line is commented out in the config file, it won't get matched at all. | Leaving root SSH login open makes it way easier for attackers to guess passwords and get straight into our servers with full admin access. | Change line 09 to `line: 'PermitRootLogin no'` and update line 08 to `'^#?\s*PermitRootLogin'` so it catches the line even if it's commented out. |
| **12** | Running `sysctl -w net.ipv4.ip_forward=0` as a one off command only changes the setting temporarily, it doesn't actually save it to the system's config files. | As soon as a server reboots, IPv4 forwarding will turn right back on without anyone noticing. | Replace that command with Ansible's built-in `sysctl` module (`ansible.posix.sysctl`) so the setting gets saved properly and stays off after a reboot. |
| **17** | Adding `changed_when: false` forces Ansible to report that nothing changed, even if it actually had to uninstall `telnet` from a server. | We wouldn't be able to tell from our logs that `telnet` was ever on the machine in the first place, and any follow-up actions wouldn't run. | Just remove `changed_when: false` on line 17 so Ansible accurately reports when it removes `telnet`. |
| **23** | Setting `ignore_errors: true` tells Ansible to keep going and pretend everything is fine even if `auditd` fails to start. | Our rules say the rollout has to stop if `auditd` fails. With this line in place, a server could go live with no activity logging while looking totally fine in the report. | Remove `ignore_errors: true` on line 23 so the playbook immediately stops if `auditd` fails on a server. |

---

## Pre-Rollout Check

* **What I'd run:**  
  Before touching production, I'd run a dry run with diffs enabled against a staging inventory:
  ```bash
  ansible-playbook --check --diff -i inventory/rocky_hosts.ini hardening.yml
