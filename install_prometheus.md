1. 修改入口
alertmanager-alertmanager.yaml
prometheus-prometheus.yaml

增加,确保externalUrl根ingress中打URL一致
```
externalUrl: http://prometheus.gzky.com
```

2. 修改alertmanager-secret.yaml配置通知方式

3. 增加job 仿照写ServiceMonitor

重新加载配置文件：
curl -X POST 10.101.142.187:9093/-/reload # (对应service打ip，port)

4. 增加警报规则
```
kind: PrometheusRule
metadata:
  creationTimestamp: null
  labels:
    prometheus: k8s
    role: alert-rules
  name: demo-test-rules
  namespace: monitoring
spec:
  groups:
  - name: test.rules
    rules:
    - alert: DemoAlert      for: 2m
      labels:
        app: demo-test
      expr: (avg by (instance) (increase(codelab_api_request_duration_seconds_bucket{method="POST",path="/api/foo",status="500"}[5m]))) > 100
  - name: demo-service.rules
    rules:
    - alert: '磁盘使用'
      expr: (node_filesystem_size{device="/dev/mapper/ol-root"} - node_filesystem_free{device="/dev/mapper/ol-root"}) / node_filesystem_size{device="/dev/mapper/ol-root"} * 100 > 80
      for: 2m
      labels:
        app: demo-test
      annotations:
        summary: "{{$labels.instance}}: High Filesystem usage detected"
        description: "{{$labels.instance}}: Filesystem usage is above 80% (current value is: {{ $value }}"
    - alert: '内存使用'
      expr: (node_memory_MemTotal - (node_memory_MemFree+node_memory_Buffers+node_memory_Cached )) / node_memory_MemTotal * 100 > 80
      for: 2m
      labels:
        app: demo-test
      annotations:
        summary: "{{$labels.instance}}: High Memory usage detected"
        description: "{{$labels.instance}}: Memory usage is above 80% (current value is: {{ $value }}"
    - alert: 'CPU使用'
      expr: (100 - (avg by (instance) (irate(node_cpu{job="node-exporter",mode="idle"}[5m])) * 100)) > 80
      for: 2m
      labels:
        app: demo-test
      annotations:
        summary: "{{$labels.instance}}: High CPU usage detected"
        description: "{{$labels.instance}}: CPU usage is above 80% (current value is: {{ $value }}"```

```

## 参考文档
https://github.com/coreos/prometheus-operator/blob/master/Documentation/user-guides/getting-started.md
https://prometheus.io/docs/prometheus/latest/querying/functions/
https://github.com/1046102779/prometheus

报警:
http://blog.qikqiak.com/post/alertmanager-of-prometheus-in-practice/
https://prometheus.io/docs/alerting/configuration/




