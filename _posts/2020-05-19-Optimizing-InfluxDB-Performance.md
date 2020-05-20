---
title: "Telegraf & Chronograf 을 이용한 influxdb 모니터링 대시보드"
date: 2020-05-19 08:26:28 -0400
categories: Influxdb
tags:
  - influxdb
  - telegraf
  - chronograf
  - monitoring
---
## Telegraf & Chronograf 을 이용한 influxdb 모니터링 대시보드

### Telegraf

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: telegraf-secrets
  namespace: kube-system
type: Opaque
stringData:
  INFLUXDB_DB: telegraf
  INFLUXDB_URL: http://10.96.1.11:8086
  INFLUXDB_USER: root
  INFLUXDB_USER_PASSWORD: root
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: telegraf-config
  namespace: kube-system
data:
  telegraf.conf: |+
    [[outputs.influxdb]]
      urls = ["$INFLUXDB_URL"]
      database = "$INFLUXDB_DB"
      username = "$INFLUXDB_USER"
      password = "$INFLUXDB_USER_PASSWORD"
    [[inputs.influxdb]]
      urls = [
        "http://10.96.1.11:8086/debug/vars"
      ]
      timeout = "5s"
    [[inputs.statsd]]
      max_tcp_connections = 250
      tcp_keep_alive = false
      service_address = ":8125"
      delete_gauges = true
      delete_counters = true
      delete_sets = true
      delete_timings = true
      metric_separator = "."
      allowed_pending_messages = 10000
      percentile_limit = 1000
      parse_data_dog_tags = true 
      read_buffer_size = 65535
    [agent]
      hostname = "monitoring-influxdb"
      flush_interval = "15s"
      interval = "15s"
    [[inputs.system]]
    [[inputs.cpu]]
      name_suffix = "_total"
      percpu = false
      totalcpu = true
    [[inputs.mem]]
    [[inputs.net]]
    [[inputs.disk]]
      tagexclude = ["fstype"]
---
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: kube-system
  name: telegraf
spec:
  selector:
    matchLabels:
      app: telegraf
  minReadySeconds: 5
  template:
    metadata:
      labels:
        app: telegraf
    spec:
      containers:
        - image: telegraf:latest
          name: telegraf
          envFrom:
            - secretRef:
                name: telegraf-secrets
          volumeMounts:
            - name: telegraf-config-volume
              mountPath: /etc/telegraf/telegraf.conf
              subPath: telegraf.conf
              readOnly: true
            - mountPath: /etc/localtime
              name: timezone
      volumes:
        - name: telegraf-config-volume
          configMap:
            name: telegraf-config
        - hostPath:
            path: /etc/localtime
            type: ''
          name: timezone
---
apiVersion: v1
kind: Service
metadata:
  name: telegraf
  namespace: kube-system
spec:
  externalTrafficPolicy: Cluster
  ports:
  - nodePort: 32285
    port: 8125
    protocol: UDP
    targetPort: 8125
  selector:
    app: telegraf
  sessionAffinity: None
  type: NodePort
```

###  Chronograf

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  namespace: kube-system
  name: chronograf
spec:
  selector:
    matchLabels:
      app: chronograf
  minReadySeconds: 5
  template:
    metadata:
      labels:
        app: chronograf
    spec:
      containers:
        - image: chronograf:latest
          name: chronograf
---
apiVersion: v1
kind: Service
metadata:
  name: chronograf
  namespace: kube-system
spec:
  externalTrafficPolicy: Cluster
  ports:
  - nodePort: 32286
    port: 8888
    protocol: TCP
    targetPort: 8888
  selector:
    app: chronograf
  sessionAffinity: None
  type: NodePort
```