---
title: "Kubernetes Pause Container"
date: 2020-07-17 18:26:28 +0900
categories: Container
classes: wide
tags:
  - Kubernetes

---


## Containerd 1.1-CRI Plugin
Containerd 1.1에서는 cri-containerd 데몬이 컨테이너 CRI 플러그인으로 리팩토링되었습니다. CRI 플러그인은 Containerd 1.1에 내장되어 있으며 기본적으로 활성화되어 있습니다. cri-containerd와 달리 CRI 플러그인은 직접 함수 호출을 통해 Containerd와 상호 작용합니다. 이 새로운 아키텍처는 통합을보다 안정적이고 효율적으로 만들고 스택에서 다른 grpc 홉을 제거합니다. 이제 Containerd 1.1과 함께 Kubernetes를 직접 사용할 수 있습니다. cri-containerd 데몬은 더 이상 필요하지 않습니다.

![containerd]({{ site.url }}{{ site.baseurl }}/assets/images/containerd.png)

## 성능
성능 향상은 Containerd 1.1 릴리스의 주요 초점 항목 중 하나입니다. Pod Startup Latency 및 데몬 리소스 사용 측면에서 성능이 최적화되었습니다.
다음 결과는 Containerd 1.1와 Docker 18.03 CE를 비교 한 것입니다. Containerd 1.1 통합은 컨테이너에 내장된 CRI 플러그인을 사용합니다. Docker 18.03 CE 통합은 dockershim을 사용합니다.


### Pod Startup Latency 
Containerd 1.1 가 dockerhim과 Docker 18.03 CE 통합보다 Pod Startup Latency이 낮음을 나타냅니다.

![latency]({{ site.url }}{{ site.baseurl }}/assets/images/20200714-latency.png)


### CPU and Memory
Containerd 1.1은 dockerhim과 Docker 18.03 CE 통합에 비해 전체적으로 CPU와 memory를 덜 소비합니다. 결과는 노드에서 실행중인 포드 수에 따라 다릅니다. 
아래 그림과 같이 dockerhim과의 Docker 18.03 CE 통합과 비교할 때 Containerd 1.1 은 kubelet CPU 사용량이 30.89 % 감소하고 컨테이너 런타임 CPU 사용량이 68.13 % 감소하고 kubelet 레지던트 세트 크기 (RSS) 메모리 사용량이 11.30 % 감소했습니다. 12.78 컨테이너 런타임 RSS 메모리 사용량이 낮습니다.

![cpu]({{ site.url }}{{ site.baseurl }}/assets/images/20200714-cpu.png)
![memory]({{ site.url }}{{ site.baseurl }}/assets/images/20200714-memory.png)


## crictl
Container runtime command-line interface(CLI)는 시스템 및 응용 프로그램 문제 해결에 유용한 도구입니다. Kubernetes의 컨테이너 런타임으로 Docker를 사용하는 경우 시스템 관리자는 때때로 Kubernetes 노드에 로그인하여 시스템 및 / 또는 응용 프로그램 정보를 수집하기위한 Docker 명령을 실행합니다. 예를 들어, docker ps 및 docker inspect 를 사용하여 응용 프로그램 프로세스 상태를 확인하고 docker 이미지 를 노드에 이미지를 나열하고 docker info 를 사용하여 컨테이너 런타임 구성을 식별할 수 있습니다.

crictl는 docker CLI와 유사한 경험을 제공하는 도구입니다.
모든 CRI 호환 컨테이너 런타임에서 일관되게 작동합니다. kubernetes-incubator / cri-tools 저장소에 호스팅되며 현재 버전은 v1.0.0-beta.1 입니다. crictl 은 Docker CLI와 유사하게 설계되어 사용자에게 더 나은 전환 환경을 제공하지만 정확히 동일하지는 않습니다.

### Docker CLI vs crictrl 
Docker CLI에는 pod 및 namespace 같은 핵심 Kubernetes 개념이 없으므로 Container 및 Pod를 명확하게 볼 수 없습니다. 
crictl 은 Kubernetes를 위해 설계되었습니다. 예를 들어 crictl pods 는 pod 정보를 나열하고 crictl ps 는 애플리케이션 컨테이너 정보만 나열합니다. 

![containerd 002]({{ site.url }}{{ site.baseurl }}/assets/images/containerd_002.png)

다른 예로 crictl pods 에는 Kubernetes에 지정된 네임 스페이스로 포드를 필터링하기위한 --namespace 옵션이 포함되어 있습니다.

![containerd 001]({{ site.url }}{{ site.baseurl }}/assets/images/containerd_001.png)

### Build Image
containerd는 이미지 빌드를 지원하지 않습니다. Kubernetes에서는 이 기능이 지원되지 않기 때문입니다. Docker를 계속 사용하여 이미지를 빌드할 수 있습니다. 


### Install Containerd
containerd는 docker-18-ce 버전부터 포함되어있어 kubelet 실행시 containerd.sock 만 지정해주면 어렵지 않게 설정 가능합니다. 
별도 containerd 만 설치도 가능합니다. 설치된 containerd는 systemd 에 등록하여 실행합니다.
```bash
--runtime-cgroups=/system.slice/containerd.service --container-runtime=remote --runtime-request-timeout=15m --container-runtime-endpoint=unix:///run/containerd/containerd.sock
```

### Conclusion
* Containered 1.1은 기본적으로 CRI를 지원합니다. Kubernetes에서 직접 사용할 수 있습니다.
* Containerd 1.1 은 Pod Startup Latency  및 시스템 리소스 활용 측면에서 우수한 성능을 제공합니다.
* crictl은 컨테이너 1.1 및 기타 CRI 호환 컨테이너 런타임과 통신하는 CLI 도구입니다.
* Docker CE 에는 containerd 1.1 가 포함됩니다. 

### 참조
* https://kubernetes.io/blog/2018/05/24/kubernetes-containerd-integration-goes-ga/
* https://cloud.google.com/kubernetes-engine/docs/concepts/using-containerd?hl=ko