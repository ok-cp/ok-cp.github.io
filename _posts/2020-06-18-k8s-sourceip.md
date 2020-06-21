---
title: "Kubernetes에서 Source IP 사용하기"
date: 2020-06-18 18:26:28 +0900
categories: Container
classes: wide
tags:
  - Container

---
## NodePort
Kubernetes 1.5부터 Type = NodePort를 사용하여 Services로 전송 된 패킷은 기본적으로 소스 NAT입니다.

NodePort가 통신하는 방식은 다음과 같습니다.

Clinet가 node2:nodePort로 패킷을 보낸다.
node2는 패킷의 소스 IP 주소 (SNAT)를 자체 IP 주소로 바꿉니다.
패킷이 node1로 라우팅 된 다음 엔드 포인트로 라우팅됩니다.
Pod의 응답을 node2로 다시 라우팅됩니다.
Pod의 응답을 클라이언트로 다시 전송됩니다.

```bash
          client
             \ ^
              \ \
               v \
   node 1 <--- node 2
    | ^   SNAT
    | |   --->
    v |
 endpoint
```

## NodePort에서 Source IP 사용하기
Kubernetes에는 클라이언트 Source IP를 보존할 수 있습니다. service.spec.externalTrafficPolicy 값을 Local 로 설정하면 로컬 엔드 포인트에 대한 요청 만 프록시하고 트래픽을 다른 노드로 전달하지 않으므로 원래 소스 IP 주소를 유지합니다. 만약 로컬 엔드 포인트가 없다면 노드에 보낸 패킷은 드랍됩니다. 패킷 처리 규칙에서 올바른 Source IP에 의존 할 수 있으므로 패킷을 끝점까지 전달할 수 있습니다.

service.spec.externalTrafficPolicy 값이 Local 로 설정된 경우, NodePort 통신방식은 다음과 같습니다.

Clinet가 node2:nodePort로 패킷을 보낸다.
Endpoint가 없으므로 패킷은 드랍된다.
Clinet가 node1:nodePort에 패킷을 보낸다.
node1은 패킷을 올바른 source IP로 엔드 포인트로 라우팅합니다.

```bash
        client
       ^ /   \
      / /     \
     / v       X
   node 1     node 2
    ^ |
    | |
    | v
 endpoint
``` 

## Weave-net 이슈
weave에서 service.spec.externalTrafficPolicy 의 Local 을 지원하지 못하는 문제 발생합니다. weave env에 NO_MASQ_LOCAL  값을 “1”로 설정하면 사용할 수 있습니다. 기본값은 0

https://github.com/weaveworks/weave/issues/2924#issuecomment-407827430

## ExternalIP 이슈
service.spec.externalIP를 지정하는 경우, NodePort를 사용하지 않고 ServicePort를 사용하면 SourceIP를 찾아올 수 없습니다. 이유는 externalIP 의 service port를 사용하게 되면 Service externalIP를 바라보며 SourceIP은 꼭 NodePort를 사용해야 Ingress Traffic이 도착합니다.

## 참고

https://kubernetes.io/docs/tutorials/services/source-ip/