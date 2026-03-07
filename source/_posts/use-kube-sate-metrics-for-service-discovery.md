---
title: 利用kube-state-metrics自动生成prometheus的配置文件 
date: 2021-02-09 13:05:00
categories:
  - 运维
tags: ["k8s", "kubernetes", "prometheus", "monitoring"]
---

最近运维的k8s集群的node节点变动频繁，总是要手动更新prometheus配置文件表示很蛋疼。所以研究一下怎么对node节点做service discovery，自动更新监控targets列表。

## 先看看已有的配置维护方案
prometheus的配置通过[file_sd_config](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#file_sd_config)实现动态加载，用Python脚本访问每个集群的apiserver获取node节点然后生成对应的json配置文件。

这个方案可以拿来直接用的，不过配置起来不够灵活。我不想在脚本里面维护各个集群的认证方式，只想安安静静的更新prometheus自己的配置。pass

## 再看看prometheus官方提供的方案
prometheus提供了[kubernetes_sd_config](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#kubernetes_sd_config)，可以在prometheus.yml中配置好集群的认证方式，这样prometheus会定期去各个apiserver获取需要监控的node列表。在测试环境折腾了半天，发现这种方式对于部署在内部的prometheus配置起来很友好，然后如果是多个集群共用一个prometheus的话认证证书维护起来比较麻烦且容易出现集群认证配置更新了，prometheus中的配置没更新的尴尬情况。虽说集群的认证更新不会很频繁，但是每次更新就得重启prometheus也是不方便。

所以这种的确是最优雅的方案，也被pass了。


## 我的方案
最后在研究promethes的各个监控项目的时候，发现了kubernetes官方提供了详细的node节点监控：[kube-state-metrics](https://github.com/kubernetes/kube-state-metrics/blob/master/docs/node-metrics.md)，这些监控配置同样可以通过file_sd_config动态加载。于是有了以下方案：

> 人工维护kube-state-metrics的配置`groups/kube-state-metrics/*.json`，使用定时执行的脚本通过获取localhost的prometheus监控数据来更新node列表

### prometheus部署
- prometheus.yaml中必备配置：
```
# my global config
global:
  scrape_interval:     60s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  scrape_timeout: 60s
  # scrape_timeout is set to the global default (10s).

  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
      monitor: 'k8s-prometheus-monitor'

alerting:
  alertmanagers:
  - static_configs:
    - targets: ["localhost:9093"]

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first.rules"
  # - "second.rules"
  - /home/server/prometheus/rule.yml
# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.


scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'kube-state-metrics'
    scrape_interval: 180s
    scrape_timeout:  30s
    file_sd_configs:
      - files: ['groups/kube-state-metrics/*.json']
    metric_relabel_configs:
      - source_labels: [__name__]
        regex: "(kube_node_status_condition|kube_node_labels|kube_node_info|kube_pod_container_resource_requests_cpu_cores|kube_node_status_allocatable_cpu_cores|kube_pod_container_resource_requests_memory_bytes|kube_node_status_allocatable_memory_bytes)"
        action: keep

  - job_name: 'cadvisor'
    scrape_interval: 90s
    scrape_timeout:  30s
    file_sd_configs:
      - files: ['groups/cadvisor/*.json']
    metric_relabel_configs:
      - source_labels: [__name__]
        regex: "(container_cpu_usage_seconds_total|container_memory_rss|container_memory_usage_bytes|container_spec_memory_limit_bytes|container_spec_cpu_quota|container_memory_swap|container_memory_cache|container_network_receive_bytes_total|container_network_transmit_bytes_total|container_cpu_cfs_throttled_periods_total|container_cpu_cfs_periods_total|container_cpu_user_seconds_total|container_cpu_system_seconds_total|container_memory_failures_total|container_fs_reads_bytes_total|container_fs_writes_bytes_total|container_cpu_cfs_throttled_seconds_total|container_memory_working_set_bytes|kube_deployment_spec_replicas|kube_node_status_capacity_cpu_cores|kube_pod_container_resource_limits|kube_pod_container_resource_limits_cpu_cores|kube_pod_container_resource_limits_memory_bytes|kube_pod_container_resource_requests|kube_pod_container_resource_requests_cpu_cores|kube_replicationcontroller_spec_replicas|kube_replicationcontroller_status_replicas)"
        action: keep

  # kubernetes > 1.13
  - job_name: 'cadvisor-standalone'
    scrape_interval: 90s
    scrape_timeout:  30s
    file_sd_configs:
      - files: ['groups/cadvisor-standalone/*.json']
    metric_relabel_configs:
      - source_labels: ['container_label_io_kubernetes_pod_name']
        target_label: 'pod_name'
      - source_labels: ['container_label_io_kubernetes_container_name']
        target_label: 'container_name'
      - source_labels: [__name__]
        regex: "(container_cpu_usage_seconds_total|container_memory_rss|container_memory_usage_bytes|container_spec_memory_limit_bytes|container_spec_cpu_quota|container_memory_swap|container_memory_cache|container_network_receive_bytes_total|container_network_transmit_bytes_total|container_cpu_cfs_throttled_periods_total|container_cpu_cfs_periods_total|container_cpu_user_seconds_total|container_cpu_system_seconds_total|container_memory_failures_total|container_fs_reads_bytes_total|container_fs_writes_bytes_total|container_cpu_cfs_throttled_seconds_total|container_memory_working_set_bytes|kube_deployment_spec_replicas|kube_node_status_capacity_cpu_cores|kube_pod_container_resource_limits|kube_pod_container_resource_limits_cpu_cores|kube_pod_container_resource_limits_memory_bytes|kube_pod_container_resource_requests|kube_pod_container_resource_requests_cpu_cores|kube_replicationcontroller_spec_replicas|kube_replicationcontroller_status_replicas)"
        action: keep

  - job_name: 'node-exporter'
    scrape_interval: 90s
    scrape_timeout:  30s
    file_sd_configs:
      - files: ['groups/node-exporter/*.json']
    metric_relabel_configs:
      - source_labels: [__name__]
        regex: "(node_cpu_seconds_total|node_memory_MemAvailable_bytes|node_memory_MemTotal_bytes|node_load1|node_load5)"
        action: keep
```


- docker-comose.yaml配置
```
prometheus:
  image: prom/prometheus:v2.24.1
  net: host
  restart: always
  environment:
   -  TZ=Asia/Shanghai
  volumes:
   -  /etc/localtime:/etc/localtime:ro
   - ./prometheus/:/etc/prometheus/
   - /home/data/prometheus_data/:/prometheus_data/:rw
  command:
   - '--config.file=/etc/prometheus/prometheus.yml'
   - '--storage.tsdb.path=/prometheus_data/'
   - '--storage.tsdb.retention.time=2d'
   - '--storage.tsdb.max-block-duration=2h'
   - '--storage.tsdb.min-block-duration=2h'
   - '--query.max-samples=100000000'
   - '--web.console.libraries=/usr/share/prometheus/console_libraries'
   - '--web.console.templates=/usr/share/prometheus/consoles'
  ports:
   - 9090:9090

alertmanager:
  image: prom/alertmanager
  ports:
    - 9093:9093
  volumes:
    - ./alertmanager/:/etc/alertmanager/
  net: host
  restart: always
  command:
    - '--config.file=/etc/alertmanager/config.yml'
    - '--storage.path=/alertmanager'
```
- 启动脚本
```
#!/bin/bash

mkdir -p /home/server/prometheus/groups/{kube-state-metrics,node-exporter,cadvisor,cadvisor-standalone}
mkdir -p /home/data/prometheus_data/
mkdir -p /home/server/alertmanager/
```
- Python脚本生成node配置
```
# coding: utf-8

import json
import socket
import time
from datetime import datetime

import requests


def send_alarm(msg):
    pass

def simple_query(query='kube_node_status_condition{status="true"}==1'):
    """
    获取5分钟前的监控数据
    """
    step = 20
    now = int(time.time())
    start = now - 300
    end = now - 300
    params = (
        ('query', str(query)),
        ('start', str(start)),
        ('end', str(end)),
        ('step', str(step)),
    )

    response = requests.get(
        'http://localhost:9090/api/v1/query_range', params=params)

    if response:
        return response.json()
    else:
        return None


def json_node_exporter_cadvisor():
    kube_node_status_condition = simple_query(
        query='kube_node_status_condition{status="true"}==1')
    kube_node_labels = simple_query(query='kube_node_labels')
    kube_node_info = simple_query(query='kube_node_info')
    try:
        cluster_count_old = {}
        with open("/home/server/prometheus/groups/node-exporter/config_node_exporter.json") as f:
            config = json.loads(f.read())
            for node in config:
                cluster = node["labels"]["cluster"]
                if cluster not in cluster_count_old:
                    cluster_count_old[cluster] = 1
                else:
                    cluster_count_old[cluster] += 1
    except:
        cluster_count_old = {}


    try:
        nodes_ready = [i["metric"]["node"]
                       for i in kube_node_status_condition["data"]["result"]]
        node_label_dict = {
            i["metric"]["node"]: {
                "cluster": i["metric"]["cluster"],
                "group": i["metric"].get("label_group")
            } for i in kube_node_labels["data"]["result"]
        }
        node_version_dict = {
            i["metric"]["node"]: str(
                i["metric"]["kubelet_version"]).strip("v").split(".")
            for i in kube_node_info["data"]["result"]
        }
        config_node_exporter = []
        config_cadvisor = []
        config_cadvisor_standalone = []
        cluster_count_new = {}
        for n in nodes_ready:
            targets_9100 = [str(n) + ":9100"]
            targets_4194 = [str(n) + ":4194"]
            version = node_version_dict[n]
            cluster = node_label_dict[n]["cluster"]
            if cluster not in cluster_count_new:
                cluster_count_new[cluster] = 1
            else:
                cluster_count_new[cluster] += 1
            item_node = {"labels": node_label_dict[n], "targets": targets_9100}
            item_cadvisor = {
                "labels": node_label_dict[n], "targets": targets_4194}
            config_node_exporter.append(item_node)
            if int(version[1]) == 1 and int(version[1]) >= 14:
                config_cadvisor_standalone.append(item_cadvisor)
            else:
                config_cadvisor.append(item_cadvisor)

        cluster_config_change = {
            cluster: cluster_count_new.get(cluster, 0) - cluster_count_old.get(cluster) for cluster in cluster_count_old
        }
        change_min = min(list(cluster_config_change.values()))
        if change_min >= -3:  #node节点减少如果大于3个则不自动更新配置
            with open("/home/server/prometheus/groups/node-exporter/config_node_exporter.json", "w") as f:
                f.write(json.dumps(config_node_exporter, indent=4))
            with open("/home/server/prometheus/groups/cadvisor/config_cadvisor.json", "w") as f:
                f.write(json.dumps(config_cadvisor, indent=4))
            with open("/home/server/prometheus/groups/cadvisor-standalone/config_cadvisor_standalone.json", "w") as f:
                f.write(json.dumps(config_cadvisor_standalone, indent=4))
        else:
            with open("/home/server/prometheus/groups/node-exporter/config_node_exporter.json.new", "w") as f:
                f.write(json.dumps(config_node_exporter, indent=4))
            with open("/home/server/prometheus/groups/cadvisor/config_cadvisor.json.new", "w") as f:
                f.write(json.dumps(config_cadvisor, indent=4))
            with open("/home/server/prometheus/groups/cadvisor-standalone/config_cadvisor_standalone.json.new", "w") as f:
                f.write(json.dumps(config_cadvisor_standalone, indent=4))

            msg = "prometheus配置中node数量变更为{}，请确认配置: {}".format(str(change_min), str(cluster_config_change))
            send_alarm(msg)
        return cluster_config_change
    except Exception as e:
        print(e)

if __name__ == "__main__":
    res = json_node_exporter_cadvisor()
    print(res)
```
