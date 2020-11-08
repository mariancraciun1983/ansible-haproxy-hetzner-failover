<h1 align="center">HAProxy HA cluster with Corosync and Hetzner's Failover IP</h1>
<br />

<div align="center">
  <a href="https://travis-ci.com/mariancraciun1983/ansible-haproxy-hetzner-failover">
    <img src="https://travis-ci.com/mariancraciun1983/ansible-haproxy-hetzner-failover.svg?branch=master" alt="Build Status" />
  </a>
  <a href="https://galaxy.ansible.com/mariancraciun1983/haproxy_ha_hetzner_cluster">
    <img src="https://img.shields.io/ansible/role/51696" alt="Ansible Galaxy" />
  </a>
  <a href="https://galaxy.ansible.com/mariancraciun1983/haproxy_ha_hetzner_cluster">
    <img src="https://img.shields.io/ansible/quality/51696" alt="Ansible Quality Score" />
  </a>
  <a href="https://opensource.org/licenses/MIT">
    <img src="https://img.shields.io/badge/License-MIT-blue.svg" alt="MIT License" />
  </a>
</div>
<br />



## Introduction

  This ansible role provides the basic configuration for deploying HAProxy installation on any server, plus the High Availability using Hetzner's failover/vswitch IPs.

  While in cloud environments, almost all providers offers load balancing solutions, on bare metal servers, this is a bit tricky. The solution is to have some sort of capability at the network level, to switch the traffic from one server to another.

  Hetzner offers A Failover IP that can be switched from one server to another. It can be ordered for any Hetzner dedicated server, and switched to any other Hetzner dedicated server, regardless of location. This will be our **public failover ip**.

  Hetzner also offers a vSwitch for dedicated root servers that lets you connect your servers in multiple locations to each other using virtual layer 2 networks. This will allow us to have a private network for the **private floating ip** or **VIP**.

  Corosync is a program that provides cluster membership and messaging capabilities and Pacemaker is a cluster resource manager.

  A python script will interact with the Hetzner's Web Service APIs to migrate the failover ip from one server to another. This process takes about 30 seconds, so in care of failover, expect for such a downtime.

## HAProxy High Availability

  Corosync will be in charge of the following:

  - manage the HAProxy service (res HAproxy)
  - assignt the **private floating IP/VIP** (res HaproxyVipInternal)
  - assignt the **public Failover IP** (res HaproxyVipExternal)
  - tell Hetzner's Web Service APIs to reassign the Failover IP (res HetznerFailoverIp)
  - make sure that all these 4 resources are assigned to a single server at once in a specific order

## Configuration example

For a simeple configuration, withouth HA

```yaml
group_vars:
  all:
    haproxy_ssl_deploy: true
    # this folder contains *.pem files
    haproxy_ssl_source: files/certificates
    haproxy_stats: true

    haproxy_frontends:
      - name: main-http
        description: Front-end for all HTTP/S traffic
        bind:
          - listen: 0.0.0.0:80
          - listen: 0.0.0.0:443
            param:
              - ssl crt {{ haproxy_ssl_dir }}
        mode: http
        config:
          - default_backend default

    haproxy_backends:
      - name: default
        mode: http
        config:
          - http-request set-header Server haproxy
          - http-request return status 200 content-type text/html file /etc/haproxy/static/default.html

```

On addition, for HAProxy HA add the following:
```yaml
# This is required by the hetzner_failover python script
robotws_user: "hetzner_ws_user"
robotws_password: "hetzner_ws_pass"

# Haproxy specific settings
haproxy_ha: true
haproxy_ha_vip_internal:
  # internal vSwitch IP - used for internal communication
  ip: 10.0.0.2
  netmask: 24
  nic: vlan4000
haproxy_ha_failover_external:
  # external/failover IP - for external configuration
  ip: 188.40.16.180
  netmask: 32
  nic: enp2s0
```


## Testing

Molecule with docker is being used.

Running the tests:
```bash
pipenv install
pipenv run molecule test
```

## License

MIT License