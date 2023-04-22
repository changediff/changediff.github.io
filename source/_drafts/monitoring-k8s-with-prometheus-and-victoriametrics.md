---
title: monitoring-k8s-with-prometheus-and-victoriametrics
tags:
---

# victoriametrics
安装 https://github.com/VictoriaMetrics/VictoriaMetrics/wiki/Single-server-VictoriaMetrics

# prometheus

```bash
kubectl get prometheuses.monitoring.coreos.com
```

```yaml
remoteWrite:
- url: http://10.2.14.176:8428/api/v1/write
externalLabels:
  prom_node: "hw.gbop-apisix-test"
```