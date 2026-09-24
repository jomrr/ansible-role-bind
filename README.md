# Ansible Role: bind

![GitHub](https://img.shields.io/github/license/jomrr/ansible-role-bind)
![GitHub last commit](https://img.shields.io/github/last-commit/jomrr/ansible-role-bind)
![GitHub issues](https://img.shields.io/github/issues-raw/jomrr/ansible-role-bind)
[![dev](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-bind/dev.yml?branch=dev&label=dev)](https://github.com/jomrr/ansible-role-bind/actions/workflows/dev.yml?query=branch%3Adev)
[![main](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-bind/main.yml?branch=main&label=main)](https://github.com/jomrr/ansible-role-bind/actions/workflows/main.yml?query=branch%3Amain)

Ansible role for setting up the ISC BIND DNS Server.

## Purpose

This role installs ISC BIND, renders BIND top-level configuration blocks,
enables and starts the BIND service, and restarts it after configuration
changes.

## Scope

### Managed

- ISC BIND packages through platform-specific package lists
- Main BIND configuration file
- TSIG key files from provided key material
- ACL, primaries, controls, options, logging, include, and DLZ clauses
- Primary, secondary, forward, RPZ, and related zone declarations
- TSIG-protected zone transfer and DDNS configuration through native BIND
  statements
- Managed authoritative forward and reverse zone files with SOA serial updates
- Ordered response policies and validated external RPZ master files downloaded
  through the controller
- Zone-level BIND update-policy rules
- BIND service handler for configuration changes

### Not Managed

- Firewall policy
- DNSSEC key lifecycle
- TLS certificate and private key lifecycle
- Automatic TSIG key material generation

## Requirements

- Target hosts need platform repositories that provide ISC BIND packages.
- Source zones require controller-side HTTPS access and a working local RNDC
  control channel on the primary.

## Dependencies

```yaml
collections:
  - name: community.general
    version: '>=12.0.0'
  - name: ansible.posix
    version: '>=2.0.0'
```

## Role Variables

### `bind_tsig_keys`

Type: `list`. Required: `false`.

TSIG key declarations included from the managed local BIND configuration.
Store real secrets in Ansible Vault.

Default:

```yaml
bind_tsig_keys: []
```

### `bind_acls`

Type: `list`. Required: `false`.

Named BIND ACL declarations.
The default includes local, my_addresses, and bogons ACLs.
The my_addresses ACL is referenced by the default deny-answer-addresses option
for recursive resolver hardening.
The bogons ACL is referenced by the default blackhole option and excludes
loopback, private, shared, and ULA ranges commonly used by local clients.

### `bind_primaries`

Type: `list`. Required: `false`.

Named BIND primaries lists for secondary zones.

Default:

```yaml
bind_primaries: []
```

### `bind_controls`

Type: `list`. Required: `false`.

Native BIND controls entries rendered inside a controls block.

Default:

```yaml
bind_controls: []
```

### `bind_tls`

Type: `list`. Required: `false`.

Top-level BIND TLS blocks referenced by DNS-over-TLS, DNS-over-HTTPS, or TLS
zone transfer configuration.

Default:

```yaml
bind_tls: []
```

### `bind_http`

Type: `list`. Required: `false`.

Top-level BIND HTTP blocks referenced by DNS-over-HTTPS listeners.

Default:

```yaml
bind_http: []
```

### `bind_listeners`

Type: `list`. Required: `false`.

BIND listen-on or listen-on-v6 statements rendered inside the options block.
Listener entries support classic DNS, DNS-over-TLS, and DNS-over-HTTPS.

### `bind_validate_except`

Type: `list`. Required: `false`.

Domains permanently excluded from DNSSEC validation, including their subdomains.
Referenced by the default bind_options list; replacing that list removes this
binding.

Default:

```yaml
bind_validate_except: []
```

### `bind_options`

Type: `list`. Required: `false`.

Ordered BIND option statements rendered inside the options block.
Internal defaults, platform defaults, and bind_options are merged with
precedence internal, platform, then user.
Entries with the same name keep the first output position and use the value from
the last definition.
Raw and include entries without a name are opaque and are rendered without
deduplication.
Set name on a raw or include entry when it should replace an earlier same-name
option.
Empty values omit the matching option.
Response rate limiting is opt-in through a rate-limit option entry.

### `bind_response_policy`

Type: `dict`. Required: `false`.

Ordered response policy zones and global BIND RPZ modifiers.
No statement is rendered for an empty mapping or an empty zones list.
Do not also define response-policy in bind_options.

Default:

```yaml
bind_response_policy: {}
```

### `bind_logging`

Type: `dict`. Required: `false`.

BIND logging configuration with channels and categories.
Platform channels and categories are extended by the supplied lists.
A supplied entry replaces a platform entry with the same name.

Default:

```yaml
bind_logging:
  channels: []
  categories: []
```

### `bind_includes`

Type: `list`. Required: `false`.

Additional top-level BIND include files rendered after platform default
includes.

Default:

```yaml
bind_includes: []
```

### `bind_dlz`

Type: `list`. Required: `false`.

Top-level BIND DLZ blocks, for example Samba BIND_DLZ integration.

Default:

```yaml
bind_dlz: []
```

### `bind_zones`

Type: `list`. Required: `false`.

BIND zone declarations for primary, secondary, forward, RPZ, and related zones.
Zone declarations and managed records use the Internet DNS class IN.
Static primary zone files are managed directly from this variable.
Primary zones without source require ns_records for the authoritative zone base.
Source zones import complete external master files through the controller,
preserving their contents and serials.
Dynamic primary zone files are created only when missing with SOA and
ns_records; runtime records belong to DDNS updates.
The file option is a file name only; the role places it in the platform-native
directory for the zone type.

Default:

```yaml
bind_zones: []
```

### `bind_extra_statements`

Type: `list`. Required: `false`.

Additional complete top-level BIND statements for unsupported edge cases.

Default:

```yaml
bind_extra_statements: []
```

### `bind_zone_file_ttl`

Type: `str`. Required: `false`.

Default TTL for managed zone files.

Default:

```yaml
bind_zone_file_ttl: 1h
```

### `bind_zone_file_refresh`

Type: `str`. Required: `false`.

Default SOA refresh interval for managed zone files.

Default:

```yaml
bind_zone_file_refresh: 1h
```

### `bind_zone_file_retry`

Type: `str`. Required: `false`.

Default SOA retry interval for managed zone files.

Default:

```yaml
bind_zone_file_retry: 15m
```

### `bind_zone_file_expire`

Type: `str`. Required: `false`.

Default SOA expire interval for managed zone files.

Default:

```yaml
bind_zone_file_expire: 1w
```

### `bind_zone_file_minimum`

Type: `str`. Required: `false`.

Default SOA MINIMUM field for negative caching (RFC 2308).
Negative answers use the lower of this value and the SOA record TTL.

Default:

```yaml
bind_zone_file_minimum: 1h
```

## Managed Files

- `/etc/bind/named.conf` on Debian-family systems
- `/etc/named.conf` on Red Hat-family and Suse systems
- `<platform config directory>/<key-name>.key` when TSIG keys are configured
- `<platform zone directory>/<zone-file>` when bind_zones includes managed zone
  file content

## Check Mode

Check mode predicts changes on configured hosts; reading existing SOA serials
remains read-only.

- A first run in check mode requires the BIND packages and their configuration
  directories to exist already.
- Source files are downloaded on the controller and compared with installed
  files in check mode.
- Source candidates are not staged, checked with named-checkzone,
  serial-validated, installed, or reloaded in check mode.

## Service Behavior

The role enables and starts the BIND service. Managed configuration
changes notify the restart handler. External source content updates only
reload changed zones through RNDC, without restarting or globally reloading
BIND.

### Handlers

- BIND | Restart service

## Security Notes

- The default listener is limited to localhost.
- Recursive defaults explicitly limit cache access to local clients.
- Recursive defaults limit concurrent recursive clients.
- Version, hostname, and server-id disclosure are disabled by default.
- Recursive defaults deny answers that resolve names to the resolver's own
  addresses.
- Recursive defaults blackhole bogon source addresses.
- Recursive defaults limit outstanding fetches per upstream server and zone.
- Response rate limiting is opt-in; the authoritative DNS example configures
  explicit limits.
- Store TSIG secrets in Ansible Vault.

## Operational Notes

- The role is idempotent; unchanged declarations leave files, SOA serials, and
  the service unchanged.
- Configuration candidates are checked before installation; replaced managed
  files receive module-provided backups.
- Debian and Ubuntu use one managed named.conf; named.conf.options and
  named.conf.local are no longer included automatically.
- Existing static zone serials are read with named-checkzone; changed templates
  advance the serial to max(previous + 1, gathered Unix timestamp).
- The main API follows BIND's own top-level blocks.
- Use variables such as `bind_acls`, `bind_options`, and `bind_zones`.
- Internal defaults, platform defaults, and `bind_options` are merged with
  precedence internal, platform, then user.
- Same-name option entries are deduplicated; the first occurrence keeps the
  output position and the last definition supplies the rendered value.
- Raw and include option entries without `name` are opaque and are rendered
  unchanged. Add `name` to a raw or include entry only when it should replace an
  earlier same-name option.
- `bind_validate_except` permanently excludes the listed domains and their
  subdomains from DNSSEC validation. Its default is `[]`, with no exceptions.
  The default `bind_options` list references this variable; replacing that list
  removes the reference unless explicitly included again.
- Platform default includes are rendered automatically before additional
  `bind_includes`.
- `bind_dlz` renders top-level DLZ blocks only; all zone declarations belong in
  `bind_zones`.
- Managed zone files are declared in `bind_zones`; `file` is a file name only
  and the role selects the platform-native static, dynamic, or secondary
  directory.
- Primary zones without `source` declare their mandatory authoritative NS base
  through `ns_records`.
- Dynamic primary zone files are created only when missing with SOA and
  `ns_records`; runtime records belong to DDNS updates.
- Reverse zones use PTR records in the same zone template.
- Use explicit ACLs before widening query or recursion access.
- Keep a `my_addresses` ACL when replacing `bind_acls`, or adjust
  `deny-answer-addresses` accordingly.
- Keep a `bogons` ACL when replacing `bind_acls`, or adjust `blackhole`
  accordingly.
- Primary and secondary relationships are declared through `bind_primaries`,
  `bind_tsig_keys`, `bind_options`, and `bind_zones`.
- Changing provided TSIG key material updates rendered key files and restarts
  BIND.
- Use zone-level `update_policy` for granular DDNS permissions on primary zones.
- Do not combine BIND `update-policy` and `allow-update` for the same zone.
- Use `bind_response_policy` and RPZ zones for DNSBL-style response filtering.
- Add a `rate-limit` entry to `bind_options` to enable BIND response rate
  limiting for authoritative DNS.
- `bind_zone_file_minimum` sets SOA.MINIMUM for negative caching, not a lower
  bound for record TTLs. Under RFC 2308, NXDOMAIN and NODATA use the lower of
  SOA.MINIMUM and the SOA record TTL; both default to one hour. The SOA inherits
  `$TTL` from `bind_zone_file_ttl` or the per-zone `ttl` override.
- Static zones receive SOA changes on convergence. Existing dynamic zone files
  are not rewritten; their SOA must be updated through DDNS.

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

### Response policy zones

The first matching policy zone takes precedence. The local zone is
listed first and uses `rpz-passthru.` to allow a name blocked by a feed.
Each host declares its policies and zones explicitly; no inventory
discovery or automatic primary/secondary pairing takes place.
The listeners permit transfers between these two servers; recursive
client access retains the localhost-only defaults unless configured separately.

A source file supplies its own SOA and NS records. The controller
downloads it over HTTPS on each role run; redirects are rejected.
The defaults are `timeout: 30` and `validate_certs: true`. There are
no scheduled refreshes, and secondaries never download source URLs.

Only complete `$TTL` and `$ORIGIN` directive tokens at the beginning
of a line are accepted. Whitespace and parentheses prefixes, other
directives including quoted `$INCLUDE`, and invalid master files are
rejected before installation. Changed contents require a newer serial
according to RFC 1982; identical files need no serial increment.
The role never rewrites a source file's contents or serial.

`allow-query { localhost; }` restricts direct access to policy data;
RPZ still applies to recursive client queries. Transfer permission
is configured separately. A feed with only `localhost.` as its NS
needs explicit `also-notify` to notify the secondary.

`ixfr-from-differences yes` can reduce transfers for large feeds.
The example uses an explicit writable journal under `/var/named/dynamic`
on Red Hat systems. Use `/var/lib/bind` on Debian/Ubuntu or
`/var/lib/named/dyn` on openSUSE. These directories come from packages;
the role does not create or change their permissions.

Zone reloads require a writable local RNDC control channel. RNDC
errors fail the run without falling back to a global reload or restart.

```yaml
---
- name: Configure RPZ primary
  hosts: dns_primary
  gather_facts: true
  roles:
    - role: jomrr.bind
      vars:
        bind_listeners:
          - name: listen-on
            entries: [127.0.0.1, 10.53.0.10]
          - name: listen-on-v6
            entries: [none]
        bind_response_policy:
          zones:
            - name: local.rpz.example.com
            - name: feed.rpz.example.com
          options:
            - break-dnssec no
        bind_zones:
          - name: local.rpz.example.com
            type: primary
            file: db.local.rpz.example.com
            primary: localhost.
            email: hostmaster.example.com.
            ns_records:
              - name: localhost.
            records:
              - name: allowed.example.com
                type: CNAME
                data: rpz-passthru.
            statements:
              - allow-query { localhost; }
              - allow-transfer { 10.53.0.11; }
              - also-notify { 10.53.0.11; }
              - notify yes
          - name: feed.rpz.example.com
            type: primary
            file: db.feed.rpz.example.com
            source:
              url: https://raw.githubusercontent.com/hagezi/dns-blocklists/main/rpz/pro.txt
            statements:
              - allow-query { localhost; }
              - allow-transfer { 10.53.0.11; }
              - also-notify { 10.53.0.11; }
              - notify yes
              - ixfr-from-differences yes
              - journal "/var/named/dynamic/db.feed.rpz.example.com.jnl"
- name: Configure RPZ secondary
  hosts: dns_secondary
  gather_facts: true
  roles:
    - role: jomrr.bind
      vars:
        bind_listeners:
          - name: listen-on
            entries: [127.0.0.1, 10.53.0.11]
          - name: listen-on-v6
            entries: [none]
        bind_response_policy:
          zones:
            - name: local.rpz.example.com
            - name: feed.rpz.example.com
          options:
            - break-dnssec no
        bind_zones:
          - name: local.rpz.example.com
            type: secondary
            file: db.local.rpz.example.com
            primaries:
              - 10.53.0.10
            statements:
              - allow-query { localhost; }
          - name: feed.rpz.example.com
            type: secondary
            file: db.feed.rpz.example.com
            primaries:
              - 10.53.0.10
            statements:
              - allow-query { localhost; }
```

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

### DNSSEC validation exceptions

Exclude example.com and its subdomains from DNSSEC validation while
retaining all other default options.

```yaml
---
- name: Configure ISC BIND with a DNSSEC validation exception
  hosts: bind
  gather_facts: true
  roles:
    - role: jomrr.bind
      vars:
        bind_validate_except:
          - example.com
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
        bind_options:
          - name: listen-on
            arguments: port 53
            entries:
              - 127.0.0.1
              - 10.53.0.53
          - name: listen-on-v6
            arguments: port 53
            entries:
              - none
          - name: allow-query
            entries:
              - 127.0.0.1
              - 10.53.0.0/24
          - name: allow-query-cache
            entries:
              - 127.0.0.1
              - 10.53.0.0/24
          - name: allow-recursion
            entries:
              - 127.0.0.1
              - 10.53.0.0/24
          - name: recursion
            value: "yes"
          - name: qname-minimization
            value: strict
          - name: deny-answer-addresses
            entries:
              - 127.0.0.1
              - 10.53.0.53
          - name: blackhole
            entries:
              - bogons
          - name: recursive-clients
            value: 300
          - name: fetches-per-server
            value: 100 fail
          - name: fetches-per-zone
            value: 200 fail
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
          - name: forward
            value: only
          - name: forwarders
            entries:
              - 10.53.0.1
              - 10.54.0.1
```

### Authoritative response rate limiting

Serve a static example.com primary zone with recursion and cache access
disabled and response rate limiting enabled. Adjust the listener address
and limits to the deployment. RRL is intended for authoritative service;
on recursive resolvers it can delay legitimate repeated queries.

```yaml
---
- name: Configure authoritative BIND with response rate limiting
  hosts: bind
  gather_facts: true
  roles:
    - role: jomrr.bind
      vars:
        bind_listeners:
          - name: listen-on
            port: 53
            entries:
              - 127.0.0.1
              - 10.53.0.53
          - name: listen-on-v6
            port: 53
            entries:
              - none
        bind_options:
          - name: recursion
            value: "no"
          - name: allow-query
            entries:
              - any
          - name: allow-query-cache
            entries:
              - none
          - name: allow-recursion
            entries:
              - none
          - name: allow-transfer
            entries:
              - none
          - name: hostname
            value: none
          - name: server-id
            value: none
          - name: version
            value: none
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
        bind_zones:
          - name: example.com
            type: primary
            file: db.example.com
            primary: ns1.example.com.
            email: hostmaster.example.com.
            ns_records:
              - name: ns1.example.com.
                addresses:
                  - type: A
                    address: 10.53.0.53
            records:
              - name: www
                type: A
                data: 10.53.0.80
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
          - name: qname-minimization
            value: strict
          - name: deny-answer-addresses
            entries:
              - my_addresses
          - name: blackhole
            entries:
              - bogons
          - name: allow-query-cache
            entries:
              - local
          - name: recursive-clients
            value: 300
          - name: fetches-per-server
            value: 100 fail
          - name: fetches-per-zone
            value: 200 fail
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
        bind_response_policy:
          zones:
            - name: rpz.example.com
        bind_zones:
          - name: rpz.example.com
            type: primary
            file: db.rpz.example.com
            statements:
              - allow-query { localhost; }
            primary: ns1.rpz.example.com.
            email: hostmaster.rpz.example.com.
            ns_records:
              - name: ns1.rpz.example.com.
                addresses:
                  - type: A
                    address: 127.0.0.1
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
          - name: qname-minimization
            value: strict
          - name: deny-answer-addresses
            entries:
              - my_addresses
          - name: blackhole
            entries:
              - bogons
          - name: allow-query
            entries:
              - localhost
          - name: allow-query-cache
            entries:
              - localhost
          - name: recursive-clients
            value: 300
          - name: fetches-per-server
            value: 100 fail
          - name: fetches-per-zone
            value: 200 fail
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
      type: secondary
      primaries:
        - '"samba-dlz"'
      allow_update_forwarding:
        - any
  roles:
    - role: jomrr.bind
      vars:
        bind_primaries:
          - name: samba-dlz
            entries:
              - 10.53.0.10
        bind_controls:
          - inet 127.0.0.1 port 953 allow { 127.0.0.1; } read-only yes
        bind_options:
          - name: listen-on
            arguments: port 53
            entries:
              - 127.0.0.1
              - 10.53.0.20
              - 10.54.0.10
          - name: listen-on-v6
            arguments: port 53
            entries:
              - none
          - name: transfer-source
            value: 10.53.0.20
          - name: recursion
            value: "yes"
          - name: qname-minimization
            value: strict
          - name: deny-answer-addresses
            entries:
              - 127.0.0.1
              - 10.53.0.20
              - 10.54.0.10
          - name: blackhole
            entries:
              - bogons
          - name: allow-query
            entries:
              - 127.0.0.1
              - 10.53.0.0/24
              - 10.54.0.0/24
          - name: allow-query-cache
            entries:
              - 127.0.0.1
              - 10.53.0.0/24
              - 10.54.0.0/24
          - name: allow-recursion
            entries:
              - 127.0.0.1
              - 10.53.0.0/24
              - 10.54.0.0/24
          - name: recursive-clients
            value: 300
          - name: fetches-per-server
            value: 100 fail
          - name: fetches-per-zone
            value: 200 fail
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
          - name: allow-transfer
            entries:
              - none
          - name: forward
            value: only
          - name: forwarders
            entries:
              - 10.53.0.1
          - name: notify
            value: primary-only
          - name: dnssec-validation
            value: "no"
        bind_zones:
          - <<: *samba_secondary_zone
            name: example.com
            comment: main forward zone
            file: db.example.com
          - <<: *samba_secondary_zone
            name: ad.example.com
            comment: AD forward zone
            file: db.ad.example.com
          - name: branch.example.com
            type: forward
            forward: only
            forwarders:
              - 10.55.0.1
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
          - name: allow-transfer
            entries:
              - 10.53.0.54
        bind_zones:
          - name: example.com
            type: primary
            dynamic: true
            file: db.example.com
            update_policy:
              - action: grant
                identity: ddns
                match_type: name
                name: update.example.com.
                types:
                  - A
            primary: ns1.example.com.
            email: hostmaster.example.com.
            ns_records:
              - name: ns1.example.com.
                addresses:
                  - type: A
                    address: 10.53.0.53
          - name: 0.53.10.in-addr.arpa
            type: primary
            file: db.0.53.10.in-addr.arpa
            primary: ns1.example.com.
            email: hostmaster.example.com.
            ns_records:
              - name: ns1.example.com.
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
              - 10.53.0.53
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
        bind_zones:
          - name: example.com
            type: secondary
            primaries:
              - '"public-primary"'
            file: db.example.com
```

## References

- [RFC 2308: Negative Caching of DNS Queries](https://www.rfc-editor.org/rfc/rfc2308.html)
- [BIND 9 Documentation](https://bind9.readthedocs.io/en/latest/)
- [BIND 9 Dynamic Update Policies](https://bind9.readthedocs.io/en/latest/reference.html#namedconf-statement-update-policy)
- [BIND 9 Response Rate Limiting](https://bind9.readthedocs.io/en/latest/reference.html#namedconf-statement-rate-limit)

## Author

[Jonas Mauer](https://github.com/jomrr)

## License

This project is licensed under the MIT License.
See [LICENSE](LICENSE) for the full license text.

Copyright (c) 2024-2026 Jonas Mauer.
