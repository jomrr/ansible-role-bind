# Ansible role: BIND

![GitHub](https://img.shields.io/github/license/jomrr/ansible-role-bind) ![GitHub last commit](https://img.shields.io/github/last-commit/jomrr/ansible-role-bind) ![GitHub issues](https://img.shields.io/github/issues-raw/jomrr/ansible-role-bind)

**Ansible role for setting up the ISC BIND DNS Server.**

## Description

Install and configure ISC BIND.

## Prerequisites

This role has no special prerequisites.

### System packages (Fedora)

- `python3` (>= 3.9)

### Python (requirements.txt)

- ansible >= 2.17

## Dependencies (requirements.yml)

This role has no dependencies.

## Supported Platforms

| OS Family | Distribution | Version | Container Image |
|-----------|--------------|---------|-----------------|
| RedHat | AlmaLinux | latest | [jomrr/molecule-almalinux:latest]( https://hub.docker.com/r/jomrr/molecule-almalinux ) |
| Debian | Debian | latest | [jomrr/molecule-debian:latest]( https://hub.docker.com/r/jomrr/molecule-debian ) |
| RedHat | Fedora | latest | [jomrr/molecule-fedora:latest]( https://hub.docker.com/r/jomrr/molecule-fedora ) |
| Debian | Ubuntu | latest | [jomrr/molecule-ubuntu:latest]( https://hub.docker.com/r/jomrr/molecule-ubuntu ) |

## Role Variables

No role default variables specified, see [defaults/main.yml](defaults/main.yml).

## Example Playbook

Example playbooks(s) that show how to use this role.

## Simple example playbook

A simple default example playbook for using jomrr.bind.
```yaml
---
# name: "jomrr.bind"
# file: "playbook_bind.yml"

- name: "PLAYBOOK | bind"
  hosts: "bind_hosts"
  gather_facts: true
  roles:
    - role: "jomrr.bind"
```

## Author(s) and License

- :octocat:                 Author::    [jomrr](https://github.com/jomrr)
- :triangular_flag_on_post: Copyright:: 2025, Jonas Mauer
- :page_with_curl:          License::   [MIT](LICENSE)

## References

- [BIND 9 Documentation](https://bind9.readthedocs.io/en/latest/)

---
