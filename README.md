# ansible-role-strongswan

## What it is

An Ansible role to install and configure Strongswan, an IPsec VPN implementation for Linux servers.

This role supports configuring both the deprecated (but functional) `stroke` plugin resources (`ipsec.conf`, `ipsec.secrets`) and the new `vici` resources (`swanctl.conf`). You must select one or the other though, not both.

## Supported distributions

This role has been tested as working on the following Linux distributions:
  - Ubuntu 24.04
  - Ubuntu 22.04

The Molecule directory contains scenarios for each supported distribution. The testing configuration is a simple Point-to-Point VPN. See the [Molecule README](./molecule/README.md) for instructions on testing.

## Role Variables

| Variable                        | Default                 | Required | Description                                                          |
|---------------------------------|-------------------------|----------|----------------------------------------------------------------------|
| `strongswan_mode`               | `vici`                  | No       | Configuration mode: `vici` (modern) or `stroke` (deprecated)         |
| `strongswan_enable_service`     | `true`                  | No       | Whether the StrongSwan service should be enabled at boot             |
| `strongswan_additional_packages`| `[]`                    | No       | Additional StrongSwan packages to install beyond the base packages   |
| `strongswan_conf`               | See `defaults/main.yml` | No       | StrongSwan daemon configuration for `/etc/strongswan.conf`           |
| **Vici Mode Variables**         |                         |          |                                                                      |
| `swanctl_conf`                  | `""`                    | Yes*     | swanctl configuration content for `/etc/swanctl/swanctl.conf`        |
| **Stroke Mode Variables**       |                         |          |                                                                      |
| `ipsec_conf`                    | `""`                    | Yes*     | IPsec configuration content for `/etc/ipsec.conf`                    |
| `ipsec_secrets`                 | `""`                    | Yes*     | IPsec secrets configuration for `/etc/ipsec.secrets`                 |

*Required only when using the respective mode (vici or stroke).

## Example Playbook (vici)

```yaml
---
- hosts: new_vpn_server
  become: true
  gather_facts: true
  roles:
    - ansible-role-strongswan
  vars:
    # Basic site-to-site VPN configuration    
    swanctl_conf: |
      connections {
        site-to-site {
          version = 2
          local_addrs = %defaultroute
          remote_addrs = 10.10.10.10
          local {
            auth = psk
            id = site1.example.com
          }
          remote {
            auth = psk
            id = site2.example.com
          }
          children {
            site-to-site {
              local_ts = 192.168.1.0/24
              remote_ts = 192.168.2.0/24
              esp_proposals = aes256-sha256
              dpd_action = restart
              start_action = start
            }
          }
          proposals = aes256-sha256-modp2048
          dpd_delay = 30s
          dpd_timeout = 120s
          rekey_time = 8h
          reauth_time = 24h
        }
      }
      
      secrets {
        ike-psk {
          id = site1.example.com
          secret = "MySecurePreSharedKey123!"
        }
      }

    # Optional: Enable additional plugins
    strongswan_additional_packages:
      - libcharon-extra-plugins
      - libstrongswan-extra-plugins
```

## Example Playbook (stroke)

```yaml
---
- hosts: old_vpn_server
  become: true
  gather_facts: true
  roles:
    - ansible-role-strongswan
  vars:
    # Basic site-to-site VPN configuration
    strongswan_mode: stroke
    
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
      192.168.1.1 192.168.2.1 : PSK "MySecurePreSharedKey123!"
```
