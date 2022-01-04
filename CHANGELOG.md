# Changelog

## 0.2.0+1.5.8

- add support for Kubernetes by allowing to optionally also install `runc`, `crtcl` and `CNI` plugins. `containerd` builds are available in two flavors: Either just the `containerd` binaries or `containerd` binaries plus everything else needed to use `containerd` together with Kubernetes. Please see [defaults/main.yml](https://github.com/githubixx/ansible-role-containerd/tree/master/defaults/main.yml) for all possible settings. With the Kubernetes support a lot of new variables were introduced but for almost all the default values should be good enough.

## 0.1.1+1.5.8

- update `containerd` to `v1.5.8`

## 0.1.0+1.5.7

- update `containerd` to `v1.5.7`

## 0.1.0+1.5.5

- initial commit
