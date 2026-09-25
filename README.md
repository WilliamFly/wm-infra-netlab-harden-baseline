# wm-infra-netlab-harden-baseline

CIS-leaning baseline hardening role for Ubuntu servers (SSH lockdown,
ufw firewall, fail2ban, unattended-upgrades, user management).

A standalone, reusable Ansible role — deliberately generic, with no
project-specific behavior. Used as a dependency by other roles (e.g. the
`router` role in
[wm-infra-netlab-network-foundation](https://github.com/<your-username>/wm-infra-netlab-network-foundation))
rather than copied into each consuming repo.

## What it does

| Area | Behavior |
|---|---|
| Users | Creates an admin user, installs SSH keys, configures sudo |
| SSH | Drops in a hardened `sshd_config.d/50-harden.conf` (no root login, no password auth, capped auth tries) |
| Firewall | Installs and enables `ufw`, default-deny incoming / allow outgoing, opens only configured ports |
| Updates | Enables `unattended-upgrades`, configurable auto-reboot policy |
| Brute-force protection | Installs and configures `fail2ban` for sshd |

Every behavior is controlled by variables in `defaults/main.yml` —
override in the consuming playbook/group_vars/host_vars, never edit
`tasks/` directly.

## Using this role from another repo

Add to that repo's `ansible/requirements.yml`:

```yaml
roles:
  - src: https://github.com/<your-username>/wm-infra-netlab-harden-baseline
    name: harden-baseline
```

Then:
```bash
ansible-galaxy install -r requirements.yml
```

This installs it into that repo's local `roles/` directory (gitignored
there — it's fetched, not committed) at whatever the `src` currently
points to. To pin a specific version instead of always tracking the
latest commit, add `version: <git-tag-or-branch>` to the entry above,
once this role has tagged releases.

## Required collections

Consuming playbooks need:
```yaml
collections:
  - name: community.general   # ufw module
  - name: ansible.posix       # authorized_key module
```

## Requirements

- Ansible >= 2.15
- Ubuntu 24.04 (noble) — other versions untested
