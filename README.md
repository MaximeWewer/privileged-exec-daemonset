# Privileged-Exec DaemonSet

This repository contains an example Kubernetes deployment of a **privileged-exec** DaemonSet that provides a privileged shell on every cluster node.

> **Warning**: This DaemonSet grants root-level permissions on the host (`privileged: true`, `hostPID: true`, `hostNetwork: true`, etc.). Use it only for testing or troubleshooting purposes, and **never** in production environments or at your own risk.

---

## Table of contents

1. [Purpose](#purpose)
2. [How it works](#how-it-works)
3. [Usage instructions](#usage-instructions)
4. [Cleanup](#cleanup)

---

## Purpose

Enable an administrator to access a **shell** on each Kubernetes node via a privileged pod. Useful for diagnosing node issues or installing packages directly on the host.

## How it works

- **DaemonSet**: Schedules one pod per node in the cluster.
- **Privileged**: Container runs with full Linux capabilities and root access.
- **hostPID**, **hostIPC**, **hostNetwork**: Shares the PID, IPC, and network namespaces of the host.
- **hostPath**: Mounts the host’s root filesystem (`/`) into the pod at `/noderoot`.
- From the BusyBox container, you can `chroot` into `/noderoot` and launch Bash.

## Usage instructions

1. **Clone the repository**:

   ```sh
   git clone git@github.com:MaximeWewer/privileged-exec-daemonset.git
   cd privileged-exec-daemonset
   ```

2. **Apply the namespace and DaemonSet**:

   ```sh
   kubectl apply -f privileged-exec-daemonset.yaml
   ```

3. **Open a shell on a node**:

   - List the pods:

     ```sh
     kubectl -n privileged-exec-daemonset get pods
     ```

   - Execute a host shell:

     ```sh
     kubectl -n privileged-exec-daemonset exec -it privileged-exec-<pod_id> -- /bin/sh -c 'chroot /noderoot /bin/bash -c "YOUR COMMAND"'
     ```

4. **Install a package (Debian/Ubuntu example)**:

   Inside the chrooted shell:

   ```sh
   kubectl -n privileged-exec-daemonset exec -it privileged-exec-<pod_id> -- /bin/sh -c 'chroot /noderoot /bin/bash -c "apt update && apt install -y <package_name>"'
   ```

## Cleanup

To remove the DaemonSet and its namespace:

```sh
kubectl delete namespace privileged-exec-daemonset
```
