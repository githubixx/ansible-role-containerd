<!--
Copyright (C) 2021 Robert Wimmer
SPDX-License-Identifier: GPL-3.0-or-later
-->

ansible-role-containerd
=======================

Ansible role to install [containerd](https://github.com/containerd/containerd). `containerd` is an industry-standard container runtime with an emphasis on simplicity, robustness and portability. It is available as a daemon for Linux and Windows, which can manage the complete container lifecycle of its host system: image transfer and storage, container execution and supervision, low-level storage and network attachments, etc.

Changelog
---------

see [CHANGELOG](https://github.com/githubixx/ansible-role-containerd/blob/master/CHANGELOG.md)


Role Variables
--------------

```yaml
# containerd version to install
containerd_version: "1.5.8"

# Where to install "containerd" binaries.
containerd_bin_directory: "/usr/local/bin"

# Location of containerd configuration file
containerd_conf_directory: "/etc/containerd"

# Directory to store the archive
containerd_tmp_directory: "{{ lookup('env', 'TMPDIR') | default('/tmp',true) }}"

# Owner/group of "containerd" binaries. If the variables are not set
# the resulting binary will be owned by the current user.
containerd_owner: "root"
containerd_group: "root"

# Specifies the permissions of the "containerd" binaries
containerd_binary_mode: "0755"

# Operarting system
# Possible options: "linux", "windows"
containerd_os: "linux"

# Processor architecture "containerd" should run on.
# Other possible values: "386","arm64","arm"
containerd_arch: "amd64"

# Name of the archive file name
containerd_archive: "containerd-{{ containerd_version }}-{{ containerd_os }}-{{ containerd_arch }}.tar.gz"

# The containerd download URL (normally no need to change it)
containerd_url: "https://github.com/containerd/containerd/releases/download/v{{ containerd_version }}/{{ containerd_archive }}"

# containerd systemd service settings
containerd_service_settings:
  "ExecStartPre": "{{ modprobe_location }} overlay"
  "ExecStart": "{{ containerd_bin_directory }}/containerd"
  "Restart": "always"
  "RestartSec": "5"
  "Delegate": "yes"
  "KillMode": "process"
  "OOMScoreAdjust": "-999"
  "LimitNOFILE": "1048576"
  "LimitNPROC": "infinity"
  "LimitCORE": "infinity"

# containerd configuration
containerd_config: |
  [plugins]
    [plugins.cri.containerd]
      snapshotter = "overlayfs"
      [plugins.cri.containerd.default_runtime]
        runtime_type = "io.containerd.runtime.v1.linux"
        runtime_engine = "/usr/local/bin/runc"
        runtime_root = ""
```

Dependencies
------------

[githubixx.runc](https://galaxy.ansible.com/githubixx/runc)

Example Playbook
----------------

```yaml
- hosts: your-host
  roles:
    - githubixx.containerd
```

Testing
-------

This role has a small test setup that is created using [Molecule](https://github.com/ansible-community/molecule), libvirt (vagrant-libvirt) and QEMU/KVM. Please see my blog post [Testing Ansible roles with Molecule, libvirt (vagrant-libvirt) and QEMU/KVM](https://www.tauceti.blog/posts/testing-ansible-roles-with-molecule-libvirt-vagrant-qemu-kvm/) how to setup. The test configuration is [here](https://github.com/githubixx/ansible-role-containerd/tree/master/molecule/kvm).

Afterwards molecule can be executed:

```bash
molecule converge -s kvm
```

This will setup a few virtual machines (VM) with different supported Linux operating systems and installs `runc` plus `containerd`.

To clean up run

```bash
molecule destroy -s kvm
```

License
-------

GNU GENERAL PUBLIC LICENSE Version 3

Author Information
------------------

[http://www.tauceti.blog](http://www.tauceti.blog)
