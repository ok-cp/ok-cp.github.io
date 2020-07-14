---
title: "Kubernetes Taint-based Evictions"
date: 2020-07-13 18:26:28 +0900
categories: Container
classes: wide
tags:
  - Kubernetes

---
## Taints & Tolerations 
Taints/Tolerations는 nodeAffinity와 동작은 유사하지만 3가지 용도로 사용 가능합니다.

* 전용 노드 : nodeselector와 taint/tolerations 조합을 사용하여 전용 노드를 만들 수 있습니다.
* 특정 하드웨어가 있는 노드 :  특정 하드웨어(예 : GPU)가 있는 노드가 있는 경우, 하드웨어가 필요하지 않은 Pod를 제외하고 필요한 Pod를 배치하려고합니다. 
* Taint-based Evictions : 노드에 문제가 있는 경우, Pod를 제거합니다.

## Taint-based Evictions 
Usecase 중에서 Taint-based Evictions에 대해 알아보도록 하겠습니다.

Kubernetes 1.12부터 TaintNodesByCondition이 Beta로 승격되었습니다. 기본적으로 true로 설정되므로 필요한 경우 --feature-gates 명령을 사용하여 비활성화해야합니다.
TaintNodesByCondition 활성화 된 경우 노드 컨트롤러는 노드 상태에 따라 자동으로 Taints를 추가합니다. 

현재 아래 conditions과 taints이 지원됩니다.
* node.kubernetes.io/out-of-disk conditions : 노드에 새 Pod를 추가하기위한 여유 공간이 충분하지 않으면 True입니다.
* node.kubernetes.io/unreachable : 노드가 정상 상태가 아니고 Unknown 상태인 경우, NodeCondition Ready 가 "Unknown"
* node.kubernetes.io/not-ready: 노드가 준비되지 않은 경우, NodeCondition Ready 가 "False"
* node.kubernetes.io/memory-pressure : 노드 메모리가 부족한 경우 True, 그렇지 않으면 False. 
* node.kubernetes.io/disk-pressure : 디스크 용량이 부족한 경우 True, 그렇지 않으면 False. 
* node.kubernetes.io/network-unavailable : 노드의 네트워크가 올바르게 그렇지 않으면 구성되지 않은 경우 True, 그렇지 않으면 False. 

### 실행중인 node conditions
다음 명령어를 입력하면 node conditions 을 확인할 수 있습니다.

```bash
$ kubectl get nodes -o jsonpath='{range .items[*]}{@.metadata.name}{"\n"}{range @.status.conditions[*]}{@.type}={@.status}{"\n"}{end}{end}'

node1

OutOfDisk=False
MemoryPressure=False
DiskPressure=False
Ready=True
PIDPressure=False
```


### tolerationSeconds
정해진 toleration 시간대로 Pod가 배치되지만 tolerationSeconds을 추가로 지정하면 사용자 정의대로 Pod가 노드에 바인딩된 상태로 유지된다.

```yaml
tolerations:
- key: "node.kubernetes.io/unreachable"
  operator: "Exists"
  effect: "NoExecute"
  tolerationSeconds: 6000
```
