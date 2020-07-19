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
Nginx Ingress Controller는 크게 kubernetes, Nginx 에서 제공하는 Ingress로 나뉩니다. 기능별 차이가 있으며 Nginx에서 제공하는 Ingress는 Plus 와도 기능 차이를 가지고 있습니다. 테스트는 kubernetes Ingress Controller로 진행하기로 합니다.

* <a href="https://github.com/nginxinc/kubernetes-ingress">nginxinc/kubernetes-ingress on GitHub</a>
* <a href="https://github.com/kubernetes/ingress-nginx">kubernetes/ingress-nginx on GitHub</a> 
* <a href="https://docs.google.com/spreadsheets/d/1DnsHtdHbxjvHmxvlu7VhzWcWgLAn_Mc5L1WlhLDA__k/edit#gid=0">INgress별 기능비교 </a>


## Nginx Ingress Traffic Monitoring
iptraf를 통해서 요청이 어떻게 들어오는 지 확인해보기로 했습니다.
nginx 이미지를 배포하였고 Pod ip는 192.168.125.70, ClusterIP는  10.100.95.61 입니다. 그리고 Nginx Ingress Controller Pod ip는 192.168.125.68 입니다.

![nginx-ingress-1]({{ site.url }}{{ site.baseurl }}/assets/images/nginx-ingress-1.png)

### Nginx Ingress & Node Port 비교 
10.10.180.11 은 ClientIP 이고 port 30303은 node port입니다.

![nginx-ingress-3]({{ site.url }}{{ site.baseurl }}/assets/images/nginx-ingress-3.png)

port 31806은 nginx ingress controller port 입니다.

![nginx-ingress-2]({{ site.url }}{{ site.baseurl }}/assets/images/nginx-ingress-2.png)

예상했던 것과 다르게 Ingress/NodePort 모두 Pod IP를 바로 찾아가고 있습니다. 다만 Ingress는 Nginx Ingress Controller Pod 에 먼저 요청하고 있습니다.

## Tunnel Interface
Calico cni를 사용하는 구성에서 Pod가 배치된 Node가 다른 NodeIP로 요청을 보낸경우 어떻게 트래픽이 전달되는지 확인해보았습니다.

## NodePort
#### worker1
192.168.149.2 는 tunnel interface 입니다. Client IP -> Node IP -> tunnel -> pod
![nginx-ingress-4]({{ site.url }}{{ site.baseurl }}/assets/images/nginx-ingress-4.png)

#### worker2
tunnel -> Pod
![nginx-ingress-5]({{ site.url }}{{ site.baseurl }}/assets/images/nginx-ingress-5.png)


## Ingress Controller
#### worker1
192.168.149.2 는 tunnel interface 입니다. Client IP -> Node IP -> tunnel -> Ingress pod
![nginx-ingress-6]({{ site.url }}{{ site.baseurl }}/assets/images/nginx-ingress-6.png)

#### worker2
tunnel -> Ingress pod -> Pod
![nginx-ingress-7]({{ site.url }}{{ site.baseurl }}/assets/images/nginx-ingress-7.png)

### 결론
ingress/nodeport 상관없이 clusterip 바라보지 않고 바로 트래픽을 전송하고 있었습니다.

### 참조
* https://www.joyfulbikeshedding.com/blog/2018-03-26-studying-the-kubernetes-ingress-system.html