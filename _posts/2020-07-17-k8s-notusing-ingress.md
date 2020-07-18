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

## Nginx Ingress Controller
Nginx Ingress Controller는 크게 kubernetes, Nginx 에서 제공하는 Ingress로 나뉩니다. 기능별 차이가 있으며 Nginx에서 제공하는 Ingress는 Plus 와도 기능 차이를 가지고 있습니다.

* nginxinc/kubernetes-ingress on GitHub  https://github.com/nginxinc/kubernetes-ingress
* kubernetes/ingress-nginx on GitHub  https://github.com/kubernetes/ingress-nginx

INgress별 기능비교 
https://docs.google.com/spreadsheets/d/1DnsHtdHbxjvHmxvlu7VhzWcWgLAn_Mc5L1WlhLDA__k/edit#gid=0

## iptraf
iptraf를 통해서 요청이 어떻게 들어오는 지 확인해보기로 했습니다.


### 참조
* https://www.joyfulbikeshedding.com/blog/2018-03-26-studying-the-kubernetes-ingress-system.html