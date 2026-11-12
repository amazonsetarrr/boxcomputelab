# boxcomputelab

## IOS-XE pre-check playbook

`playbooks/ios_xe_precheck.yml` runs 10 read-only `show` commands on Cisco IOS-XE
devices and writes the output to one report file per host.

| # | Command |
|---|---------|
| 1 | `show version` |
| 2 | `show clock` |
| 3 | `show inventory` |
| 4 | `show ip interface brief` |
| 5 | `show interfaces description` |
| 6 | `show ip route summary` |
| 7 | `show processes cpu` |
| 8 | `show memory statistics` |
| 9 | `show logging` |
| 10 | `show ntp status` |

The command list is the `precheck_commands` variable in the playbook.

### Layout

```
ansible.cfg                  # defaults (inventory path, timeouts)
requirements.yml             # cisco.ios and ansible.netcommon
inventory/hosts.yml          # example devices, credentials from env vars
playbooks/ios_xe_precheck.yml
templates/precheck_report.j2 # report format
reports/                     # generated output (git-ignored)
```

### Setup

```bash
pip install ansible-core
ansible-galaxy collection install -r requirements.yml
```

### Run

```bash
export IOSXE_USER=admin
export IOSXE_PASSWORD='...'
export IOSXE_ENABLE_PASSWORD='...'

# Edit inventory/hosts.yml first, then:
ansible-playbook playbooks/ios_xe_precheck.yml

# Limit to one device
ansible-playbook playbooks/ios_xe_precheck.yml --limit iosxe-rtr-01

# Fail the host if IOS-XE rejects any command (for example, an unsupported command on that platform)
ansible-playbook playbooks/ios_xe_precheck.yml -e precheck_fail_on_error=true
```

Each run writes `reports/<host>_<UTC timestamp>.txt`. The report lists every
command with its output and a count of commands that IOS-XE rejected.

### Notes

- Every command is read-only and does not change device configuration.
- Credentials are never stored in the repository. Set them through the environment variables above.
- Some `show` commands do not exist on every platform. A rejected command is reported, not treated as a crash, unless `precheck_fail_on_error=true`.
