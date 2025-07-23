# ansible-role-strongswan

## What it is

An Ansible role to install and configure Strongswan, an IPsec VPN implementation for Linux servers.

Currently the role only supports configuring the deprecated (but functional) `stroke` plugin resources (`ipsec.conf`, `ipsec.secrets`). In the future support for the `vici` API resources (`swanctl.conf`).

## Supported distributions

This role has been tested as working on the following Linux distributions:
  - Ubuntu 24.04
  - Ubuntu 22.04

The Molecule directory contains scenarios for each supported distribution. The testing configuration is a simple Point-to-Point VPN. See the [Molecule README](./molecule/README.md) for instructions on testing.

## Role Variables

| Variable                        | Default                 | Required | Description                                                          |
|---------------------------------|-------------------------|----------|----------------------------------------------------------------------|
| `ipsec_conf`                    | `""`                    | Yes      | IPsec configuration content for `/etc/ipsec.conf`                    |
| `ipsec_secrets`                 | `""`                    | Yes      | IPsec secrets configuration for `/etc/ipsec.secrets`                 |
| `strongswan_conf`               | See `defaults/main.yml` | No       | StrongSwan daemon configuration for `/etc/strongswan.conf`           |
| `strongswan_enable_service`     | `true`                  | No       | Whether the strongswan-starter service should be enabled at boot     |
| `strongswan_additional_packages`| `[]`                    | No       | Additional StrongSwan packages to install beyond the base packages   |

## Example Playbook

```yaml
---
- hosts: vpn_server
  become: true
  gather_facts: true
  roles:
    - ansible-role-strongswan
  vars:
    # Basic site-to-site VPN configuration
    ipsec_conf: |
      config setup
        charondebug="ike 1, knl 1, cfg 0"
        uniqueids=no

      conn %default
        ikelifetime=60m
        keylife=20m
        rekeymargin=3m
        keyingtries=1
        keyexchange=ikev2
        authby=psk

      conn site-to-site
        left=192.168.1.1
        leftsubnet=192.168.1.0/24
        right=192.168.2.1
        rightsubnet=192.168.2.0/24
        auto=start

    ipsec_secrets: |
      192.168.1.1 192.168.2.1 : PSK "your-pre-shared-key-here"

    # Optional: Enable additional plugins
    strongswan_additional_packages:
      - libcharon-extra-plugins
      - libstrongswan-extra-plugins

    # Optional: Custom strongswan.conf (uses sensible defaults if not specified)
    strongswan_conf: |
      charon {
        load_modular = yes
        plugins {
          include strongswan.d/charon/*.conf
        }
        dns1 = 8.8.8.8
        dns2 = 8.8.4.4
      }

      include strongswan.d/*.conf
```
