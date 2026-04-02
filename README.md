# vm-bootstrap-ansible

Ansible project for bootstrapping fresh Ubuntu VMs — sets up system packages, admin user, security hardening, and Docker before any higher-level automation (Kubernetes, GitOps, CI runners) takes over.

## Roles

| Role | Responsibility |
|------|---------------|
| `common` | Package updates, timezone, qemu-guest-agent, UFW |
| `bootstrap` | Hostname, admin user, SSH keys, passwordless sudo |
| `hardening` | SSH config, fail2ban, unattended-upgrades, sysctl, auditd |
| `docker` | Docker CE installation from official repository |

Roles run in order: `common → bootstrap → hardening → docker`

## Requirements

- Ansible 2.14+
- Target: Ubuntu 22.04 / 24.04
- SSH access to target VM with a user that has sudo

Install required collections:

```bash
ansible-galaxy collection install -r requirements.yml
```

## Usage

**1. Add your VM to the inventory:**

```ini
# inventory/hosts.ini
[bootstrap]
my-vm ansible_host=192.168.1.50
```

**2. Configure variables in `group_vars/all.yml`:**

```yaml
ansible_user: ubuntu
ansible_ssh_private_key_file: ~/.ssh/id_ed25519

bootstrap_hostname: my-vm
bootstrap_admin_user: cosmin
bootstrap_admin_ssh_keys:
  - ssh-ed25519 AAAA...

common_timezone: Europe/Bucharest
common_enable_qemu_guest_agent: true
```

**3. Run:**

```bash
ansible-playbook playbooks/bootstrap.yml
```

## Variables

### common

| Variable | Default | Description |
|----------|---------|-------------|
| `common_packages` | `[]` | Extra packages to install |
| `common_upgrade_packages` | `true` | Run dist-upgrade |
| `common_reboot_after_upgrade` | `false` | Reboot after upgrade |
| `common_timezone` | `UTC` | System timezone |
| `common_enable_qemu_guest_agent` | `false` | Install qemu-guest-agent |
| `common_enable_ufw` | `false` | Enable UFW firewall |
| `common_ufw_allowed_tcp_ports` | `[22]` | TCP ports to allow through UFW |

### bootstrap

| Variable | Default | Description |
|----------|---------|-------------|
| `bootstrap_hostname` | `""` | Hostname to set (skipped if empty) |
| `bootstrap_admin_user` | `devops` | Admin user to create |
| `bootstrap_admin_groups` | `[sudo]` | Groups for admin user |
| `bootstrap_admin_shell` | `/bin/bash` | Shell for admin user |
| `bootstrap_admin_ssh_keys` | `[]` | SSH public keys to authorize |
| `bootstrap_enable_passwordless_sudo` | `true` | Enable passwordless sudo |

### hardening

| Variable | Default | Description |
|----------|---------|-------------|
| `hardening_ssh_port` | `22` | SSH port |
| `hardening_ssh_permit_root_login` | `no` | Allow root SSH login |
| `hardening_ssh_password_authentication` | `no` | Allow password auth |
| `hardening_ssh_max_auth_tries` | `3` | Max SSH auth attempts |
| `hardening_fail2ban_ssh_maxretry` | `5` | fail2ban max retries |
| `hardening_fail2ban_ssh_bantime` | `3600` | fail2ban ban duration (seconds) |
| `hardening_enable_unattended_upgrades` | `true` | Enable auto security updates |
| `hardening_unattended_reboot` | `false` | Allow automatic reboot |
| `hardening_sysctl_settings` | see defaults | Kernel/network hardening params |

### docker

| Variable | Default | Description |
|----------|---------|-------------|
| `docker_users` | `[]` | Users to add to the docker group |

## CI

Pull requests run `ansible-lint` and syntax check via GitHub Actions.
