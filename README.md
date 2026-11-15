# Kubernetes Node Provisioning with Ansible

An Ansible-based automation project for provisioning and configuring Linux servers as Kubernetes nodes.

The project automates the preparation of fresh machines, including system configuration, networking, firewall rules, container runtimes, Kubernetes installation, CA certificates, and developer tools.

The goal is to have a reproducible, idempotent, and maintainable infrastructure configuration that can be applied to both Kubernetes control-plane and worker nodes.

## Features

The project currently aims to automate the following components:

### 1. System preparation
- Update and upgrade system packages.
- Install essential system utilities.
- Configure hostname and timezone.
- Configure kernel modules.
- Configure kernel parameters using sysctl.
- Configure IP forwarding.
- Configure swap.
- Prepare systemd and other required services.

### 2. Firewall and networking
- Configure iptables.
- Configure IPv4 and IPv6 firewall policies.
- Persist firewall rules across reboots.
- Configure Kubernetes networking requirements.
- Configure forwarding and bridge networking.
- Allow required Kubernetes communication ports.

### 3. Container Runtime (containerd)

Install and configure [containerd](https://containerd.io/) as the container runtime for Kubernetes nodes.

- Install containerd.
- Configure containerd using `config.toml`.
- Enable and configure the systemd cgroup driver.
- Configure the CRI (Container Runtime Interface) plugin.
- Configure the container image registry settings.
- Configure containerd systemd services.
- Enable and start containerd.
- Verify container runtime health.
- Configure required kernel modules and networking prerequisites.

Containerd is used as the Kubernetes container runtime through the Container Runtime Interface (CRI).

The configuration is designed to work with kubelet and kubeadm and follows Kubernetes container runtime requirements.

### 4. Kubernetes
- Install Kubernetes packages.
- Configure kubelet.
- Install kubeadm and kubectl.
- Configure the container runtime.
- Configure Kubernetes kernel and networking prerequisites.
- Initialize the control-plane node.
- Join worker nodes to the cluster.
- Configure node-specific settings.

The Kubernetes installation should support separate configurations for control-plane and worker nodes.

### 5. CA certificates
- Install trusted CA certificates.
- Configure the operating system certificate store.
- Support custom/private CA certificates.
- Update the system certificate bundle.
- Make trusted certificates available to applications and container runtimes where necessary.

### 6. Neovim
- Install Neovim.
- Deploy a custom Neovim configuration.
- Install the provided `init.lua` configuration.
- Deploy custom binaries and scripts.
- Configure user-specific Neovim settings.

The Neovim configuration and binaries are maintained as project files and deployed to the appropriate user's home directory.

---

## Architecture

The project uses Ansible roles to separate responsibilities and make individual components reusable.

Suggested project structure:

```text
.
├── ansible.cfg
├── inventory/
│   ├── hosts.ini
│   └── group_vars/
│       ├── all.yml
│       ├── masters.yml
│       └── workers.yml
│
├── playbooks/
│   ├── site.yml
│   ├── prepare.yml
│   ├── masters.yml
│   └── workers.yml
│
├── roles/
│   ├── common/
│   ├── kernel/
│   ├── iptables/
│   ├── container_runtime/
│   ├── kubernetes/
│   ├── kubernetes_control_plane/
│   ├── kubernetes_worker/
│   ├── ca_certificates/
│   └── neovim/
│
├── files/
│   └── neovim/
│       ├── config/
│       ├── bin/
│       └── share/
│
├── templates/
│
├── vars/
│
├── requirements.yml
├── .ansible-lint
├── .gitignore
└── README.md
```

## Requirements

### Control machine

- Ansible installed.
- SSH access to all managed nodes.
- SSH private key or another configured authentication method.
- Python available on managed nodes.
- Privileged access through sudo/become.

### Managed nodes

- Supported Linux distribution.
- systemd.
- SSH server.
- Internet access or access to configured package repositories.
- Sufficient disk space and memory.
- Static IP addresses or predictable network configuration.

The exact package requirements depend on the selected Linux distribution.


## Installation and Usage

### 1. Clone the repository

```bash
git clone <repository-url>
cd <repository-directory>
```

### 2. Install Ansible dependencies

```bash
ansible-galaxy install -r requirements.yml
```

### 3. Check connectivity

```bash
ansible all -i inventory/hosts.ini -m ping
```

### 4. Run system preparation

```bash
ansible-playbook \
    -i inventory/hosts.ini \
    playbooks/prepare.yml \
    --ask-become-pass
```

### 5. Configure all nodes

```bash
ansible-playbook \
    -i inventory/hosts.ini \
    playbooks/site.yml \
    --ask-become-pass
```

### 6. Configure control-plane nodes

```bash
ansible-playbook \
    -i inventory/hosts.ini \
    playbooks/masters.yml \
    --ask-become-pass
```

### 7. Configure worker nodes

```bash
ansible-playbook \
    -i inventory/hosts.ini \
    playbooks/workers.yml \
    --ask-become-pass
```

## Security

Security is an important part of the provisioning process.

The project should follow these principles:

- Prefer rootless container execution where appropriate.
- Avoid storing passwords and private keys in plain text.
- Use Ansible Vault for sensitive variables.
- Restrict SSH access.
- Apply firewall rules using an explicit allowlist.
- Use trusted CA certificates.
- Keep Kubernetes and system packages updated.
- Avoid unnecessary privileged services.
- Apply the principle of least privilege.

## Idempotency

All Ansible roles should be idempotent.

Running the same playbook multiple times should not produce unnecessary configuration changes or unexpected side effects.

Use Ansible modules instead of shell commands whenever possible.

## Testing

Before applying changes to production nodes:

```bash
ansible-playbook \
    -i inventory/hosts.ini \
    playbooks/site.yml \
    --syntax-check
```

Run in check mode:

```bash
ansible-playbook \
    -i inventory/hosts.ini \
    playbooks/site.yml \
    --check --diff
```
