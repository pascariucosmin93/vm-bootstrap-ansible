# vm-bootstrap-ansible

Ansible project for preparing a fresh Ubuntu VM for Docker-based workloads and follow-up automation.

This repo is designed as a clean bootstrap layer you can run before higher-level automation such as `k3s`, GitOps, CI runners, or self-hosted services.

## Features

The bootstrap flow handles:

- package update and distribution upgrade
- common CLI tooling for day-to-day administration
- hostname configuration
- admin user creation
- passwordless sudo setup
- optional SSH key provisioning
- Docker installation from the official Docker repository
- optional `qemu-guest-agent` installation
- optional UFW firewall enablement

## Structure

```text
.
├── .gitignore
├── ansible.cfg
├── group_vars/
├── inventory/
├── playbooks/
├── requirements.yml
└── roles/
```

## Repository layout

- `playbooks/bootstrap.yml` runs the VM preparation flow
- `playbooks/site.yml` provides a simple entry point
- `roles/common` manages package updates, timezone, firewall and guest agent
- `roles/bootstrap` manages hostname, admin user and sudoers
- `roles/docker` installs and enables Docker

## Quick Start

Install collections:

```bash
ansible-galaxy collection install -r requirements.yml
```

Edit inventory:

```ini
[bootstrap]
vm-rui ansible_host=192.168.1.50
```

Edit shared vars in `group_vars/all.yml`:

```yaml
ansible_user: ubuntu
vm_hostname: vm-rui
vm_admin_user: cosmin
vm_admin_ssh_keys:
  - ssh-ed25519 AAAA...
bootstrap_enable_ufw: true
```

Run the bootstrap playbook:

```bash
ansible-playbook playbooks/bootstrap.yml
```

Or use the aggregate entry point:

```bash
ansible-playbook playbooks/site.yml
```

## Example Outcome

After a successful run, the target VM should have:

- updated packages
- a dedicated admin user with sudo access
- passwordless sudo when enabled
- Docker installed and running
- optional firewall rules applied through UFW
- optional guest agent enabled for VM environments

## Notes

- The inventory and variables are intentionally simple so the repo is easy to demo.
- For a real environment, split variables into `group_vars` and `host_vars` per VM.
- If you use Proxmox, VMware or another virtualized environment, `qemu-guest-agent` is a practical addition for better VM management.

## Push to GitHub

```bash
git init
git remote add origin git@github.com:pascariucosmin93/vm-bootstrap-ansible.git
git add .
git commit -m "Add VM bootstrap Ansible setup"
git branch -M main
git push -u origin main
```
