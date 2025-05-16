# Systemd
Manage systemd units.

## Requirements
[supported platforms](https://github.com/r-pufky/ansible_systemd/blob/main/meta/main.yml)

## Role Variables
[defaults](https://github.com/r-pufky/ansible_systemd/tree/main/defaults)

Use `section` in defaults to reference specific systemd variables, datatypes,
and usage for variables.

## Dependencies
**galaxy-ng** roles cannot be used independently. Part of
[r_pufky.deb](https://github.com/r-pufky/ansible_collection_deb) collection.

## Example Playbook
Standard ansible built-ins may be used **after** configuration. Systemd will be
force reloaded on role completion to ensure unit availability.

Currently supported units are:
* mount
* automount
* service
* timer

### Systemd cronjob example
Create a systemd timer that periodically reboots a system (e.g. cronjob).

host_vars/client.example.com/vars/systemd.yml
``` yaml
systemd_services:
  - name: 'reboot'
    state: 'present'
    drop_in: false
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
systemd_timers:
  - name: 'reboot'
    state: 'present'
    drop_in: false
    unit:
      description: 'periodic system reboot timer'
    timer:
      unit: 'reboot.service'
      on_boot_sec: '1month'
    install:
      wanted_by:
        - 'timers.target'
```

Apply the role
``` yaml
- name: 'Manage systemd'
  ansible.builtin.include_role:
    name: 'r_pufky.deb.systemd'
```

## Systemd mount / automount example
Create filesystem mounts and automounts for NFS shares.

host_vars/client.example.com/vars/systemd.yml
``` yaml
systemd_mounts:
  - name: 'data-pictures'
    state: 'present'
    drop_in: false
    unit:
      description: 'mount NFS share /data/pictures'
    mount:
      what: '172.16.24.192:/home'
      where: '/data/pictures'
      options:
        - 'vers=4'
        - 'minorversion=2'
      type: 'nfs'
      timeout_sec: 30
    install:
      wanted_by:
        - 'multi-user.target'
  - name: 'mnt-test'
    state: 'present'
    drop_in: false
    unit:
      description: 'mount a new tmpfs filesystem to /mnt/test'
    mount:
      what: 'tmpfs'
      where: '/mnt/test'
      type: 'tmpfs'
      options:
        - 'noatime'
        - 'nosuid'
        - 'nodev'
        - 'noexec'
        - 'mode=1777'
    install:
      wanted_by:
        - 'multi-user.target'
  - name: 'tmp'
    state: 'present'
    drop_in: true
    unit:
      description: 'override default /tmp tmpfs mode'
    mount:
      options:
        - 'mode=1222'
systemd_automounts:
  - name: 'mnt-test'
    state: 'present'
    drop_in: false
    unit:
      description: 'automount /mnt/test instead of static mount'
    automount:
      where: '/mnt/test'
    install:
      wanted_by:
        - 'multi-user.target'
```

Apply the role
``` yaml
- name: 'Manage systemd'
  ansible.builtin.include_role:
    name: 'r_pufky.deb.systemd'
```

### Remove systemd units
Units will be disable then removed; this is done after any unit creation and
enabling.

host_vars/client.example.com/vars/systemd.yml
``` yaml
systemd_services:
  - name: 'my_service'
    state: 'absent'
    drop_in: false
systemd_timers:
  - name: 'my_timer'
    state: 'absent'
    drop_in: false
```

Apply the role
``` yaml
- name: 'Manage systemd'
  ansible.builtin.include_role:
    name: 'r_pufky.deb.systemd'
```

## Override existing systemd units (drop-ins)
Using unit drop-ins (overrides) are possible, allowing for tweaking of existing
systemd services without re-defining the entire configuration. This is
supported for all supported units. Overrides are stored in
`systemd/{UNIT}.d/override.conf`.


Override NFS server and disable V3
``` yaml
- name: 'Manage systemd'
  ansible.builtin.include_role:
    name: 'r_pufky.deb.systemd'
  vars:
    systemd_services:
      - name: 'nfs-server'
        state: 'present'
        drop_in: true
        service:
          exec_start:
            - ''
            - '/usr/bin/rpc.nfsd --no-nfs-version 3'
```

Sections may be created without values to render headers only; enabling
extremely targeted drop-in use. See [headers](https://github.com/r-pufky/ansible_systemd/tree/main/templates/header)
for complete list.

Override User/Group for Service
``` yaml
- name: 'Manage systemd'
  ansible.builtin.include_role:
    name: 'r_pufky.deb.systemd'
  vars:
    systemd_services:
      - name: 'nfs-server'
        state: 'present'
        drop_in: true
        service: {}  # service header only
        exec:
          user: 'nfs'
          group: 'nfs'
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

## Development
Configure [environment](https://github.com/r-pufky/ansible_collection_docs/blob/main/dev/environment/README.md)

Run all unit tests:
``` bash
molecule test --all
```

### Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License](https://www.tldrlegal.com/license/gnu-affero-general-public-license-v3-agpl-3-0)
 [(direct link)](https://github.com/r-pufky/ansible_systemd/blob/main/LICENSE)

## Author Information
PGP Fingerprint: [466EEC2B67516C7117C85CE3A0BC35D16698BAB9](https://keys.openpgp.org/vks/v1/by-fingerprint/466EEC2B67516C7117C85CE3A0BC35D16698BAB9)
| [github gist](https://gist.github.com/r-pufky/a8df36977c55b5bb20829267c4c49d22)
