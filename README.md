Ansible Role: fspart
=========

An Ansible Role that configure partitions on RHEL/CentOS, Fedora and Debian/Ubuntu. This role also perform security checks on filesystems.


Requirements
------------

No requirements.

Role Variables
--------------

**Available variables are listed below, along with default values (see defaults/main.yml):**

    fs_world_writeable_fix_enabled: yes

If set to true, the role will fix any file that can be written by everyone (others).

    fs_fstrim_timer_enabled: yes

If set to true, fstrim.timer will be enabled and started

    #fspart_cryptkeys_path: /etc/cryptkeys

Path to store files storing keys for encrypted partitions

    #fspart_crypttab_entries:
    #  name: luks-test
    #  backing_device: UUID=6b244d35-a72b-1234-5678-4258d364809c
    #  password: /etc/cryptkeys/mykey
    #  opts: discard,luks

Crypttab entries

    #fspart_cryptkeys_files:
    #  - { name: mykey, src: cryptkeys/mykey }

Files to be copied to fspart_cryptkeys_path defined path

    #partitions:
    #  - name: /tmp
    #    unit_name: tmp.mount
    #    mount_options: ['mode=1777', 'strictatime', 'nodev', 'nosuid', 'noexec']
    #    validate_options: ['nodev', 'nosuid', 'noexec']
    #    autofix: yes
    #  - name: /var
    #  - name: /var/log
    #  - name: /var/log/audit
    #  - name: /var/tmp
    #    options: ['nodev', 'nosuid', 'noexec']
    #    unit_name: fstab
    #    mount_options: ['strictatime', 'nodev', 'nosuid', 'noexec']
    #    validate_options: ['nodev', 'nosuid', 'noexec']
    #    src: 'tmpfs'
    #    fstype: tmpfs
    #    autofix: yes
    #  - name: /home
    #    options: ['nodev']
    #  - name: /dev/shm
    #    options: ['nodev', 'nosuid', 'noexec']
    #  - name: /tmp_dir
    #    unit_name: tmp_dir.mount
    #    mount_options: ['mode=1777', 'strictatime', 'nodev', 'nosuid', 'noexec']
    #    validate_options: ['nodev', 'nosuid', 'noexec']
    #    autofix: yes

List of partitions that must be present on the system and the mount options they should have.

Autofix will check and adjust configuration files so the filesystem is mounted properly. If the unit cannot be found in /etc/systemd/system, the role will attempt to copy a local file with same unit name from role's file directory, to the target system. If a local file with the unit name is found locally on role's file directory, it will try to copy it from /lib/systemd/system.

NOTE: files on /etc/systemd/system will NOT be overwritten

The mount_options is a list of all mount options that will be set on the unit mount point definition file (xxxxx.mount). The role will change the unit file to match these options.

The validate_options is a list of mount options that the mounted filesystem must have. This variable will not cause any change on any configuration, but if a partition is mounted without it, the execution will fail.

    fspart_allow_reboot: yes

If mounting/remounting fail, is reboot allowed? Sometimes partitions cannot be mounted because they are busy (have other system components using it).

**The variables listed below do not need to be changed for targeted systems (see vars/main.yml):**

    fspart_systemd_etc_path: /etc/systemd/system

This is the place to create custom units or to copy existing units from other places in order to customize it. Units found in other locations will be written to this location by this role, to prevent updates overwritting customizations.

    fspart_systemd_lib_path: /lib/systemd/system

If not already on etc path, this is the second place to look for units

    fspart_systemd_share_path: /usr/share/systemd

On Debian family systems this path has tmp.mount definition. So, this is the third path that the role will search for units.

    fspart_unit_notify: ''

This variable will be set during processing with the unit name being processed so handler can work on the correct mount unit file.

Dependencies
------------

No dependencies.

Example Playbook
----------------

    - hosts: servers
      vars:
          fs_world_writeable_fix_enabled: yes
          partitions:
            - name: /tmp
              unit_name: tmp.mount
              mount_options: ['mode=1777', 'strictatime', 'nodev', 'nosuid', 'noexec']
              validate_options: ['nodev', 'nosuid', 'noexec']
              autofix: yes
            - name: /var/tmp
              options: ['nodev', 'nosuid', 'noexec']
              unit_name: fstab
              mount_options: ['strictatime', 'nodev', 'nosuid', 'noexec']
              validate_options: ['nodev', 'nosuid', 'noexec']
              src: 'tmpfs'
              fstype: tmpfs
              autofix: yes
            - name: /tmp_dir
              unit_name: tmp_dir.mount
              mount_options: ['mode=1777', 'strictatime', 'nodev', 'nosuid', 'noexec']
              validate_options: ['nodev', 'nosuid', 'noexec']
              autofix: yes
          fspart_allow_reboot: false
      roles:
         - { guidugli.fspart }

License
-------

MIT / BSD

Author Information
------------------

This role was created in 2020 by Carlos Guidugli.
