---
title: "Kubernetes node resource 관리"
date: 2020-07-09 18:26:28 +0900
categories: Container
classes: wide
tags:
  - Kubernetes

---
## kubelet 
Kubernetes의 주요 작업을 수행하는 노드 구성 요소입니다. 

  * kube-apiserver에 노드 등록
  * kube-apiserver 모니터링하여 새로운 Pod가 가 schedule된 후 Container를 시작하도록 container runtime에 알리기
  * 컨테이너 실행 모니터링 및 해당 상태를 kube-apiserver에 보고
  * liveness probes 실행하고 Container가 실패한 경우 재시작
  * kubelet이 관리하는 static Pod를 실행합니다.
  * Core Metrics 파이프 라인 및 container runtime과 상호 작용하여 container 및 node metrics을 수집

## eviction signals, eviction thresholds
그리고 kubelet은 CPU/Memory/Disk와 같은 컴퓨팅 자원이 부족할 때 노드 안정성을 유지하는 중요한 역할을 합니다. 
kubelet은 eviction signals 및 eviction thresholds을 기반으로 리소스를 회수 시기를 결정 합니다. 
eviction signals은 Memory, Disk와 같은 시스템 리소스의 현재 용량입니다.
eviction thresholds은 kubelet에 의해 유지되어야하는 리소스의 최소값입니다.
eviction signals 및 eviction thresholds는 시스템 리소스를 되찾기 시작하는 시점을 kubelet에 알려주게 됩니다.

  * memory.available : 클러스터 메모리의 상태 신호입니다. 메모리의 기본 eviction threshold은 100 Mi입니다. 즉, 여유 메모리가 100 Mi로 내려 가면 kubelet이 Pod를 제거하기 시작합니다.
  * nodefs.available : nodefs는 볼륨, 데몬 로그 등을 위해 kubelet에서 사용하는 파일 시스템입니다. 기본적으로 kubelet은 nodefs.available < 10 % 인 경우 노드 리소스를 회수하기 시작합니다.
  * nodefs.inodesFree : nodefs inode 메모리의 상태를 설명하는 신호. 기본적으로 kubelet은 nodefs.inodesFree < 5% 인 경우 제거를 시작합니다.
  * imagefs.available : imagefs 파일 시스템은 container image와 container 쓰기 가능 레이어를 저장하기 위해 container runtime에서 사용하는 파일 시스템입니다. 기본적으로 kubelet은 imagefs.available < 15% 인 경우 제거를 시작합니다 .
  * imagefs.inodesFree : imagefs inode 메모리의 상태입니다. 기본 eviction threshold 값은  없습니다.

## hard/soft eviction thresholds
Kubernetes는 hard,soft eviction thresholds을 지원합니다.

#### hard eviction thresholds
hard eviction thresholds 값에 도달하면 kubelet은 유예 기간없이 즉시 리소스를 회수하기 시작합니다. 
  * --eviction-hard : hard eviction threshold 값을 정의 할 수 있습니다. 
  
```yaml
--eviction-hard=memory.available<1Gi
노드 memory.available가 1Gi 미만일 때 kubelet에 자원을 회수하도록 지시합니다.
```

#### soft eviction thresholds
반대로 soft eviction thresholds 값에는 kubelet이 리소스를 회수하기 전에 만료되어야하는 사용자 정의 유예 기간이 포함됩니다.
  * --eviction-soft & --eviction-soft-grace-period : soft eviction threshold 값을 정의 할 수 있습니다. 

```yaml
--eviction-soft=memory.available<2Gi --eviction-soft-grace-period=1m30s 
eviction threshold 값을 90초간 제거를 유예합니다. --eviction-max-pod-grace-periodin 으로 허용되는 최대 유예 기간을 지정할 수도 있습니다.
```


## kubelet 리소스 회수대상 선정
kubelet은 기본적으로 Pod를 제거하여 리소스를 회수합니다. 먼저 사용하지 않은 container image 또는 dead pod 같은 리소스를 회수하려고 시도합니다.

### nodefs, imagefs eviction threshold 
nodefs eviction threshold 값에 도달하면 kubelet은 모든 죽은 포드 및 해당 container를 삭제합니다. 
imagefs eviction threshold 값에 도달하면 kubelet은 먼저 모든 죽은 Pod와 해당 container를 삭제한 다음 사용하지 않은 모든 이미지를 제거합니다.

만약, container image, Pod 및 기타 리소스를 회수하여도 리소스가 부족한 경우가 발생한다면 kubelet은 사용자가 실행한 Pod를 삭제하기 시작합니다. 
kubelet은 Pod QoS, Pod Priority 및 Guaranteed, Burstable, Best-Effort 기반으로 어떤 사용자 Pod를 제거할지 결정합니다.

