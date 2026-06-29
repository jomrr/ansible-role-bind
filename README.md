# Ansible Role: bind

![GitHub](https://img.shields.io/github/license/jomrr/ansible-role-bind) ![GitHub last commit](https://img.shields.io/github/last-commit/jomrr/ansible-role-bind) ![GitHub issues](https://img.shields.io/github/issues-raw/jomrr/ansible-role-bind) [![dev](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-bind/dev.yml?branch=dev&event=push&label=dev)](https://github.com/jomrr/ansible-role-bind/actions/workflows/dev.yml?query=branch%3Adev) [![main](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-bind/main.yml?branch=main&event=push&label=main)](https://github.com/jomrr/ansible-role-bind/actions/workflows/main.yml?query=branch%3Amain)

Ansible role for setting up the ISC BIND DNS Server.

## Purpose

This role installs ISC BIND, renders BIND top-level configuration blocks,
enables and starts the BIND service, and restarts it after configuration
changes.

## Scope

### Managed

- ISC BIND packages through platform-specific package lists
- Main BIND configuration file
- TSIG key files
- ACL, primaries, controls, options, logging, include, DLZ, and zone clauses
- Managed authoritative forward and reverse zone files
- Zone-level BIND update-policy rules
- BIND service handler for configuration changes

### Not Managed

- Firewall policy
- DNSSEC key lifecycle
- Dynamic DNS update operations
- Samba AD database content for BIND_DLZ
- TSIG secret generation or rotation
- Multi-server primary/secondary orchestration

## Requirements

- Target hosts need platform repositories that provide ISC BIND packages.

## Dependencies

```yaml
collections:
  - name: community.general
    version: '>=12.0.0'
```

## Role Variables

The following variables are part of the public role interface.

| Name | Type | Required | Default | Description |
| ---- | ---- | -------- | ------- | ----------- |
| `bind_tsig_keys` | `list` | `false` | [] | TSIG key declarations included from the managed local BIND configuration.<br>Store real secrets in Ansible Vault. |
| `bind_acls` | `list` | `false` |  | Named BIND ACL declarations.<br>The default includes local and bogons ACLs.<br>The bogons ACL can be referenced from a blackhole option and excludes private, shared, and ULA ranges commonly used by local clients. |
| `bind_primaries` | `list` | `false` | [] | Named BIND primaries lists for secondary zones. |
| `bind_controls` | `list` | `false` | [] | Native BIND controls entries rendered inside a controls block. |
| `bind_tls` | `list` | `false` | [] | Top-level BIND TLS blocks referenced by DNS-over-TLS, DNS-over-HTTPS, or TLS zone transfer configuration. |
| `bind_http` | `list` | `false` | [] | Top-level BIND HTTP blocks referenced by DNS-over-HTTPS listeners. |
| `bind_listeners` | `list` | `false` |  | BIND listen-on or listen-on-v6 statements rendered inside the options block.<br>Listener entries support classic DNS, DNS-over-TLS, and DNS-over-HTTPS. |
| `bind_qname_minimization` | `str` | `false` | `strict` | QNAME minimization mode for the recursive resolver.<br>Strict mode follows the minimization algorithm without fallback to normal query mode. |
| `bind_options` | `list` | `false` |  | Ordered BIND option statements rendered after package-native platform options.<br>Entries with the same name as package-native options replace those native options.<br>Empty values omit the matching native option.<br>Use a rate-limit entry for BIND response rate limiting instead of a role-specific RRL schema. |
| `bind_logging` | `dict` | `false` |  | BIND logging configuration with channels and categories. |
| `bind_includes` | `list` | `false` | [] | Additional top-level BIND include files rendered after platform default includes. |
| `bind_dlz` | `list` | `false` | [] | Top-level BIND DLZ blocks, for example Samba BIND_DLZ integration. |
| `bind_zones` | `list` | `false` | [] | BIND zone declarations for primary, secondary, forward, RPZ, and related zones. |
| `bind_extra_statements` | `list` | `false` | [] | Additional complete top-level BIND statements for unsupported edge cases. |
| `bind_zone_files` | `list` | `false` |  | Managed authoritative forward or reverse zone files.<br>Reverse zones use the same template with PTR records. |
| `bind_zone_file_ttl` | `str` | `false` | `1h` | Default TTL for managed zone files. |
| `bind_zone_file_refresh` | `str` | `false` | `1h` | Default SOA refresh interval for managed zone files. |
| `bind_zone_file_retry` | `str` | `false` | `15m` | Default SOA retry interval for managed zone files. |
| `bind_zone_file_expire` | `str` | `false` | `1w` | Default SOA expire interval for managed zone files. |
| `bind_zone_file_minimum` | `str` | `false` | `1d` | Default SOA minimum TTL for managed zone files. |

## Managed Files

- `/etc/bind/named.conf` on Debian-family systems
- `/etc/bind/named.conf.options` on Debian-family systems
- `/etc/bind/named.conf.local` on Debian-family systems
- `/etc/named.conf` on Red Hat-family and Suse systems
- `/etc/named/named.conf.local` on Red Hat-family systems
- `/etc/named.d/local.conf` on Suse systems
- `<platform config directory>/<key-name>.key` when TSIG keys are configured
- `<platform zone directory>/<zone-file>` when bind_zone_files is configured

## Check Mode

Package and template tasks support check mode where the underlying modules support it.

## Service Behavior

The role enables and starts the BIND service. Managed configuration
changes notify the restart handler.

### Handlers

- BIND | Restart service

## Security Notes

- The default listener is limited to localhost.
- Version, hostname, and server-id disclosure are disabled by default.
- Store TSIG secrets in Ansible Vault.

## Operational Notes

- The main API follows BIND's own top-level blocks.
- Use variables such as `bind_acls`, `bind_options`, and `bind_zones`.
- Package-native path options are used by default and can be replaced by same-name entries in `bind_options`.
- Platform default includes are rendered automatically before additional `bind_includes`.
- `bind_dlz` renders top-level DLZ blocks only; all zone declarations belong in `bind_zones`.
- Managed forward and reverse zone files use absolute paths in `bind_zone_files`.
- Reverse zones use PTR records in the same zone template.
- Use explicit ACLs before widening query or recursion access.
- Use zone-level `update_policy` for granular DDNS permissions on primary zones.
- Do not combine BIND `update-policy` and `allow-update` for the same zone.
- Use RPZ zones and `response-policy` statements for DNSBL-style response filtering.
- Use a `rate-limit` entry in `bind_options` for BIND response rate limiting.
- Samba database and DLZ module lifecycle stay outside the role.

## Supported Platforms

| OS Family | Distribution | Version | Container Image |
| --------- | ------------ | ------- | --------------- |
| RedHat | AlmaLinux | latest | [jomrr/molecule-almalinux:latest](https://hub.docker.com/r/jomrr/molecule-almalinux) |
| Debian | Debian | latest | [jomrr/molecule-debian:latest](https://hub.docker.com/r/jomrr/molecule-debian) |
| RedHat | Fedora | latest | [jomrr/molecule-fedora:latest](https://hub.docker.com/r/jomrr/molecule-fedora) |
| Suse | OpenSuse Leap | latest | [jomrr/molecule-opensuse-leap:latest](https://hub.docker.com/r/jomrr/molecule-opensuse-leap) |
| Suse | OpenSuse Tumbleweed | latest | [jomrr/molecule-opensuse-tumbleweed:latest](https://hub.docker.com/r/jomrr/molecule-opensuse-tumbleweed) |
| Debian | Ubuntu | latest | [jomrr/molecule-ubuntu:latest](https://hub.docker.com/r/jomrr/molecule-ubuntu) |

## Example Playbook

### Local resolver

Apply the default localhost-only BIND configuration.

```yaml
---
- name: Configure ISC BIND
  hosts: bind
  gather_facts: true
  roles:
    - role: jomrr.bind
```
### Recursive resolver for a local network

Allow local clients to query a recursive resolver with explicit upstream forwarders.

```yaml
---
- name: Configure ISC BIND for a local network
  hosts: bind
  gather_facts: true
  roles:
    - role: jomrr.bind
      vars:
        bind_acls:
          - name: local
            entries:
              - localhost
              - 192.0.2.0/24
        bind_options:
          - name: listen-on
            arguments: port 53
            entries:
              - 127.0.0.1
              - 192.0.2.53
          - name: listen-on-v6
            arguments: port 53
            entries:
              - none
          - name: allow-query
            entries:
              - local
          - name: allow-recursion
            entries:
              - local
          - name: recursion
            value: "yes"
          - name: forward
            value: only
          - name: forwarders
            entries:
              - 1.1.1.1
              - 9.9.9.9
```
### Authoritative response rate limiting

Configure BIND response rate limiting through the native options block.

```yaml
---
- name: Configure authoritative BIND with response rate limiting
  hosts: bind
  gather_facts: true
  roles:
    - role: jomrr.bind
      vars:
        bind_tsig_keys:
          - name: ddns
            algorithm: hmac-sha256
            secret: "{{ vault_bind_ddns_secret }}"
        bind_options:
          - name: recursion
            value: "no"
          - name: rate-limit
            entries:
              - responses-per-second 5
              - referrals-per-second 5
              - nodata-per-second 5
              - nxdomains-per-second 5
              - errors-per-second 5
              - all-per-second 20
              - window 5
              - slip 2
              - qps-scale 250
```
### DNSBL with RPZ

Enable a local response policy zone for DNSBL-style filtering.

```yaml
---
- name: Configure ISC BIND with RPZ
  hosts: bind
  gather_facts: true
  roles:
    - role: jomrr.bind
      vars:
        bind_options:
          - name: recursion
            value: "yes"
          - name: response-policy
            entries:
              - zone "rpz.local"
        bind_zones:
          - name: rpz.local
            type: primary
            file: /var/named/zones/db.rpz.local
            statements:
              - allow-query { localhost; }
        bind_zone_files:
          - name: rpz.local
            file: /var/named/zones/db.rpz.local
            primary: ns1.rpz.local.
            email: hostmaster.rpz.local.
            records:
              - name: bad.example
                type: CNAME
                data: .
```
### Samba BIND_DLZ primary

Load Samba's BIND_DLZ database module and keep zones inside Samba.

```yaml
---
- name: Configure Samba BIND_DLZ primary
  hosts: bind
  gather_facts: true
  roles:
    - role: jomrr.bind
      vars:
        bind_options:
          - name: recursion
            value: "yes"
          - name: allow-query
            entries:
              - localhost
        bind_dlz:
          - name: AD DNS Zone
            statements:
              - database "dlopen /usr/lib64/samba/bind9/dlz_bind9.so"
```
### Secondary for Samba DNS primary

Configure secondary zones that transfer from a Samba BIND_DLZ primary.

```yaml
---
- name: Configure ISC BIND secondary zones
  hosts: bind
  gather_facts: true
  vars:
    samba_secondary_zone: &samba_secondary_zone
      class: IN
      type: secondary
      primaries:
        - '"samba-dlz"'
      allow_update_forwarding:
        - any
  roles:
    - role: jomrr.bind
      vars:
        bind_acls:
          - name: dns-clients
            entries:
              - 127.0.0.1
              - 192.168.0.0/16
              - 10.0.0.0/16
          - name: dns-listen
            entries:
              - 127.0.0.1
              - 192.168.254.20
              - 10.0.0.10
        bind_primaries:
          - name: samba-dlz
            entries:
              - 192.168.254.10
        bind_controls:
          - inet 127.0.0.1 port 953 allow { 127.0.0.1; } read-only yes
        bind_options:
          - name: listen-on
            arguments: port 53
            entries:
              - '"dns-listen"'
          - name: listen-on-v6
            arguments: port 53
            entries:
              - none
          - name: transfer-source
            value: 192.168.254.20
          - name: recursion
            value: "yes"
          - name: allow-query
            entries:
              - '"dns-clients"'
          - name: allow-recursion
            entries:
              - '"dns-clients"'
          - name: allow-transfer
            entries:
              - none
          - name: forward
            value: only
          - name: forwarders
            entries:
              - 10.0.0.1
          - name: notify
            value: primary-only
          - name: dnssec-validation
            value: "no"
        bind_zones:
          - <<: *samba_secondary_zone
            name: mauer.in
            comment: main forward zone
            file: slaves/db.mauer.in
          - <<: *samba_secondary_zone
            name: home.mauer.in
            comment: AD forward zone
            file: slaves/db.home.mauer.in
          - name: fritz.box
            type: forward
            forward: only
            forwarders:
              - 192.168.72.1
```
### Authoritative primary

Configure an authoritative primary zone and render the zone file.

```yaml
---
- name: Configure authoritative BIND primary
  hosts: bind
  gather_facts: true
  roles:
    - role: jomrr.bind
      vars:
        bind_options:
          - name: recursion
            value: "no"
          - name: allow-transfer
            entries:
              - 192.0.2.54
        bind_zones:
          - name: example.com
            class: IN
            type: primary
            file: /var/named/zones/db.example.com
            update_policy:
              - action: grant
                identity: ddns
                match_type: name
                name: update.example.com.
                types:
                  - A
        bind_zone_files:
          - name: example.com
            file: /var/named/zones/db.example.com
            primary: ns1.example.com.
            email: hostmaster.example.com.
            records:
              - name: "@"
                type: NS
                data: ns1.example.com.
              - name: ns1
                type: A
                data: 192.0.2.53
          - name: 2.0.192.in-addr.arpa
            file: /var/named/zones/db.2.0.192.in-addr.arpa
            primary: ns1.example.com.
            email: hostmaster.example.com.
            records:
              - name: "53"
                type: PTR
                data: ns1.example.com.
```
### Authoritative secondary

Configure an authoritative secondary zone.

```yaml
---
- name: Configure authoritative BIND secondary
  hosts: bind
  gather_facts: true
  roles:
    - role: jomrr.bind
      vars:
        bind_primaries:
          - name: public-primary
            entries:
              - 192.0.2.53
        bind_options:
          - name: recursion
            value: "no"
        bind_zones:
          - name: example.com
            class: IN
            type: secondary
            primaries:
              - '"public-primary"'
            file: slaves/db.example.com
```

## References

- [BIND 9 Documentation](https://bind9.readthedocs.io/en/latest/)
- [BIND 9 Dynamic Update Policies](https://bind9.readthedocs.io/en/latest/reference.html#namedconf-statement-update-policy)
- [BIND 9 Response Rate Limiting](https://bind9.readthedocs.io/en/latest/reference.html#namedconf-statement-rate-limit)

## Author

[Jonas Mauer](https://github.com/jomrr)

## License

This project is licensed under the MIT License.
See [LICENSE](LICENSE) for the full license text.

Copyright (c) 2024-2026 Jonas Mauer.
