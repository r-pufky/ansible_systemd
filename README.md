# Systemd
Manage systemd units.

## Requirements
[supported platforms](https://github.com/r-pufky/ansible_systemd/blob/main/meta/main.yml)

## Role Variables
[defaults](https://github.com/r-pufky/ansible_systemd/tree/main/defaults)

Use `section` in defaults to reference specific systemd variables, datatypes,
and usage for variables.

## Dependencies
N/A

## Example Playbook
Standard ansible built-ins may be used after configuration (stop, start,
reload, etc). Systemd will be force reloaded on role completion to ensure unit
availability after role application.

### Systemd cronjob example
Create a systemd timer that periodically reboots a system (e.g. cronjob).

host_vars/client.example.com/vars/systemd.yml
``` yaml
systemd_services:
  - name: 'reboot'
    unit:
      description: 'periodic system reboot service'
      requires:
        - 'reboot.timer'
    service:
      type: 'simple'
      exec_start: ['perl -e "sleep(rand(30))";/sbin/reboot']
    exec:
      user: 'root'
      group: 'root'
    state: 'present'
systemd_timers:
  - name: 'reboot'
    unit:
      description: 'periodic system reboot timer'
    timer:
      unit: 'reboot.service'
      on_boot_sec: '1month'
    install:
      wanted_by:
        - 'timers.target'
    state: 'present'
```

Apply the role
``` yaml
- name: 'Manage systemd'
  ansible.builtin.include_role:
    name: 'r_pufky.srv.systemd'
```

### Remove systemd units
Units will be disable then removed; this is done after any unit creation and
enabling.

host_vars/client.example.com/vars/systemd.yml
``` yaml
systemd_services:
  - name: 'my_service'
    state: 'absent'
systemd_timers:
  - name: 'my_timer'
    state: 'absent'
```

Apply the role
``` yaml
- name: 'Manage systemd'
  ansible.builtin.include_role:
    name: 'r_pufky.srv.systemd'
```

### Manage normally with `ansible.builtin.service`
Once systemd units are created and role is applied, units may be managed
normally with ansible.

``` yaml
- name: 'Restart reboot service'
  ansible.builtin.service:
    name: 'reboot'
    state: 'restarted'
```

## Unit Testing
Test framework requires molecule and rootless podman setup.

Run all unit tests:
``` bash
molecule test --all
```

## Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License](https://www.tldrlegal.com/license/gnu-affero-general-public-license-v3-agpl-3-0)
 [(direct link)](https://github.com/r-pufky/ansible_users/blob/main/LICENSE)

## Author Information
PGP Fingerprint: [466EEC2B67516C7117C85CE3A0BC35D16698BAB9](https://keys.openpgp.org/vks/v1/by-fingerprint/466EEC2B67516C7117C85CE3A0BC35D16698BAB9)
| [github gist](https://gist.github.com/r-pufky/a8df36977c55b5bb20829267c4c49d22)
