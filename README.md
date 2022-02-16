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
# "containerd" is distributed in two flavors. Possible values:
#
# - base
# - k8s
#
# First difference is the download size:
# Choosing "base" will download around 35 MByte of data during installation.
# Choosing "k8s" will download around 100 MByte of data during installation.
#
# If you choose "base" the role will only install a minimal set of
# "containerd" binaries:
#
# containerd, containerd-shim, containerd-shim-runc-v1, containerd-shim-runc-v2
# and ctr.
#
# Please note: Specifying "base" also means that no "runc" will be installed.
# "runc" needs to be available before this role gets installed. So make sure
# that "runc" is already installed if you don't want to use the "k8s" flavor.
# To install "runc" separately you can also use this Ansible role:
#
# - https://github.com/githubixx/ansible-role-runc
# - https://galaxy.ansible.com/githubixx/runc
#
# If you choose "k8s" this role will install a set of files specifically for
# Kubernetes. It contains all required binaries and files for using containerd
# with Kubernetes. Besides the binaries mentioned above it additionally contains:
#
# crictl, critest and runc
#
# "runc" can also be installed separately if it's already installed by a
# different process (see further down below).
#
# It also contains the CNI binaries which currently are:
#
# vlan, host-local, flannel, bridge, host-device, tuning, firewall, bandwidth,
# ipvlan, sbr, dhcp, portmap, ptp, static, macvlan, loopback
#
# So no need to install them separately. But this role also allows you to skip
# CNI installation (see further down below).
#
# Before 0.2.0+1.5.8 of this role this setting wasn't available but "base"
# basically mimics this behavior.
containerd_flavor: "base"

# containerd version to install
containerd_version: "1.6.0"

# Directory where to store "containerd" binaries
containerd_binary_directory: "/usr/local/bin"

# Location of containerd configuration file
containerd_config_directory: "/etc/containerd"

# Directory to store the archive
containerd_tmp_directory: "{{ lookup('env', 'TMPDIR') | default('/tmp',true) }}"

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

# Name of the release tarball specifically for Kubernetes
containerd_archive_k8s: "cri-containerd-cni-{{ containerd_version }}-{{ containerd_os }}-{{ containerd_arch }}.tar.gz"

# The containerd download URL (normally no need to change it)
containerd_url: "https://github.com/containerd/containerd/releases/download/v{{ containerd_version }}/{{ containerd_archive_base if containerd_flavor == 'base' else containerd_archive_k8s }}"

# containerd systemd service settings
containerd_service_settings:
  "ExecStartPre": "{{ modprobe_location }} overlay"
  "ExecStart": "{{ containerd_binary_directory }}/containerd"
  "Restart": "always"
  "RestartSec": "5"
  "Delegate": "yes"
  "KillMode": "process"
  "OOMScoreAdjust": "-999"
  "LimitNOFILE": "1048576"
  "LimitNPROC": "infinity"
  "LimitCORE": "infinity"

