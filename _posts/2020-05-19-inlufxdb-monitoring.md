---
title: "Telegraf & Chronograf 을 이용한 influxdb 모니터링 대시보드"
date: 2020-05-19 08:26:28 +0900
categories: Influxdb
tags:
  - influxdb
  - telegraf
  - chronograf
  - monitoring
---
## Telegraf & Chronograf 을 이용한 influxdb 모니터링 대시보드
Kubernetes에서 사용중인 Influxdb 모니터링을 위해 대시보드를 구성해보았습니다.
대시보드 구성을 위한 에이전트와 데이터 시각화 응용프로그램은 Influxdata에서 개발한 오픈소스 Telegraf와 chronograf를 사용하였습니다.

### Telegraf
Telegraf는 메트릭을 수집, 처리, 집계 및 작성하는 에이전트입니다.
디자인 목표는 플러그인 시스템으로 최소한의 메모리 사용 공간을 확보하여 커뮤니티의 개발자가 메트릭 수집 지원을 쉽게 추가 할 수 있도록하는 것입니다.
Telegraf는 플러그인 중심이며 4 가지 고유 한 플러그인 유형 개념이 있습니다.

입력 플러그인 은 시스템, 서비스 또는 타사 API에서 메트릭을 수집합니다.
프로세서 플러그인 은 메트릭을 변환, 장식 및 / 또는 필터링합니다.
집계 플러그인은 집계 메트릭 (예 : 평균, 최소, 최대, Quantile 등)을 만듭니다.
출력 플러그인 은 다양한 대상에 메트릭을 작성합니다.
새로운 플러그인은 쉽게 기여할 수 있도록 설계되었으며, 풀 요청을 환영하며 가능한 많은 풀 요청을 통합하기 위해 노력하고 있습니다.

https://github.com/influxdata/telegraf

#### 구성방법
Influxdb 연결정보가 있는 Secret, Telegraf 설정을 위한 Configmap 그리고 Deployments, service yaml을 작성하여 실행시킵니다.
telegraf 설정 파일은 /etc/telegraf/telegraf.conf 에 위치합니다. 
telegraf가 실행된다고 해서 필요한 수집요소가 쌓이는것은 아닙니다. inputs.system, inputs,cpu와 같이 필요한 수집요소를 명시해야합니다.


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
Chronograf는 Go 및 React.js로 작성된 오픈 소스 웹 응용 프로그램으로 모니터링 데이터를 시각화하고 경고 및 자동화 규칙을 쉽게 만들 수있는 도구를 제공합니다.
{% capture fig_img %}
[![Foo](https://github.com/influxdata/chronograf/blob/master/docs/images/overview-readme.png)](https://github.com/influxdata/chronograf)
{% endcapture %}

Chronograf는 다양한 대시보드 템플릿을 제공하고 있어 초기구성시 시간 절약에 도움이 됩니다.
Apache, Docker, InfluxDB, Kubernetes,Memcached, MongoDB등등 지원합니다.

https://github.com/influxdata/chronograf

#### 구성방법
Deployment, service로 구성하며 외부 접속을 위해 NodePort로 설정하였습니다.

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


