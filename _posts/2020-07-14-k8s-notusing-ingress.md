---
title: "Not using Services - Ingress"
date: 2020-07-17 18:26:28 +0900
categories: Container
classes: wide
tags:
  - Kubernetes

---
## Not using Services
지금까지 Nginx Ingress Controller는 Service(Cluster IP)를 거쳐 Pod IP로 전달되는 것으로 알고 있었는데 잘못 알고 있었음을 발견하고 테스트를 해보기로 했습니다.


## iptraf
iptraf를 통해서 요청이 어떻게 들어오는 지 확인해보기로 했습니다.

### 참조
* https://www.joyfulbikeshedding.com/blog/2018-03-26-studying-the-kubernetes-ingress-system.html