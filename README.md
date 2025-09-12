# Ansible Role: Redis (bernylinville.redis)

![CI](https://github.com/bernylinville/ansible-role-redis/actions/workflows/ci.yml/badge.svg?branch=master)
![Release](https://img.shields.io/github/v/release/bernylinville/ansible-role-redis?sort=semver)
![License](https://img.shields.io/badge/license-MIT%2FBSD-blue)

Installs Redis 6.2.19 from prebuilt release binaries on systemd-based Linux. The role installs binaries into `/usr/local/bin`, renders `redis.conf`, prepares data/log/pid directories, enables `vm.overcommit_memory=1`, and manages a systemd service.

中文说明请见：Redis-6.2.19-配置参数详解.md

## Table of Contents
- About the Role
- Requirements
- Supported Platforms
- Install
- Usage
- Role Variables
- Handlers
- Development & Testing
- License
- Maintainer

## About the Role
- Installs Redis 6.2.19 from GitHub release tarball
- Idempotent install to `/usr/local/bin` (redis-server, redis-cli, etc.)
- Templated `redis.conf` with common settings and overrides
- Sets sysctl `vm.overcommit_memory=1`
- Manages `redis` systemd unit
- Molecule-tested on Docker

## Requirements
- Systemd-based host (tested on openEuler 24.03)
- Network access to GitHub releases (to fetch the tarball)

## Supported Platforms
- openEuler 24.03 (CI via Molecule Docker image)
  Other distributions may work but are not covered by CI

## Install
- Via requirements.yml (recommended):
  ```yaml
  # requirements.yml
  - name: bernylinville.redis
    src: https://github.com/bernylinville/ansible-role-redis
    version: v2.2.0
  ```
  Run: `ansible-galaxy install -r requirements.yml`

- Direct from Git (specific tag):
  `ansible-galaxy install git+https://github.com/bernylinville/ansible-role-redis.git,v2.2.0`

## Usage
Example playbook:
```yaml
- hosts: all
  become: true
  roles:
    - role: bernylinville.redis
      vars:
        redis_bind_interface: 0.0.0.0
        redis_maxmemory: 4gb
        redis_appendonly: "yes"
```

## Role Variables
See all defaults in `defaults/main.yml`.

```yaml
# Version and download
redis_version: "6.2.19"
redis_roles_version: "2.2.0"
redis_repo_url: "https://github.com/bernylinville/ansible-role-redis"
redis_download_url: "{{ redis_repo_url }}/releases/download/{{ redis_roles_version }}/redis-{{ redis_version }}-{{ ansible_distribution }}-{{ ansible_distribution_version }}-{{ ansible_architecture }}.tar.gz"

# System user/group
redis_system_user: "redis"
redis_system_group: "{{ redis_system_user }}"

# Paths
redis_data_dir: "/var/lib/redis"
redis_config_file_dir: "/etc"
redis_config_file: "{{ redis_config_file_dir }}/redis.conf"
redis_pid_dir: "/var/run/redis"
redis_pid_file: "{{ redis_pid_dir }}/redis.pid"
redis_log_dir: "/var/log/redis"
redis_log_file: "{{ redis_log_dir }}/redis-server.log"
redis_max_open_files: 10240

# Network & runtime
redis_port: 6379
redis_bind_interface: 127.0.0.1
redis_unix_socket: ''
redis_timeout: 0
redis_loglevel: "notice"

# DB and persistence
redis_databases: 16
redis_save:
  - 900 1
  - 300 10
  - 60 10000
redis_rdbcompression: "yes"
redis_dbfilename: dump.rdb
redis_maxmemory: 0
redis_maxmemory_policy: "noeviction"
redis_maxmemory_samples: 5
redis_appendonly: "no"
redis_appendfsync: "everysec"

# Includes and security
redis_includes: []
redis_requirepass: ""
redis_disabled_commands: []

# Extra config appended to redis.conf
redis_extra_config: ""
```

Variables are scoped with the `redis_*` prefix.

## Handlers
- Restart Redis service (`redis`).

## Development & Testing
- Lint YAML: `yamllint .`
- Lint Ansible: `ansible-lint`
- Full test: `MOLECULE_DISTRO=openeuler2403 molecule test`
- Fast iterate: `molecule converge` → edit → `molecule converge` → `molecule destroy`
- Local run: `ansible-playbook -i 'localhost,' -c local molecule/default/converge.yml`

## License
MIT/BSD

## Maintainer
bernylinville
