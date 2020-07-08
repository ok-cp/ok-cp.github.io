---
title: "Kubernetes Scheduler"
date: 2020-06-27 18:26:28 +0900
categories: Container
classes: wide
tags:
  - Kubernetes

---
## Kubernetes Pod가 적절한 노드를 찾는 방법
Scheduler는 결정을 하기 위해 사용하는 알고리즘이 있습니다. 

1. Kubernetes Scheduler는 Nodename 매개 변수없는 새로운 Pod가 있음을 감지합니다. 
2. Scheduler는 Pod에 적합한 노드를 선택하고 노드 이름에 Pod를 업데이트합니다.
3. 선택된 노드의 kubelet 에 Pending Pod가 있음을 알립니다.
4. kubelet은 Pod를 실행하고 노드에서 실행합니다.

### Pod를 실행하기 위해서는 조건(우선순위)이 있습니다.
만약 높은 CPU, Memory 사용량의 Pod가 있다면 노드에 과부하가 걸릴 수 있습니다. 
Scheduler는 배치할 포드가 있으면 노드에 필요한 자원이 있는지 여부를 판별하게 됩니다.
Pod가 요청한 Memory가 충분하지 않은 노드에 Pod가 배포된 경우 호스팅된 응용 프로그램이 예기치 않게 작동하거나 심지어 충돌 할 수 있습니다.

만약, Pod를 배포할 노드를 선택하기 전에 Kubernetes Scheduler가 고려해야 할 요소가 너무 많을 경우에는 먼저 정상 상태의 모든 노드를 선택합니다.
그리고 predicate 테스트를 실행하여 적합하지 않은 노드를 필터링합니다. 그 다음 우선 순위 테스트를 실행합니다. 가장 높은 점수를 받은 노드를 선택하게 됩니다.
때때로 같은 점수를 가진 노드가 두 개 이상이면 Scheduler는 라운드 로빈 방식으로 노드를 선택하여 균등하게 분배합니다.


### NodeSelector
Scheduler의 결정보다 사용자 의사 결정이 필요한 경우에는  .spec.nodeSelector 이 필요합니다.
nodeSelector는 특정 레이블이 하나 이상있는 노드를 선택합니다. 

### NodeAffinity
nodeSelector는 단순하게 노드를 결정이 필요한 경우에는 유용하지만 다양한 선택 조건을 가질 수 없습니다. 
NodeAffinity는 이러한 다양한 조건을 충족하여 배포할 수 있습니다.
다음과 예제와 같이 feature = ssd Labels을 가진 노드에 배포할 수 있게 합니다.
```yaml
apiVersion: v1
kind: Pod
metadata:
 name: nginx
spec:
 containers:
 - name: nginx
   image: nginx
 affinity:
   nodeAffinity:
     requiredDuringSchedulingIgnoredDuringExecution:
       nodeSelectorTerms:
       - matchExpressions:
         - key: feature
           operator: In
           values:
           - ssd

```

#### RequiredDuringSchedulingIgnoredDuringExecution 옵션
matchExpressions labels 노드에서만 Pod를 실행합니다. nodeSelector와 동일한 동작입니다.

#### PreferredDuringSchedulingIgnoredDuringExecution 옵션
matchExpressions labels이 지정된 노드에서 Pod를 실행하려고 시도합니다. 
그러나 해당 노드를 사용할 수없는 경우 다음 최상의 노드에서 Pod를 실행하려고 시도합니다.


### Node Anti-Affinity
간혹 특정 Pod를 제외하고 하나 이상의 노드를 사용하지 않아야합니다. 모니터링 애플리케이션을 호스팅하는 노드를 생각해보십시오. 이러한 노드에는 역할 특성으로 인해 많은 리소스가 없어야합니다. 따라서 모니터링 앱이있는 포드 이외의 포드가 해당 노드에 예약 된 경우 모니터링이 손상되고 호스팅중인 애플리케이션이 저하됩니다. 이 경우 포드를 노드 세트에서 멀리 유지하려면 노드 반 선호도를 사용해야합니다. 다음은 반 선호도가 추가 된 이전 포드 정의입니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
 name: mongo
spec:
 affinity:
   nodeAffinity:
     requiredDuringSchedulingIgnoredDuringExecution:
       nodeSelectorTerms:
       - matchExpressions:
         - key: feature
           operator: In
           values:
           - ssd
           - eight-cores
         - key: role
           operator: NotIn
           values:
           - monitoring

 containers:
 - name: mongodb
   image: mogo
```


Adding another key to the matchExpressions with the operator NotIn will avoid scheduling the mongo pods on any node labelled role=monitoring.

Nodes Taints And Tolerations
While nodes anti-affinity patterns allow you to prevent pods from running on specific nodes, they suffer from a drawback: the pod definition must explicitly declare that it shouldn’t run on those nodes. So, what if a new member joins the development team, writes a Deployment for her application, but forgets to exclude the monitoring nodes from the target nodes? Kubernetes administrators need a way to repel pods from nodes without having to modify every pod definition. That’s the role of taints and tolerations.

When you taint a node, it is automatically excluded from pod scheduling. When the schedule runs the predicate tests on a tainted node, they’ll fail unless the pod has toleration for that node. For example, let’s taint the monitoring node, mon01:

kubectl taint nodes mon01 role=monitoring:NoSchedule
Now, for a pod to run on this node, it must have a toleration for it. For example, the following .spec.toleration:

```yaml
tolerations:
- key: "role"
  operator: "Equal"
  value: "monitoring"
  effect: "NoSchedule"
```

matches the key, value, and effect of the taint on mon01. This means that mon01 will pass the predicate test when the Scheduler decides whether or not it can use it for deploying this pod.

An important thing to notice, though, is that tolerations may enable a tainted node to accept a pod but it does not guarantee that this pod runs on that specific node. In other words, the tainted node mon01 will be considered as one of the candidates for running our pod. However, if another node has a higher priority score, it will be chosen instead. For situations like this, you need to combine the toleration with nodeSelector or node affinity parameters.

TL;DR
The Kubernetes Scheduler is the component in charge of determining which node is most suitable for running pods.
It does that using two main decision-making processes:
Predicates: which are a set of tests, each of them qualifies to true or false. A node that fails the predicate is excluded from the process.
Priorities: where each node is tested against some functions that give it a score. The node with the highest score is chosen for pod deployment.
The Kubernetes Scheduler also honors user-defined factors that affect its decision:
Node Selector: the .spec.nodeSelector parameter in the pod definition narrows down node selection to those having the labels defined in the nodeSelector.
Node affinity and anti-affinity: those are used for greater flexibility in node selection as they allow for more expressive selection criteria. Node affinity can be used to guarantee that only the matching nodes are used or only to set a preference.
Taints and tolerations work in the same manner as node affinity. However, their default action is to repel pods from the tainted nodes unless the pods have the necessary tolerations (which are just a key, a value, and an effect). Tolerations are often combined with node affinity or node selector parameters to guarantee that only the matched nodes are used for pod scheduling.