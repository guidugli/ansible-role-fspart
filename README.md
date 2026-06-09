# Ansible Role: fspart

[![CI](https://github.com/guidugli/ansible-role-fspart/actions/workflows/CI.yml/badge.svg)](https://github.com/guidugli/ansible-role-fspart/actions/workflows/CI.yml)
[![Release](https://github.com/guidugli/ansible-role-fspart/actions/workflows/release.yml/badge.svg)](https://github.com/guidugli/ansible-role-fspart/actions/workflows/release.yml)
[![Galaxy](https://img.shields.io/badge/galaxy-guidugli.fspart-blue.svg)](https://galaxy.ansible.com/ui/standalone/roles/guidugli/fspart/)
[![License](https://img.shields.io/badge/license-MIT-yellow.svg)](LICENSE)

Configure and validate filesystem mounts, crypttab entries, and filesystem security controls.

## Overview

This role focuses on filesystem compliance and guardrails rather than full disk provisioning.
It can:

- validate required mount points and mount options;
- optionally autofix mount configuration via `/etc/fstab` or systemd mount units;
- manage crypttab entries and optional key files;
- enable `fstrim.timer`;
- disable `autofs` when requested; and
- review or optionally remediate several filesystem permission findings.

## Features

- Explicit defaults for all supported public variables.
- Automatic role argument validation through `meta/argument_specs.yml`.
- Semantic validation in `tasks/assert.yml` for combinations that argument specs alone cannot express.
- Clean task dispatch in `tasks/main.yml`.
- Idempotent handlers for mount-unit reloads and remount/reboot follow-up.
- Metadata template in `templates/meta_main.yml.j2` with generated `meta/main.yml`.
- Shared Molecule verify playbook pattern available under `molecule/shared/`.

## Supported platforms

This role currently targets:

- Fedora 42, 43
- Ubuntu 22.04 (jammy), 24.04 (noble)
- Debian 12 (bookworm), 13 (trixie)

## Role variables

### Permission and ownership checks

```yaml
fs_run_fix_permissions: true
fs_world_writeable_fix_enabled: false
fs_log_files_fix_enabled: false
fs_unowned_detection_enabled: false
fs_ungrouped_detection_enabled: false
fs_world_writeable_excludes: []
fspart_log_exception: []
```

### Services and platform behavior

```yaml
fs_fstrim_timer_enabled: true
fs_disable_automount: true
fspart_allow_reboot: true
```

### Crypttab management

```yaml
fspart_cryptkeys_path: /etc/cryptkeys
fspart_crypttab_entries: []
fspart_cryptkeys_files: []
```

Example crypttab entries:

```yaml
fspart_crypttab_entries:
  - name: luks-test
    backing_device: UUID=6b244d35-a72b-1234-5678-4258d364809c
    password: /etc/cryptkeys/mykey
    opts: discard,luks

fspart_cryptkeys_files:
  - name: mykey
    src: cryptkeys/mykey
```

### Partition definitions

```yaml
partitions: []
```

Each partition item supports these keys:

```yaml
partitions:
  - name: /tmp
    unit_name: tmp.mount
    mount_options:
      - mode=1777
      - strictatime
      - nodev
      - nosuid
      - noexec
    validate_options:
      - nodev
      - nosuid
      - noexec
    autofix: true

  - name: /var/tmp
    unit_name: fstab
    src: tmpfs
    fstype: tmpfs
    mount_state: mounted
    mount_options:
      - strictatime
      - nodev
      - nosuid
      - noexec
    validate_options:
      - nodev
      - nosuid
      - noexec
    autofix: true
```

#### Partition item notes

- `name` is required and must be an absolute path.
- `autofix: true` requires `unit_name` and `mount_options`.
- `unit_name: fstab` uses `ansible.posix.mount`.
- `unit_name: <name>.mount` uses a systemd mount unit under `/etc/systemd/system`.
- `src` and `fstype` are required only when creating a new `fstab` entry.
- `mount_state` defaults to `mounted`.

## How it works

1. Ansible validates argument structure using `meta/argument_specs.yml`.
2. `tasks/assert.yml` validates semantic combinations and path expectations.
3. `tasks/main.yml` dispatches optional behaviors only when relevant.
4. Mount validation uses `findmnt`.
5. Permission checks can run in audit-only mode or in remediation mode, depending on the selected booleans.

## Usage

### Basic validation-only run

```yaml
---
- name: Validate filesystem hardening expectations
  hosts: all
  become: true
  roles:
    - role: guidugli.fspart
      vars:
        partitions:
          - name: /tmp
            validate_options:
              - nodev
              - nosuid
              - noexec
```

### Autofix a systemd mount unit and an fstab entry

```yaml
---
- name: Configure secure mount points
  hosts: all
  become: true
  roles:
    - role: guidugli.fspart
      vars:
        fs_world_writeable_fix_enabled: true
        partitions:
          - name: /tmp
            unit_name: tmp.mount
            mount_options:
              - mode=1777
              - strictatime
              - nodev
              - nosuid
              - noexec
            validate_options:
              - nodev
              - nosuid
              - noexec
            autofix: true
          - name: /var/tmp
            unit_name: fstab
            src: tmpfs
            fstype: tmpfs
            mount_options:
              - strictatime
              - nodev
              - nosuid
              - noexec
            validate_options:
              - nodev
              - nosuid
              - noexec
            autofix: true
        fspart_allow_reboot: false
```

## Design notes

- This role expects privilege escalation to be set at the play level (`become: true`) when needed.
- The role does not force `become` inside its task files, which keeps it compatible with Molecule container scenarios.
- `raw` is not used in the modernized task flow because this role is not a pre-Python bootstrap role.
- `fs_disable_automount` stops and masks `autofs` only when the service exists.

## Molecule testing

The repository currently contains role-specific scenario files under `molecule/default/` that you asked to keep untouched.
A modern shared verification playbook pattern is included as `molecule/shared/verify.yml` for consistency with newer roles.
When you are ready to standardize scenario wiring, point the scenario verify playbooks to the shared file.

## Release workflow

- `templates/meta_main.yml.j2` is the metadata source of truth.
- `meta/main.yml` should be regenerated from that template as part of the release/update workflow.
- If you later allow updates under `scripts/`, reuse the same metadata/render helper pattern used in your bootstrap role rather than creating a role-specific variant.

## Repository structure

```text
.
├── defaults/
├── files/
├── handlers/
├── meta/
├── molecule/
│   ├── default/
│   └── shared/
├── tasks/
├── templates/
└── vars/
```

## License

MIT

## Author

Carlos Guidugli
