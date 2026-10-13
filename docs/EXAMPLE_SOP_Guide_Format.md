<h1 align="center">Example: Implementation Guide in SOP Format</h1>

> This is a format example for Sani, adapted from Joy's Kubernetes deployment
> guide and rewritten to the bmw SOP `implementation.md` structure. The subject
> (a single-node Kubernetes cluster) is just the vehicle. Copy the shape, not the
> content. Real lab IPs and hostnames are replaced with placeholders.
>
> SOP rules this example shows: one command one block with its output commented in
> the same block (rule 1), a `# run from:` line on every block (rule 2), a
> Prerequisites and Project Root declared before any step, a System Architecture
> diagram, Post-Installation Verification, and Known Issues separate from
> Troubleshooting. SOP source: `implementation.md`, `readme-guide.md`.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Project Root](#project-root)
- [Project Description](#project-description)
- [System Architecture](#system-architecture)
- [Access Method](#access-method)
- [Installation Steps](#installation-steps)
- [Post-Installation Verification](#post-installation-verification)
- [Known Issues](#known-issues)
- [Troubleshooting](#troubleshooting)
- [Additional Resources](#additional-resources)

## Prerequisites

Everything that must exist before setup. Secret values stay out of Git, list pointers only.

| Input | What it is | Example |
| --- | --- | --- |
| Hardware | the test VM or server | 4 vCPU, 16 GB RAM, 1 NIC on the lab LAN |
| Access | SSH reachability | `ssh <user>@<vm-ip>` via lab VPN |
| Software | OS and versions | Ubuntu 22.04, containerd 2.x |
| Credentials | pointers only, never values | `secret/lab/<project>/kubeconfig` (Vault) |

## Project Root

| Item | Value |
| --- | --- |
| SSH entry | `ssh <user>@<vm-ip>` |
| Clone command | `git clone https://github.com/<org>/<repo>.git` |
| Project root (PATH) | `~/<repo>` , every step runs here unless its `# run from:` says otherwise |

## Project Description

**Project Name:** Single-node Kubernetes cluster (O-Cloud role).

**Description:** Brings up a one-node kubeadm cluster so the O-RAN services can run on it.

**Key Features:**
- kubeadm cluster, control plane and worker on one node
- Calico pod network
- a verifiable NodePort service

**Target Users:** lab interns bringing up the testbed.

## System Architecture

```
 laptop ──── lab LAN <lab-subnet> ────┐
                                      │ <nic> <vm-ip>
 ┌──────────────── VM <node> ─────────┴──────────────────────┐
 │ kubeadm cluster (API :6443)                               │
 │   control plane + worker on one node                      │
 │   pods: Calico 10.244.0.0/16                              │
 │   nginx NodePort :30080                                   │
 └───────────────────────────────────────────────────────────┘
```

Include in a real diagram: IP addresses per component, protocols and ports, data-flow direction.

## Access Method

```shell
# run from: your laptop
ssh <user>@<vm-ip>

# The authenticity of host '<vm-ip>' can't be established.
# Are you sure you want to continue connecting (yes/no)? yes
# Welcome to Ubuntu 22.04.4 LTS
# <user>@<node>:~$
```

## Installation Steps

### Step 1: Check the VM

Confirm the NIC, that swap is off, and that containerd is present before anything else.

```shell
# run from: ~/<repo>   (project root)
ip -brief addr
swapon --show
containerd --version

# <nic>  UP  <vm-ip>/24
# (swapon prints nothing = swap is off)
# containerd containerd.io 2.x
```

What this proves: the NIC is up on the lab LAN, swap is off (kubeadm requires it), containerd is installed.

### Step 2: Enable the CRI plugin in containerd

Kubernetes talks to containerd over the CRI plugin, which is disabled by default.

```shell
# run from: ~/<repo>
sudo sed -i 's/disabled_plugins = \["cri"\]/disabled_plugins = []/' /etc/containerd/config.toml
sudo systemctl restart containerd
systemctl is-active containerd

# active
```

What this proves: containerd restarted with CRI on, so kubelet can create pods.

### Step 3: Create the cluster

Initialise the control plane and pin the node IP.

```shell
# run from: ~/<repo>
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=<vm-ip>

# [init] Using Kubernetes version: v1.29.15
# ...
# Your Kubernetes control-plane has initialized successfully!
```

What this proves: the API server is up on `<vm-ip>:6443`.

> [!WARNING]
> `kubeadm init` prints a `kubeadm join ... --token ...` line. Never paste it into
> chat or commit it. That token lets any machine on the LAN join the cluster.
> Delete it after init if the cluster will stay single-node.

## Post-Installation Verification

One command that proves the testbed is up.

```shell
# run from: ~/<repo>
kubectl get nodes

# NAME     STATUS   ROLES           AGE   VERSION
# <node>   Ready    control-plane   3m    v1.29.15
```

Pass when the node reads `Ready`.

## Known Issues

| # | Behaviour | Handling |
| --- | --- | --- |
| 1 | Ubuntu 22.04 shows a pink `needrestart` dialog during `apt-get install` | run installs with `sudo NEEDRESTART_MODE=a apt-get install -y ...` to skip it |
| 2 | kind or kubeadm node reads `NotReady` for the first seconds | the pod network installs last; wait for it, do not read the node once |

## Troubleshooting

### Problem 1: Node stays NotReady after create

**Symptom.** `kubectl get nodes` shows `NotReady` right after cluster creation.

**Cause.** The pod network (Calico or kindnet) is the last thing to start; the node is not Ready until it runs.

**Fix.**

```shell
# run from: ~/<repo>
kubectl wait --for=condition=Ready node/<node> --timeout=120s

# node/<node> condition met
```

## Additional Resources

- SOP format spec: `implementation.md` (the five writing rules and the section list).
- SOP user-guide spec: `readme-guide.md`.
- A fuller real guide (private): Joy's `O2-project/vm-setup/k8s-deployment-guide.md`.
