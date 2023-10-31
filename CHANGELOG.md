# Changelog

## 0.11.0+1.7.8

- update `containerd` to `v1.7.8`

## 0.10.0+1.7.3

- update `containerd` to `v1.7.3`
- add support for Ubuntu 22.04
- **BREAKING**: remove support for Ubuntu 18.04 (reached end of life)
- **BREAKING**: When `containerd` or `CNI` binaries are upgraded `containerd.service` will be restarted. Up until this version this only happened when `containerd` configuration was updated.

## 0.9.0+1.7.0

- update `containerd` to `v1.7.0`
- add Molecule `verify` step

## 0.8.0+1.6.19

- update `containerd` to `v1.6.19`
- fix Molecule `converge.yml`

## 0.7.0+1.6.14

- add Github release action to push new release to Ansible Galaxy

## 0.6.2+1.6.14

- update `containerd` to `v1.6.14`

## 0.6.1+1.6.9

- update `containerd` to `v1.6.9`

## 0.6.0+1.6.8

- fix file permissions in task `Downloading containerd archive`
- ansible-lint: fix issues
- `min_ansible_version` in `meta/main.yml` value should be string
- add `.yamllint`

## 0.5.5+1.6.8

- update `containerd` to `v1.6.8`

## 0.5.4+1.6.6

- update `containerd` to `v1.6.6`

## 0.5.3+1.6.4

- update `containerd` to `v1.6.4`

## 0.5.2+1.6.3

- update `containerd` to `v1.6.3`

## 0.5.1+1.6.2

- update `containerd` to `v1.6.2`

## 0.5.0+1.6.0

- update `containerd` to `v1.6.0`
- update `containerd_config` variable value for containerd `v.1.6.0`

## 0.4.0+1.5.9

- The `BinaryName` value in `container_config` variable will now use the value of `containerd_runc_binary_directory` if defined by default. (contribution by @tiagoblackcode)
- Ansible's `first_found` lookup plugin was used to determine the location of `modprobe` binary. But since all Ansible lookup plugins are only executed on the controller host the modprobe path of the controller host was used in `containerd.service` file. That's fixed now. (contribution by @tiagoblackcode)

## 0.3.0+1.5.9

- In `plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options` the setting `SystemdCgroup` is now set to `true` instead of `false`. This is relevant for Kubernetes e.g. Also see: [Kubernetes container-runtimes - containerd](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd-systemd)

## 0.2.1+1.5.9

- update `containerd` to `v1.5.9`

## 0.2.0+1.5.8

- add support for Kubernetes by allowing to optionally also install `runc`, `crtcl` and `CNI` plugins. `containerd` builds are available in two flavors: Either just the `containerd` binaries or `containerd` binaries plus everything else needed to use `containerd` together with Kubernetes. Please see [defaults/main.yml](https://github.com/githubixx/ansible-role-containerd/tree/master/defaults/main.yml) for all possible settings. With the Kubernetes support a lot of new variables were introduced but for almost all the default values should be good enough. The default `containerd_flavor: "base"` mimics the behavior of the previous version of this role. So upgrading shouldn't be an issue.

## 0.1.1+1.5.8

- update `containerd` to `v1.5.8`

## 0.1.0+1.5.7

- update `containerd` to `v1.5.7`

## 0.1.0+1.5.5

- initial commit
