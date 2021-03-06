---
title: "bridge-nf-call-iptables contents are not set to 1"
date: 2020-06-22 18:26:28 +0900
categories: Container
classes: wide
tags:
  - Kubernetes
  - Docker

---
## /proc/sys/net/bridge/bridge-nf-call-iptables contents are not set to 1
간혹 Kubernetes 부트스트랩 과정에서 다음과 같은 에러가 발생하는 경우가 생깁니다.

```bash
[root@localhost ~]# kubeadm init
[kubeadm] WARNING: kubeadm is in beta, please do not use it for production clusters.
[init] Using Kubernetes version: v1.6.4
[init] Using Authorization mode: RBAC
[preflight] Running pre-flight checks
[preflight] Some fatal errors occurred:
    /proc/sys/net/bridge/bridge-nf-call-iptables contents are not set to 1
[preflight] If you know what you are doing, you can skip pre-flight checks with `--skip-preflight-checks`
```

### bridge-nf-call-iptable
bridge-nf-call-iptable 값을 1로 설정하라는 내용인데 bridge network에서 송수신되는 패킷이 iptables 설정에 따라 제어할 것인지 정하는 옵션입니다.
CentOS/RHEL 경우에 기본값은 0으로 되어있으며 off 상태입니다. 

문제는 이 에러가 동일한 이미지의 VM을 여러대 생성해서 Kubernetes를 설치해보면 위와 같은 문제가 발생할 때도 있고 넘어갈 때도 있습니다.
컨테이너를 사용한다면 당연히 1로 기능이 활성화되어있는 상태여야하는데 0인 상태에서 정상적으로 설치가 되고 통신에도 문제 없는 것을 볼 수 있습니다.
찾아보니 일련의 버그로 보이며 컨테이너(Docker, CRI-O) 사용시 선행 설정이 꼭 필요한 것으로 보입니다.

```bash
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
```