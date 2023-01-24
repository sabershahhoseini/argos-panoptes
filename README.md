# Argos Panoptes
*Argus Panoptes* is a character in Greek mythology. He was a giant with 100 eyes on his body. **Panoptes** means all-seeing.

Argus was a servant of the goddess Hera and he made an excellent watchman because he never fell asleep.

This project will monitor network endpoints like **HTTP, HTTPS, DNS, TCP, ICMP, Network Traffic, Sockets** and more.

# Topology

This is how everything works:

![Diagram](pictures/diagram.png)

Blackbox exporter and node exporter are present in each node. Each of them expose network stats and information of other nodes.
These metrics will monitor each node's network status from every other node's point of view. So we could call it a monitoring mesh.

# Gettings Started

If you don't have Prometheus and Grafana already installed, you can head to [Setup](#setup) and setup everything.

If you already have Prometheus and Grafana servers, You can deploy exporters on your target nodes, and for configuration part, read [Prometheus Config](#prometheus-config) section to config your setup and after you've done the configuration, 

# Setup

Here, we will install everything with `docker-compose`. We have two compose files. First one will be deployed on your monitoring server (Can be any node). The compose file contains Prometheus, Grafana, node exporter and blackbox exporter containers. You can remove each one of the containers you want (For example, maybe you don't want to export anything on your monitoring server, so you would remove the blackbox exporter and node exporter containers).

## On Exporter nodes

Clone the repo:

```
git clone 'https://github.com/sabershahhoseini/argos-panoptes.git'
```

Start the containers:

```
docker-compose -f nodes-compose.yml up -d
```

Nice! You just deployed blackbox exporter and node exporter on your target nodes!

## On Monitoring Server

Clone the repo:

```
git clone 'https://github.com/sabershahhoseini/argos-panoptes.git'
```

### Prometheus Config

Change Prometheus config file at `./config-files/prometheus/`. You should add each node as a seperate blackbox job. The Prometheus will pull metrics from exporters.

**Please notice** that in Argos Panoptes dashboard, We have a variable called `job` that will match blackbox exporters and will be used to repeat Grafana rows. This variable uses `(blackbox.*(node.*))` regex, so for example, your blackbox job can be `blackbox_node_1, blackbox_node_2, ...`.
Same thing applies to node exporter with regex `(.*node_exporter.*)`, and you can change it if you want.

For example, if you want to monitor 3 nodes, you should first deploy exporters on those 3 nodes, and then add 3 blackbox jobs and 1 node exporter job in Prometheus config file.

Prometheus example config with 1 blackbox job:
```yaml
scrape_configs:

  - job_name: 'node_exporter'
    scrape_interval: 5s
    static_configs:
      - targets: ['192.168.56.101:9100', '192.168.56.103:9100', '192.168.56.104:9100']

#
# Blackbox exporter configuration
#

  - job_name: 'blackbox_node_1'
    scrape_interval: 10s
    metrics_path: /probe
    file_sd_configs:
      - files:
          - '/etc/prometheus/blackbox-targets/*.yml'   # Which modules to use
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [module]
        target_label: __param_module
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 192.168.56.101:9115   # IP of one of the nodes with blackbox exporter
```

After you've added your configuration to Prometheus config file, you will need to add your nodes in blackbox modules. Each module will export different metrics. We have modules for HTTP and HTTPS, TCP, Ping and SSH.
Example config will be:

```yaml
- labels:
    module: icmp
  targets:
    - 192.168.56.101
    - 192.168.56.103
    - 192.168.56.104
```
You will need to add IP of your nodes to be monitored in each file. Edit every module you want at `./config-files/prometheus/blackbox-targets/` and remove any module you won't use.

Now that you have configured Prometheus and blackbox, you can start the containers on your monitoring server:

```
docker-compose -f prometheus-server-compose.yml up -d
```

Now, go to Grafana settings and add Prometheus data source. Then, copy contents of`grafana-dashboard/argos-panoptes-dashboard.json` and import it in your Grafana server.

Important notice: This dashboard uses `job` variable for blackbox exporter jobs and `node_exporter_job` as node exporter jobs. If you see *No Data* in your Grafana dashboard, you probably will need to change these two variables.
