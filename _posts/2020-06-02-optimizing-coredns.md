---
title: "Coredns 최적화"
date: 2020-06-02 08:26:28 -0400
categories: Kubernetes
tags:
  - kubernetes 
  - coredns
---
## Coredns 최적화
kubernetes 1.13 부터 dns를 담당하는 kube-dns가 coredns로 변경되었습니다. kuberenetes 설치시 기본설정의 coredns를 사용하여도 초기에는 문제없을 수 있으나 pod와 service가 늘어날 수록 coredns에서 필요한 cpu, memory 사용량은 늘어나기 때문에 먼저 대비하는 것이 좋습니다.

### Memory & Pod
대규모 Kubernetes 클러스터에서 CoreDNS의 메모리 사용량은 주로 클러스터의 포드 및 서비스 수에 영향을 받습니다. 
또한 DNS 응답 캐시의 크기 및 CoreDNS 인스턴스 당 수신 된 쿼리 속도 (QPS)가 있습니다.

#### CoreDNS 설정
CoreDNS 인스턴스에 필요한 메모리 양을 계산할 수 있습니다.

```yaml
필요 메모리 MB = (Pods + Services) / 1000 + 54
```

### autopath 플러그인 사용시
플러그인을 활성화하면 더 많은 메모리가 필요합니다. autopath 플러그인을 사용하면 pod에 대한 모든 변경 사항을 모니터링해야하므로 Kubernetes API에 추가로드가 발생합니다.

autopath 플러그인을 사용하여 CoreDNS 인스턴스에 필요한 메모리 양을 계산할 수 있습니다.

```yaml
필요 메모리 MB = (Pods + Services) / 250 + 56
```