panoptes - Network monitoring between multiple hosts
=========

This role will setup a network monitoring stack consisting *Node Exporter*, *Blackbox exporter*, *Prometheus* and *Grafana*.


Requirements
------------

As many nodes as you want. This role has been tested on 3 nodes of Ubuntu 22.04.

Role Variables
--------------

This will be your inventory:

```yaml
all:
  hosts:
  children:
    exporters:
      hosts:
        node-11:
          ip: 192.168.56.111
          job_name: blackbox_node_1
        node-12:
          ip: 192.168.56.112
          job_name: blackbox_node_2
        node-13:
          ip: 192.168.56.113
          job_name: blackbox_node_3
    monitoring:
      hosts:
        node-11:
          ip: 192.168.56.111
  vars:
    ping_endpoints:
      - 192.168.56.111
      - 192.168.56.112
      - 192.168.56.113

    ssh_endpoints:
      - 192.168.56.111
      - 192.168.56.112
      - 192.168.56.113

    http_https_endpoints:
      - https://jibit.ir
      - 192.168.56.101
      - 192.168.56.102
      - 192.168.56.103

    tcp_endpoints:
      - 192.168.56.101:8000
      - 192.168.56.102:8000
      - 192.168.56.103:8000
```

**Host Groups:**
This role will install *Blackbox exporter* and *Node exporter* on every host inside `exporters` group.
And it will setup Grafana and Prometheus on a host in `monitoring` group.

**Variables:**
Each variable endicates a hosts and enpoints being monitored by blackbox.
For example, in `ping_endpoints`, each blackbox node will ping the endpoint.
And `http_https_endpoints` will monitor http and https endpoints.

Example Playbook
----------------

This is an example playbook:

```yaml
---
- name: Converge - Install blackbox exporter and node exporter
  hosts: exporters
  become: true
  become_method: sudo
  vars:
    action: "setup_exporters"
  tasks:
    - name: "Include jbt.panoptes"
      include_role:
        name: "../roles/panoptes"

- name: Converge - Install Prometheus
  hosts: monitoring
  become: true
  become_method: sudo
  vars:
    action: "setup_prometheus"
  tasks:
    - name: "Include jbt.panoptes"
      include_role:
        name: "../roles/panoptes"

- name: Converge - Install Grafana
  hosts: monitoring
  become: true
  become_method: sudo
  vars:
    action: "setup_grafana"
  tasks:
    - name: "Include jbt.panoptes"
      include_role:
        name: "../roles/panoptes"

```
It's self-explaining, no need to over-explain things, right?


Author Information
------------------

JIBit DevOps Team.
