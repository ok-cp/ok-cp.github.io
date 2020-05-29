---
title: "Calico 최적화"
date: 2020-05-27 08:26:28 -0400
categories: Kubernetes
tags:
  - kubernetes 
  - cni
  - calico
---
## Calico 최적화
Kubernetes CNI 중 대표적인 Calico는 Vxlan,IPinIP를 지원하며 IP Pool 설정이 가능하고 다른 CNI에 비해 성능과 안정성이 보장되기 때문에 사랑받고 있습니다.
Calico는 초기 설정 그대로 사용하여도 무방하나 조금 더 나은 성능을 유지하기 위해 최적화하는 방법에 대해 알아보겠습니다.

### MTU
성능을 올리기 위한 방법 중에 가장 간단한 방법은 Calico 최대 전송 단위 (MTU)를 올리는 방법입니다.
노드의 허용된 MTU에 한해 Calico MTU를 늘릴 수 있습니다.

calico-config configmap을 수정하여 변경할 수 있습니다.
```yaml
kubectl edit cm calico-config -n kube-system
```

mtu또는 veth_mtu의 값을 변경해줍니다.
```yaml
   veth_mtu: "1480"
```
저장 후 모든 Calico pod를 재기동합니다.

### IPVS mode
kubernetes에서는 트래픽을 제어하는 기술 3가지(userspace, iptables, IPVS)를 지원합니다.
userspace는 오래되고 느리기때문에 사용되고 있지않습니다. 그래서 iptables, IPVS 중 선택해야합니다.

iptables는 다양한 일반적인 패킷 조작 및 필터링 요구를 처리 할 수있는 충분한 유연성을 갖춘 효율적인 방화벽으로 설계된 Linux 커널 기능입니다.
IPVS는 로드 밸런싱을 위해 설계된 Linux 커널 기능입니다. 로드 밸런서로서 라운드 로빈, 최단 지연, 최소 연결 및 다양한 해싱 접근 방식과 같은 여러 가지 예약 알고리즘을 가지고 있습니다.
반대로 iptables의 kube-proxy는 무작위로 동일한 비용 선택 알고리즘을 사용합니다.

기능뿐만 아니라 IPVS는 iptables에 비해 Round-Trip Response Times, CPU 성능이 뛰어납니다.
성능 해택은 10,000개 이상 서비스 이상 넘어갈 때 볼 수 있으므로 클러스터 환경에 따라 설정하면 되겠습니다.
https://www.projectcalico.org/comparing-kube-proxy-modes-iptables-or-ipvs/

kube-proxy는 기본 모드가 iptables 이기 때문에 configmap을 통해서 ipvs모드로 변경할 수 있습니다.
```yaml
kubectl edit configmap kube-proxy -n kube-system
```

mode에서 ipvs로 변경한 뒤 kube-proxy pod를 재기동합니다.
```yaml
mode: ipvs
```



### IP in IP mode 
Calico는 Vxlan, IPinIP 중에 선택하여 설치할 수 있지만 기본 IPinIP로 설치되어있습니다.
Vxlan heaer는 IPinIP header보다 사이즈가 크기때문에 Vxlan은 IPinIP보다 오버헤드가 있습니다.
물론 multicast를 지원하면 L7 기능을 지원하기 때문에 장점도 가지고 있습니다. 

기본 설치되는 IPinIP 또한 IP 안에 다른 IP header를 가지고 있는 구조이기 때문에 
Multi subnet을 지원한다는 장점이 있지만 Multi subnet이 굳이 필요없는 환경이라면 IPinIP는 불필요합니다.

그래서 IPinIP를 On/Off 할수 있으며 daemonset의 env 중에 CALICO_IPV4POOL_IPIP을 "Off" 또는 "Never"로 설정하면 기능이 비활성화됩니다.
```yaml
kubectl edit ds calico-node  -n kube-system
```

```yaml
        - name: CALICO_IPV4POOL_IPIP
          value: Never
```
변경되면 모든 Calico pod가 재기동됩니다.

### portmap plug-in

portmap plugin은  hostPort출 할 수 있습니다. 클러스터의 Calico CNI 구성에서 포트 맵 플러그인을 제거하여 iptables 성능 문제를 방지하십시오.

클러스터에 500 개 이상의 서비스와 같은 많은 서비스가 있거나 10 개 이상의 서비스에 대해 서비스 당 50 개 이상의 포트와 같은 서비스의 많은 포트가있는 경우 많은 iptables 규칙이 생성됩니다. 이러한 서비스에 대한 Calico 및 Kubernetes 네트워크 정책 많은 수의 iptables 규칙으로 인해 포트 맵 플러그인의 성능 문제가 발생할 수 있으며 iptables 규칙의 향후 업데이트를 막거나 calico-node 지정된 시간 내에 iptables 규칙을 업데이트 할 잠금이 수신되지 않으면 컨테이너가 다시 시작될 수 있습니다. 이러한 성능 문제를 방지하기 위해 클러스터의 Calico CNI 구성에서 포트 맵 플러그인을 제거하여 포트 맵 플러그인을 비활성화 할 수 있습니다.

portmap plugin을 비활성화하면 hostPorts를 사용할 수 없습니다.

포트 맵 플러그인을 비활성화하려면 구성 calico-config맵 자원을 편집하십시오 .

```yaml
kubectl edit cm calico-config -n kube-system
```

portmap 구문을 삭제하고 calico pod를 모두 재기동합니다.
```yaml
        {
          "type": "portmap",
          "snat": true,
          "capabilities": {"portMappings": true}
        },
```


```yaml
apiVersion: v1
data:
  calico_backend: bird
  cni_network_config: |-
    {
      "name": "k8s-pod-network",
      "cniVersion": "0.3.1",
      "plugins": [
        {
          "type": "calico",
          "log_level": "info",
          "datastore_type": "kubernetes",
          "nodename": "__KUBERNETES_NODE_NAME__",
          "mtu": __CNI_MTU__,
          "ipam": {
              "type": "calico-ipam"
          },
          "policy": {
              "type": "k8s"
          },
          "kubernetes": {
              "kubeconfig": "__KUBECONFIG_FILEPATH__"
          }
        },
        {
          "type": "bandwidth",
          "capabilities": {"bandwidth": true}
        }
      ]
    }
```