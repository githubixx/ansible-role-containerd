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
# Only value "base" is currently supported
containerd_flavor: "base"

# containerd version to install
containerd_version: "1.7.7"

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

# The containerd download URL (normally no need to change it)
containerd_url: "https://github.com/containerd/containerd/releases/download/v{{ containerd_version }}/{{ containerd_archive_base }}"

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