# Content of configuration file of containerd. The settings below are (mainly) the
# default "containerd" settings generated with the following command (needs
# "containerd" binary installed of course):
#
# containerd config default
#
# Difference to default configuration:
#
# - The configuration file contains a few role variables that will be replaced when
#   the configuration template is processed.
# - In "plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options" the
#   setting "SystemdCgroup" is set to "true" instead of "false". This is relevant for
#   Kubernetes e.g. Also see:
#   https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd-systemd)
#
containerd_config: |
  disabled_plugins = []
  imports = []
  oom_score = 0
  plugin_dir = ""
  required_plugins = []
  root = "/var/lib/containerd"
  state = "/run/containerd"
  temp = ""
  version = 2

  [cgroup]
    path = ""

  [debug]
    address = ""
    format = ""
    gid = 0
    level = ""
    uid = 0

  [grpc]
    address = "/run/containerd/containerd.sock"
    gid = 0
    max_recv_message_size = 16777216
    max_send_message_size = 16777216
    tcp_address = ""
    tcp_tls_ca = ""
    tcp_tls_cert = ""
    tcp_tls_key = ""
    uid = 0

  [metrics]
    address = ""
    grpc_histogram = false

  [plugins]

    [plugins."io.containerd.gc.v1.scheduler"]
      deletion_threshold = 0
      mutation_threshold = 100
      pause_threshold = 0.02
      schedule_delay = "0s"
      startup_delay = "100ms"

    [plugins."io.containerd.grpc.v1.cri"]
      device_ownership_from_security_context = false
      disable_apparmor = false
      disable_cgroup = false
      disable_hugetlb_controller = true
      disable_proc_mount = false
      disable_tcp_service = true
      enable_selinux = false
      enable_tls_streaming = false
      enable_unprivileged_icmp = false
      enable_unprivileged_ports = false
      ignore_image_defined_volumes = false
      max_concurrent_downloads = 3
      max_container_log_line_size = 16384
      netns_mounts_under_state_dir = false
      restrict_oom_score_adj = false
      sandbox_image = "k8s.gcr.io/pause:3.6"
      selinux_category_range = 1024
      stats_collect_period = 10
      stream_idle_timeout = "4h0m0s"
      stream_server_address = "127.0.0.1"
      stream_server_port = "0"
      systemd_cgroup = false
      tolerate_missing_hugetlb_controller = true
      unset_seccomp_profile = ""

      [plugins."io.containerd.grpc.v1.cri".cni]
        bin_dir = "{% if containerd_cni_binary_directory is defined %}{{ containerd_cni_binary_directory }}{% else %}/opt/cni/bin{% endif -%}"
        conf_dir = "{% if containerd_cni_netconfig_directory is defined %}{{ containerd_cni_netconfig_directory }}{% else %}/etc/cni/net.d{% endif %}"
        conf_template = ""
        ip_pref = ""
        max_conf_num = 1

      [plugins."io.containerd.grpc.v1.cri".containerd]
        default_runtime_name = "runc"
        disable_snapshot_annotations = true
        discard_unpacked_layers = false
        ignore_rdt_not_enabled_errors = false
        no_pivot = false
        snapshotter = "overlayfs"

        [plugins."io.containerd.grpc.v1.cri".containerd.default_runtime]
          base_runtime_spec = ""
          cni_conf_dir = ""
          cni_max_conf_num = 0
          container_annotations = []
          pod_annotations = []
          privileged_without_host_devices = false
          runtime_engine = ""
          runtime_path = ""
          runtime_root = ""
          runtime_type = ""

          [plugins."io.containerd.grpc.v1.cri".containerd.default_runtime.options]

        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]

          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
            base_runtime_spec = ""
            cni_conf_dir = ""
            cni_max_conf_num = 0
            container_annotations = []
            pod_annotations = []
            privileged_without_host_devices = false
            runtime_engine = ""
            runtime_path = ""
            runtime_root = ""
            runtime_type = "io.containerd.runc.v2"

            [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
              BinaryName = "{% if containerd_runc_binary_directory is defined %}{{ containerd_runc_binary_directory }}/runc{% endif -%}"
              CriuImagePath = ""
              CriuPath = ""
              CriuWorkPath = ""
              IoGid = 0
              IoUid = 0
              NoNewKeyring = false
              NoPivotRoot = false
              Root = ""
              ShimCgroup = ""
              SystemdCgroup = true

        [plugins."io.containerd.grpc.v1.cri".containerd.untrusted_workload_runtime]
          base_runtime_spec = ""
          cni_conf_dir = ""
          cni_max_conf_num = 0
          container_annotations = []
          pod_annotations = []
          privileged_without_host_devices = false
          runtime_engine = ""
          runtime_path = ""
          runtime_root = ""
          runtime_type = ""

          [plugins."io.containerd.grpc.v1.cri".containerd.untrusted_workload_runtime.options]

      [plugins."io.containerd.grpc.v1.cri".image_decryption]
        key_model = "node"

      [plugins."io.containerd.grpc.v1.cri".registry]
        config_path = ""

        [plugins."io.containerd.grpc.v1.cri".registry.auths]

        [plugins."io.containerd.grpc.v1.cri".registry.configs]

        [plugins."io.containerd.grpc.v1.cri".registry.headers]

        [plugins."io.containerd.grpc.v1.cri".registry.mirrors]

      [plugins."io.containerd.grpc.v1.cri".x509_key_pair_streaming]
        tls_cert_file = ""
        tls_key_file = ""

    [plugins."io.containerd.internal.v1.opt"]
      path = "/opt/containerd"

    [plugins."io.containerd.internal.v1.restart"]
      interval = "10s"

    [plugins."io.containerd.internal.v1.tracing"]
      sampling_ratio = 1.0
      service_name = "containerd"

    [plugins."io.containerd.metadata.v1.bolt"]
      content_sharing_policy = "shared"

    [plugins."io.containerd.monitor.v1.cgroups"]
      no_prometheus = false

    [plugins."io.containerd.runtime.v1.linux"]
      no_shim = false
      runtime = "runc"
      runtime_root = ""
      shim = "containerd-shim"
      shim_debug = false

    [plugins."io.containerd.runtime.v2.task"]
      platforms = ["linux/amd64"]
      sched_core = false

    [plugins."io.containerd.service.v1.diff-service"]
      default = ["walking"]

    [plugins."io.containerd.service.v1.tasks-service"]
      rdt_config_file = ""

    [plugins."io.containerd.snapshotter.v1.aufs"]
      root_path = ""

    [plugins."io.containerd.snapshotter.v1.btrfs"]
      root_path = ""

    [plugins."io.containerd.snapshotter.v1.devmapper"]
      async_remove = false
      base_image_size = ""
      discard_blocks = false
      fs_options = ""
      fs_type = ""
      pool_name = ""
      root_path = ""

    [plugins."io.containerd.snapshotter.v1.native"]
      root_path = ""

    [plugins."io.containerd.snapshotter.v1.overlayfs"]
      root_path = ""
      upperdir_label = false

    [plugins."io.containerd.snapshotter.v1.zfs"]
      root_path = ""

    [plugins."io.containerd.tracing.processor.v1.otlp"]
      endpoint = ""
      insecure = false
      protocol = ""

  [proxy_plugins]

  [stream_processors]

    [stream_processors."io.containerd.ocicrypt.decoder.v1.tar"]
      accepts = ["application/vnd.oci.image.layer.v1.tar+encrypted"]
      args = ["--decryption-keys-path", "{{ containerd_config_directory }}/ocicrypt/keys"]
      env = ["OCICRYPT_KEYPROVIDER_CONFIG={{ containerd_config_directory }}/ocicrypt/ocicrypt_keyprovider.conf"]
      path = "ctd-decoder"
      returns = "application/vnd.oci.image.layer.v1.tar"

    [stream_processors."io.containerd.ocicrypt.decoder.v1.tar.gzip"]
      accepts = ["application/vnd.oci.image.layer.v1.tar+gzip+encrypted"]
      args = ["--decryption-keys-path", "{{ containerd_config_directory }}/ocicrypt/keys"]
      env = ["OCICRYPT_KEYPROVIDER_CONFIG={{ containerd_config_directory }}/ocicrypt/ocicrypt_keyprovider.conf"]
      path = "ctd-decoder"
      returns = "application/vnd.oci.image.layer.v1.tar+gzip"

  [timeouts]
    "io.containerd.timeout.bolt.open" = "0s"
    "io.containerd.timeout.shim.cleanup" = "5s"
    "io.containerd.timeout.shim.load" = "5s"
    "io.containerd.timeout.shim.shutdown" = "3s"
    "io.containerd.timeout.task.state" = "2s"

  [ttrpc]
    address = ""
    gid = 0
    uid = 0


#################################################
# runc
#################################################

# Directory where to store the "runc" binary.
#
# As mentioned above if "containerd_flavor: base" was specified "runc" needs
# to be installed separately. Without a "runc" binary containerd won't work.
#
# Commented by default. If "runc" should be installed just remove the comment
# and adjust the value if needed. But the default should be just fine.
# As long as "containerd_runc_binary_directory" is commented all the other
# "containerd_runc_*" variables have no effect. The same is true if
# "containerd_flavor: base" is specified.
# containerd_runc_binary_directory: "/usr/local/sbin"

# Owner/group of "runc" binary. If the variables are not set
# the resulting binary will be owned by the current user.
containerd_runc_owner: "root"
containerd_runc_group: "root"

# Specifies the permissions of the "runc" binary
containerd_runc_binary_mode: "0755"


#################################################
# crictl
#################################################

# Configuration file for "crictl" (client for CRI).
#
# Commented by default. If "crictl" should be installed just remove the
# comments and adjust the values if needed. But the defaults should be just fine.
# As long as "containerd_crictl_config_(file|directory)" are commented all the other
# "containerd_crictl_config_*" variables have no effect. The same is true if
# "containerd_flavor: base" is specified.
# containerd_crictl_config_file: "crictl.yaml"
# containerd_crictl_config_directory: "/etc"

# Directory permissions for directory defined in "containerd_crictl_config_directory"
# As the default for this directory is "/etc" be careful about permissions!
# If this variable is not specified the default is "0755".
containerd_crictl_config_directory_mode: "0755"

# Owner/group of directory defined in "containerd_crictl_config_directory".
# As the default for this directory is "/etc" be careful about ownership!
# If these variables are not specified the default owner is "root" / group "root".
containerd_crictl_config_directory_owner: "root"
containerd_crictl_config_directory_group: "root"

# Owner/group of "crictl.yaml". If the variables are not set
# the resulting binary will be owned by the current user.
containerd_crictl_config_file_owner: "root"
containerd_crictl_config_file_group: "root"

# Specifies the permissions of the "crictl.yaml" file.
containerd_crictl_config_file_mode: "0644"

containerd_crictl_config_file_content: |
  runtime-endpoint: unix:///run/containerd/containerd.sock


#################################################
# CNI
#################################################

# Directory where to store the CNI binaries.
#
# Commented by default. If CNI binaries should be installed just remove the
# comment and adjust the value if needed. But the default should be just fine.
# As long as "containerd_cni_binary_directory" is commented all the other
# "containerd_cni_*" variables have no effect. The same is true if
# "containerd_flavor: base" is specified.
# containerd_cni_binary_directory: "/opt/cni/bin"

# Directory permissions for directory containing CNI binaries
containerd_cni_binary_directory_mode: "0755"

# Owner/group of CNI binaries. If the variables are not set
# the resulting binary will be owned by the current user.
containerd_cni_binary_owner: "root"
containerd_cni_binary_group: "root"

# Specifies the permissions of the "CNI" files.
containerd_cni_binary_mode: "0755"

# Name of the CNI network configuration file and the directory where this file
# should be placed.
# These variables are commented by default which means that no CNI network
# configuration file will be created. Some Kubernetes networking platforms like
# "Cilium" are creating this file on the fly while starting up on a K8s node.
# So in this case it's sufficient to only have the CNI binaries installed.
# As long as "containerd_cni_netconfig_(file|directory)" are commented all the other
# "containerd_cni_netconfig_*" variables have no effect. The same is true if
# "containerd_flavor: base" is specified.
# containerd_cni_netconfig_file: "10-containerd-net.conflist"
# containerd_cni_netconfig_directory: "/etc/cni/net.d"

# Directory permissions of network configuration directory
containerd_cni_netconfig_directory_mode: "0755"

# Owner/group of CNI network configuration file. If these variables are not set
# the file will be owned by the current user.
containerd_cni_netconfig_file_owner: "root"
containerd_cni_netconfig_file_group: "root"

# Specifies the permissions of the file specified in "containerd_cni_netconfig_file"
containerd_cni_netconfig_file_mode: "0644"

# Content of CNI network configuration file. This is only an example! Please
# adjust to your needs! If this variable is commented the configuration file
# specified in "containerd_cni_netconfig_file" won't be created.
containerd_cni_netconfig_file_content: |
  {
    "cniVersion": "0.4.0",
    "name": "containerd-net",
    "plugins": [
      {
        "type": "bridge",
        "bridge": "cni0",
        "isGateway": true,
        "ipMasq": true,
        "promiscMode": true,
        "ipam": {
          "type": "host-local",
          "ranges": [
            [{
              "subnet": "10.88.0.0/16"
            }],
            [{
              "subnet": "2001:4860:4860::/64"
            }]
          ],
          "routes": [
            { "dst": "0.0.0.0/0" },
            { "dst": "::/0" }
          ]
        }
      },
      {
        "type": "portmap",
        "capabilities": {"portMappings": true}
      }
    ]
  }
```

Dependencies
------------

[githubixx.runc](https://galaxy.ansible.com/githubixx/runc) is an optional dependency in case `containerd_flavor` is set to `base`. But in general every Ansible role that installs `runc` binary should be good enough. In case `runc` is already installed this dependency can be ignored.

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
molecule converge -s kvm
```

This will setup a few virtual machines (VM) with different supported Linux operating systems and installs `containerd` and optionally `runc`, `crictl` and the `CNI` plugins (which are needed by Kubernetes e.g.).

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
