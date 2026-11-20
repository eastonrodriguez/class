# HW 11 - Ansible Hardening Change Review

## Playbook Findings & Corrections

| Line | Issue and evidence | Risk | Suggested correction |
| :--- | :--- | :--- | :--- |
| **09** | Right now, `line: 'PermitRootLogin yes'` is actually turning root login *on*, which is the exact opposite of our baseline (`PermitRootLogin no`). Also, if that line is commented out in the config file, `^PermitRootLogin` won't even match it. | Leaving direct root SSH logins enabled makes it way easier for attackers to brute-force or credential-stuff their way straight into our servers as root. | I'd swap line 09 to `line: 'PermitRootLogin no'` and tweak the regex on line 08 to `'^#?\s*PermitRootLogin'` so it catches commented-out lines too. |
| **12** | Using `ansible.builtin.command: sysctl -w net.ipv4.ip_forward=0` only changes the setting in live memory—it never writes it to `/etc/sysctl.conf` or `/etc/sysctl.d/`. Plus, `command` isn't idempotent, so it'll always show up as `changed`. | The moment one of these Rocky boxes reboots, IPv4 forwarding will turn right back on, breaking our baseline without anyone noticing. | I'd ditch the raw `command` task and use the `ansible.posix.sysctl` module instead (`name: net.ipv4.ip_forward`, `value: '0'`, `sysctl_set: true`, `state: present`, `reload: true`) so it sticks across reboots. |
| **17** | Adding `changed_when: false` to the `dnf` task forces Ansible to report `ok` even if it actually had to uninstall `telnet` from a server. | It completely hides configuration drift in our logs—we wouldn't know if `telnet` somehow ended up on a box, and any handler waiting on that change would never trigger. | I'd just delete `changed_when: false` on line 17 so `dnf` tells us the truth when it removes the package. |
| **23** | Putting `ignore_errors: true` on the `auditd` task tells Ansible to shrug and keep rolling even if the logging service fails to enable or start. | Our baseline specifically says an `auditd` failure has to stop the rollout. With this in place, we could push a server into production with zero security auditing while the playbook claims everything is green. | I'd remove `ignore_errors: true` on line 23 so the playbook immediately fails and stops on any host where `auditd` won't run. |

---

## Pre-Rollout Check

* **What I'd run:**  
  Before touching production, I'd run a dry run with diffs enabled against a staging inventory:
  ```bash
  ansible-playbook --check --diff -i inventory/rocky_hosts.ini hardening.yml
