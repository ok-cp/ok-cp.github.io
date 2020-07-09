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
반대로 특정 노드를 피해서 Pod를 배포하는 경우도 있습니다.
예를 들어 모니터링 애플리케이션은 많은 리소스를 사용하기 때문에 모니터링 앱과 서비스앱이 같은 노드에 있을 경우, 리소스 경쟁이 발생하며 pod가 정상적으로 실행하지 못할 수 있습니다.
operator: NotIn 을 사용하여 role=monitoring 을 가진 노드를 제외하고 Pod를 배포할 수 있습니다.

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
         - key: role
           operator: NotIn
           values:
           - monitoring
```

###  Nodes Taints And Tolerations
Node Anti-Affinity 패턴을 사용하면 특정 노드에서 Pod가 실행되는 것을 방지할 수 있지만 실행하는 Pod마다 명시적으로 선언해야합니다. 
Tolerations은 비슷하지만 노드에 taint 실행을 하여 처음부터 다른 Pod가 실행되는 것을 방지합니다. 그리고 tolerations 을 명시한 Pod만 스케쥴합니다.

먼저 노드에 taint를 실행합니다.
```bash
$ kubectl taint nodes infra-node1 role=monitoring:NoSchedule
```

pod애 spec.tolerations 를 명시하면 role=monitoring taint를 가진 노드에 Pod가 실행되게 됩니댜.

```yaml
tolerations:
- key: "role"
  operator: "Equal"
  value: "monitoring"
  effect: "NoSchedule"
```

하지만 꼭 infra-node에 먼저 Pod가 배치되는 것은 아니며 우선순위가 높을 뿐입니다. 꼭 infra-node에 Pod를 실행해야한다면 NodeSelector나 nodeAffinity를 선택하여 명시해야합니다.

## Conclusion
Kubernetes Scheduler는 Pod 실행에 가장 적합한 노드를 결정하는 구성 요소입니다.

의사 결정 프로세스는 두 가지 조건이 있습니다.
  * Predicates : 테스트 세트이며, 각각의 테스트는 true 또는 false입니다. Predicates에 실패한 노드는 프로세스에서 제외됩니다.
  * Priorities(우선 순위) : 각 노드는 점수를주는 일부 기능에 대해 테스트됩니다. 포드 배포에는 점수가 가장 높은 노드가 선택됩니다.

Kubernetes Scheduler는 사용자 정의로 인해 가장 적합한 노드를 결정할 수도 있습니다.
  * Node Selector: Pod .spec.nodeSelector 에 정의 된 Labels로 결정합니다.
  * Node affinity, anti-affinity : nodeSelector 보다 노드 선택의 유연성을 가지고 있습니다. Node affinity에 일치하는 노드만 사용하거나 상황에 따른 우선순위가 높은 다른 노드를 선택할 수 있습니다.
  * Taints, tolerations : 동작은 비슷하지만 노드에서 taint 조건(key, value, effect)을 가지고 있지 않은 Pod는 스케쥴에서 제외합니다.