# Ansible Laravel Deployment Playbooks

This repository contains Ansible playbooks and roles to provision a Laravel PHP stack with
MariaDB Galera, Nginx, PHP-FPM, and supporting firewall configuration. The main entry point
is `playbook.yml`, which wires together the roles for database clustering and the web
application stack. The codebase is structured as a classic Ansible repo with inventories,
group variables, and reusable roles.

## Repository Layout

```
.
├── playbook.yml
├── inventories/
│   └── production/hosts
├── group_vars/
│   └── all
└── roles/
    ├── firewalld/
    ├── laravel/
    ├── mariadb_galera/
    ├── mariadb_gmaster/
    ├── mariadb_gnode/
    ├── nginx/
    └── php/
```

## Inventory and Variables

- **Inventory**: `inventories/production/hosts` defines groups for `dbserver`, `webserver`,
  and `php` hosts. The current example uses the same MariaDB nodes for all groups, but you
  can split them as needed.
- **Group Variables**: `group_vars/all` holds shared variables like database root passwords
  and Galera node IPs (`ghost1`, `ghost2`, `ghost3`).

## Playbook Flow

`playbook.yml` runs the following sequence:

1. **firewalld** on all hosts
2. **mariadb_gmaster** on the `galera_master` group
3. **mariadb_gnode** on the `galera_node` group
4. **nginx**, **php**, **laravel** on all hosts

## Roles Overview

- **firewalld**: Renders and executes a firewall configuration script.
- **mariadb_gmaster**: Installs MariaDB + Galera packages, configures Galera, and bootstraps
  the first cluster node via `galera_new_cluster`.
- **mariadb_gnode**: Installs MariaDB + Galera packages and joins additional nodes to the
  cluster.
- **mariadb_galera**: A standalone Galera cluster setup that configures all nodes based on
  static hostnames (`server-1`, `server-2`, `server-3`).
- **nginx**: Installs Nginx, creates `sites-available`/`sites-enabled`, and deploys the
  default virtual host template.
- **php**: Installs PHP 7.3 from the Remi repository and configures PHP-FPM.
- **laravel**: Installs Laravel with Composer into `/var/www/laravel` and restarts Nginx and
  PHP-FPM.

## Usage

1. Update `inventories/production/hosts` with your real hostnames or IPs.
2. Update `group_vars/all` with secure passwords and the correct Galera node IPs.
3. Run the playbook:

```bash
ansible-playbook -i inventories/production/hosts playbook.yml
```

## Notes

- The repository currently focuses on provisioning the stack. If you need SSL distribution,
  GitLab-based deploys, or backups, you can extend this structure with new roles or playbooks.
