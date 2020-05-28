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


### MTU

### ipvs mode

### IP in IP mode 

### portmap plug-in

portmap plugin은 네트워크 인터페이스를위한 플러그인 (CNI)은 hostPort출 할 수 있습니다. 클러스터의 Calico CNI 구성에서 포트 맵 플러그인을 제거하여 iptables 성능 문제를 방지하십시오.

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