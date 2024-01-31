<!--
Copyright (C) 2021-2023 Robert Wimmer
SPDX-License-Identifier: GPL-3.0-or-later
-->

ansible-role-containerd
=======================

Ansible role to install [containerd](https://github.com/containerd/containerd). `containerd` is an industry-standard container runtime with an emphasis on simplicity, robustness and portability. It is available as a daemon for Linux and Windows, which can manage the complete container lifecycle of its host system: image transfer and storage, container execution and supervision, low-level storage and network attachments, etc.

Changelog
---------

**Change history:**

See full [CHANGELOG](https://github.com/githubixx/ansible-role-containerd/blob/master/CHANGELOG.md)

**Recent changes:**

## 0.12.5+1.7.13

- update `containerd` to `v1.7.13`

0.12.4+1.7.12

- add task that creates directory specified in containerd_tmp_directory
- fix typo in defaults/main.yml
- Molecule: Change IP addresses
- Molecule: add two new host variables in `molecule/default/molecule.yml`

0.12.3+1.7.12

NOTE: If you upgrade from a version <= `0.11.0+1.7.8` please read [CHANGELOG](https://github.com/githubixx/ansible-role-containerd/blob/master/CHANGELOG.md) for that version carefully because of breaking changes!

- update `containerd` to `v1.7.12`
- adjust Github action because of Ansible Galaxy changes

0.12.2+1.7.10

- update `containerd` to `v1.7.10`

Installation
------------

- Directly download from Github (change into Ansible role directory before cloning):
`git clone https://github.com/githubixx/ansible-role-containerd.git githubixx.containerd`

- Via `ansible-galaxy` command and download directly from Ansible Galaxy:
`ansible-galaxy install role githubixx.containerd`

- Create a `requirements.yml` file with the following content (this will download the role from Github) and install with
`ansible-galaxy role install -r requirements.yml`:

```yaml
---
roles:
  - name: githubixx.containerd
    src: https://github.com/githubixx/ansible-role-containerd.git
    version: 0.12.4+1.7.13
```

Role Variables
--------------

```yaml
# Only value "base" is currently supported
containerd_flavor: "base"

# containerd version to install
containerd_version: "1.7.13"

# Directory where to store "containerd" binaries
containerd_binary_directory: "/usr/local/bin"

# Location of containerd configuration file
containerd_config_directory: "/etc/containerd"

# Directory to store the archive
containerd_tmp_directory: "{{ lookup('env', 'TMPDIR') | default('/tmp', true) }}"

# Owner/group of "containerd" binaries. If the variables are not set
# the resulting binary will be owned by the current user.
containerd_owner: "root"
containerd_group: "root"

# Specifies the permissions of the "containerd" binaries
containerd_binary_mode: "0755"

# Operating system
# Possible options: "linux", "windows"
containerd_os: "linux"

# Processor architecture "containerd" should run on.
# Other possible values: "arm64","arm"
containerd_arch: "amd64"

# Name of the archive file name
containerd_archive_base: "containerd-{{ containerd_version }}-{{ containerd_os }}-{{ containerd_arch }}.tar.gz"

# The containerd download URL (normally no need to change it)
containerd_url: "https://github.com/containerd/containerd/releases/download/v{{ containerd_version }}/{{ containerd_archive_base }}"

# containerd systemd service settings
containerd_service_settings:
  "ExecStartPre": "{{ modprobe_location }} overlay"
  "ExecStart": "{{ containerd_binary_directory }}/containerd"
  "Restart": "always"
  "RestartSec": "5"
  "Type": "notify"
  "Delegate": "yes"
  "KillMode": "process"
  "OOMScoreAdjust": "-999"
  "LimitNOFILE": "1048576"
  "LimitNPROC": "infinity"
  "LimitCORE": "infinity"

# Content of configuration file of "containerd". The settings below are the
# settings that are different to the default "containerd" settings.
#
# The default "containerd" configuration can be generated with this command:
#
# containerd config default
#
# Difference to default configuration:
#
# - The configuration file contains a few role variables that will be replaced when
#   the configuration template is processed.
# - In 'plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options' the
#   setting "SystemdCgroup" is set to "true" instead of "false". This is relevant for
#   Kubernetes e.g. Also see:
#   https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd-systemd)
#
containerd_config: |
  version = 2
  [plugins]
    [plugins."io.containerd.grpc.v1.cri"]
      sandbox_image = "registry.k8s.io/pause:3.8"
      [plugins."io.containerd.grpc.v1.cri".cni]
        bin_dir = "/opt/cni/bin"
        conf_dir = "/etc/cni/net.d"
      [plugins."io.containerd.grpc.v1.cri".containerd]
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
            runtime_type = "io.containerd.runc.v2"
            [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
              BinaryName = "/usr/local/sbin/runc"
              SystemdCgroup = true
  [stream_processors]
    [stream_processors."io.containerd.ocicrypt.decoder.v1.tar"]
      args = ["--decryption-keys-path", "{{ containerd_config_directory }}/ocicrypt/keys"]
      env = ["OCICRYPT_KEYPROVIDER_CONFIG={{ containerd_config_directory }}/ocicrypt/ocicrypt_keyprovider.conf"]
    [stream_processors."io.containerd.ocicrypt.decoder.v1.tar.gzip"]
      args = ["--decryption-keys-path", "{{ containerd_config_directory }}/ocicrypt/keys"]
      env = ["OCICRYPT_KEYPROVIDER_CONFIG={{ containerd_config_directory }}/ocicrypt/ocicrypt_keyprovider.conf"]
```

Dependencies
------------

Optional dependencies (e.g. needed for Kubernetes):

- [runc](https://github.com/githubixx/ansible-role-runc)
- [CNI](https://github.com/githubixx/ansible-role-cni)

You can use every other `runc` and `CNI` role of course.

Example Playbook
----------------

```yaml
- hosts: your-host
  roles:
    - githubixx.containerd
```

More examples are available in the [Molecule tests](https://github.com/githubixx/ansible-role-containerd/tree/master/molecule/kvm).

Testing
-------

This role has a small test setup that is created using [Molecule](https://github.com/ansible-community/molecule), libvirt (vagrant-libvirt) and QEMU/KVM. Please see my blog post [Testing Ansible roles with Molecule, libvirt (vagrant-libvirt) and QEMU/KVM](https://www.tauceti.blog/posts/testing-ansible-roles-with-molecule-libvirt-vagrant-qemu-kvm/) how to setup. The test configuration is [here](https://github.com/githubixx/ansible-role-containerd/tree/master/molecule/kvm).

Afterwards molecule can be executed:

```bash
molecule converge
```

This will setup a few virtual machines (VM) with different supported Linux operating systems and installs `containerd`, `runc` and the `CNI` plugins (which are needed by Kubernetes e.g.).

A small verification step is also included. It pulls a nginx container and runs it to make sure that `containerd` is setup correctly and is able to run container images:

```bash
molecule verify
```

To clean up run

```bash
molecule destroy
```

License
-------

GNU GENERAL PUBLIC LICENSE Version 3

Author Information
------------------

[http://www.tauceti.blog](http://www.tauceti.blog)