### Guaranteed, Burstable, Best-Effort
  * Guaranteed : 모든 Container에서 CPU 및 Memory 모두에 대해 리소스 제한 및 요청이 설정되는 포드입니다.
  * Burstable : 하나 이상의 Container에 대한 하나 이상의 리소스 (예 : CPU, Memory)에 대한 requests, limits이 설정되고 동일하지 않은 포드입니다.
  * Best-Effort : 리소스가 설정되지 않은 Pod입니다.


### QoS model

  * resource requests 초과된 경우 : Kubernetes에서 Pod는 limits이 아니라 requests에 따라 배치됩니다. 따라서 모든 Pod는 requests CPU/Memory를 보장합니다. 그러나 limits 설정되어 있지 않고 Pod가 resource requests을 초과한 경우 보장된 Pod 또는 일부 시스템 작업에 제한된 리소스가 필요한 경우에는 해당 Pod가 종료되거나 제한될 수 있습니다. 하지만 resource requests보다 적게 소비하는 Pod조차도 특정 상황에서 종료될 수 있습니다. (시스템 작업 메모리가 매우 부족하고 Pod priority가 낮은 Pod가 종료되지 않은 경우)

  * Pod priority : requests를 초과한 Pod가 없는 경우 kubelet은 Pod priority를 확인합니다. priority를가 낮은 Pod를 먼저 제거하려고 시도합니다.


## Pod 제거 순서
Pod QoS, Pod Priority 및 Guaranteed, Burstable, Best-Effort 규칙이 적용되면 kubelet은 사용자 Pod를 다음 순서로 제거합니다.

먼저 resource requests을 초과하는 Best-Effort 그리고/또는 Burstable Pod입니다. 이러한 Pod가 여러 개인 경우 kubelet은 우선 priority 따라 순위를 정한 다음 resource requests 대비 소비에 따라 제거합니다.
requests 미만 소비되는 Guaranteed, Burstable Pod는 마지막으로 제거됩니다. 
그러나 노드에 Best-Effort Pod가 없는 경우 requests 미만으로 소비되는 Guaranteed Pod를 제거할 수 있습니다. 이 경우 priority가 가장 낮은 Guaranteed 그리고/또는 Burstable Pod를 먼저 제거합니다.


## 최소 Eviction 회수
kubelet이 회수하는 자원의 양이 적으면 시스템은 반복적으로 eviction thresholds에 도달할 수 있습니다. 
이러한 시나리오를 피하기 위해 --eviction-minimum-reclaim 를 사용하여 자원별 최소 회수레벨을 설정할 수 있습니다 .

```yaml
--eviction-hard=memory.available<1Gi, nodefs.available<2Gi, imagefs.available<200Gi
--eviction-minimum-reclaim=memory.available=0Mi, nodefs.available=1Gi, imagefs.available=2Gi
```

  * --eviction-minimum-reclaim : nodefs 스토리지의 최소​용량이 3Gi이고 imagefs  스토리지의 최소​​용량이 202Gi가 되도록합니다. 따라서, 위의 구성은 시스템이 제거 임계 값에 매우 자주 도달하지 않도록 충분한 자원을 확보 할 수 있도록합니다.
  * eviction-pressure-transition-period : 유예 기간이 긴 soft eviction thresholds을 사용하면 eviction 불확실성이 발생하여 Schedule이 잘못 될 수 있습니다. 이 문제를 피하기 위해, 제거 조건을 충족하기 전에 kubelet이 얼마나 대기해야하는지 정할 수 있습니다.


## Out-of-Resource Handling
만약 Node Memory만 고려하여 Memory 용량이 10Gi인 경우에 kernel, kubelet, Docker 등과 같은 시스템 데몬에 대해 총 메모리의 10 %를 예약하려고합니다. 또한 메모리 사용률의 95%에서 Pod를 제거하려고합니다.
다음 조건을 반영하기 위해서는 kubelet에 플래그를 설정해야합니다.
최소한 시스템 데몬이 실행되기 위한 Memory는 10%인 1Gi 이고 여기에 eviction threshold .5Gi를 더하여 system-reserved=memory=1.5Gi 을 설정합니다.
```yaml
kubelet : 
  eviction-hard=memory.available<500Mi 
  system-reserved=memory=1.5Gi
```

## Conclusion
관리자는 eviction thresholds, eviction grace periods 을 설정하여 노드 안정성을 고려하여 조건을 결정할 수 있습니다. 
그러나 eviction thresholds을 너무 높게 설정하거나eviction grace periods을 너무 길게 설정할 때는 주의해야합니다.