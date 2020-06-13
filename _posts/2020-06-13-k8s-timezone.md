---
title: "Kubernetes timezonne 동기화"
date: 2020-06-13 18:26:28 +0900
categories: Kubernetes
classes: wide
tags:
  - Container
  - network
  - namespaces
---
Kubernetes cluster 구성은 매일하는 것이 아니기 때문에 설치할 때마다 필수 체크 요소를 놓치는 경우가 많습니다.
디스크 용량 확인이 제일 중요하고 그 다음은 클러스터를 구성하는 노드의 timezone 동기화입니다.
디스크 용량 사용률을 파악하지 못한 경우는 대개 클러스터가 구성되고 나서 한참 뒤 이슈가 발견됩니다.
하지만 timezone 동기화는 클러스터가 구성되고 바로 이슈가 발생됩니다.

### 증상
1. 클러스터 구성 초반에는 원활하게 통신이 가능하나 얼마지나지 않아 클러스터간 통신이 되지 않는다.
2. 노드에서 kube-apiserver port가 SYN_SENT로 걸려있다.
3. nodeport를 사용할 수 없으며 cni에서도 별다른 에러 메세지가 보이지 않는다.
4. master 노드에서 docker, kubelet을 재시작하면 다른 node가 NotReady 상태로 바뀌며 다시 상태가 돌아오지 않는다.

### 원인
master와 node 의 시간동기화가 되지않아 발생하는 문제입니다.
kubennetes 클러스터 구성시 생성되는 인증서때문에 각노드의 시간 동기화가 필수적입니다.

### chrony
rhel 7부터 ntp 대신 chrony를 사용하고 있다.
chrony는 ntp보다 빠르게 동기화하며 간헐적으로 네트워크 문제가 발생하더라도 동기화합니다.