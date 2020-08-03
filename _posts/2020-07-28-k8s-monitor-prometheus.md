---
title: "Kubernetes Monitoring - Prometheus"
date: 2020-07-28 18:26:28 +0900
categories: Container
classes: wide
tags:
  - Kubernetes

---
## Prometheus
Kubernetes는 Cluster 단위로 구성이 되기 때문에 장애 발생시 문제 파악이 어렵습니다. 그래서 Kubrenetes에서는 모니터링 시스템이 중요합니다. 상용 제품을 제외한 오픈소스 솔루션 중에 Prometheus가 메인으로 주목 받고 있습니다.

### Metrics Pulling 방식 수집
일반적으로 다른 모니터링 시스템에서는 Target system의 Agent에서 Push 방식을 사용합니다. 하지만 Prometheus는 주기적으로 Exporter(target system)로부터 Metrics을 수집하는 Pulling 방식을 사용합니다. Push 방식은 Auto Scale 경우에 유리하지만 Pulling 경우 Target system이 가변적으로 변경될 경우, 대상의 IP Address를 알 수 없는 문제가 있습니다. 그래서 보완하기 위해 서비스 디스커버리를 지원하고 있습니다. 또한 Pulling 방식으로 인해 정확한 수치는 얻을 수 없습니다. Pulling 은 주기적으로 요청되기 때문에 근사치를 얻을 수 밖에 없습니다.

### 데이터 저장
Prometheus에서 수집된 정보는 메모리와 로컬 디스크에 저장됩니다. 구성이 간단하지만 스케일링이 어렵다는 단점을 가지고 있습니다. 만약 Prometheus을 스케일링하기 위해서는 구조상 고가용성이 불가능하여 샤딩 방식으로 구성해야합니다. 싱글로 구성할 시 저장 용량이 부족하면 디스크 용량을 늘리는 것 밖에 방안이 없습니다. 그리고 Prometheus Server Down시 기존 Metrics은 저장되지 않습니다.

