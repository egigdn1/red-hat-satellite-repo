# set_dns_ubuntu

Configures the DNS resolver on Ubuntu hosts using **systemd-resolved** by placing a drop-in
configuration file under `/etc/systemd/resolved.conf.d/`. The main `resolved.conf` is never
modified directly, keeping the change clean and reversible.

The role is idempotent: re-running it when nothing has changed reports `ok` for every task.

## Requirements

- Ubuntu 20.04 (focal), 22.04 (jammy), or 24.04 (noble)
- `systemd-resolved` active (default on modern Ubuntu)
- Ansible ≥ 2.9

## Variables

| Variable        | Default   | Description                          |
|-----------------|-----------|--------------------------------------|
| `dns_primary`   | `1.1.1.1` | Primary DNS server IP address        |
| `dns_secondary` | `1.0.0.1` | Secondary (fallback) DNS server IP   |

Both variables are defined in `defaults/main.yml` and can be overridden at inventory,
group_vars, host_vars, or play level.

## Usage

### Via Red Hat Satellite

Assign the role to a host or host group in Satellite's Ansible integration and run a job
template against the target host(s). No extra variables are required to use the defaults.

### Ad-hoc with ansible-playbook

Create a minimal playbook (outside the `roles/` tree):

```yaml
# test_set_dns_ubuntu.yml
- hosts: ubuntu_hosts
  roles:
    - set_dns_ubuntu
```

Run it:

```bash
ansible-playbook test_set_dns_ubuntu.yml -i inventory.ini
```

Override DNS servers at runtime:

```bash
ansible-playbook test_set_dns_ubuntu.yml -i inventory.ini \
  -e dns_primary=8.8.8.8 -e dns_secondary=8.8.4.4
```

## What the role does

1. Creates `/etc/systemd/resolved.conf.d/` if it does not exist.
2. Writes `99-ansible-dns.conf` with the configured DNS servers.
3. Restarts `systemd-resolved` via a handler **only** when the config file changes.
4. Ensures the service is enabled and running.
5. Runs `resolvectl status` and prints the output so you can confirm the change visually.

## Reversibility

Remove the drop-in file to restore the previous DNS behaviour:

```bash
rm /etc/systemd/resolved.conf.d/99-ansible-dns.conf
systemctl restart systemd-resolved
```
