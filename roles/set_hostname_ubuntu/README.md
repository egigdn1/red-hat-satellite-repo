# set_hostname_ubuntu

Ansible role that sets the system hostname on Ubuntu servers using
`hostnamectl` (via the `ansible.builtin.hostname` module) and keeps the
`/etc/hosts` loopback mapping in sync.

## Requirements

- Ubuntu (focal / jammy / noble)
- `systemd` (default on supported Ubuntu releases)
- Privilege escalation (`become: true`) — handled inside the role

## Role Variables

Defined in `defaults/main.yml` so they can be overridden from inventory,
group/host vars, or Red Hat Satellite host parameters.

| Variable | Default | Description |
|---|---|---|
| `set_hostname_name` | `ubuntu-server` | Desired hostname to apply. Override per host. |
| `set_hostname_manage_hosts` | `true` | Whether to manage the `/etc/hosts` loopback mapping. |
| `set_hostname_hosts_ip` | `127.0.1.1` | IP used for the `/etc/hosts` mapping line. |

## What it does

1. Sets the hostname with `hostnamectl` (persistent across reboots).
2. Updates `/etc/hosts` so the loopback line maps to the new hostname.
3. Runs `hostnamectl status` and prints it for verification.

## Example Playbook

```yaml
- name: Configure hostname on Ubuntu hosts
  hosts: ubuntu
  become: true
  roles:
    - role: set_hostname_ubuntu
      vars:
        set_hostname_name: "web01.example.local"
```

## Usage with Red Hat Satellite

This role lives under `roles/` so it can be imported into a Satellite
Capsule. Set `set_hostname_name` per host through Satellite host parameters
to give each provisioned Ubuntu server its correct hostname.

## License

MIT
