---
title: "Kubernetes Pause Container(1)"
date: 2020-07-20 18:26:28 +0900
categories: Container
classes: wide
tags:
  - Kubernetes

---
## Pause Container
Kubernetes 에서 Pod를 생성하면 Pause container도 동시에 생성되는 것을 볼 수 있습니다. Pod 마다 생성되는 pause container의 역할에 대해 간략하게 알아보겠습니다.

![pause_container001]({{ site.url }}{{ site.baseurl }}/assets/images/pause_container001.png)

## Docker 
Kubernetes 에서는 자연스럽게도 Container간 namespaces를 공유하고 있습니다. 하지만 Container를 처음 접할때 namespaces 가 구분되어있으며 IPC(Inter-process communication)프로세스간 통신 격리, PID(Process ID), NS(file system, mount 지점을 분할하여 격리), Network interface, user/groupid 등을 격리한다고 알고 있습니다. 

다음과 같이 docker 명령 옵션을 통해 Container간 namespaces를 공유할 수 있습니다.
```bash
$ docker run -d --name nginx --net=container:pause --ipc=container:pause nginx
```

### IPC settings (--ipc)
IPC Mode가  "shareable" 경우, "container:<donor-name-or-ID>" 다른 컨테이너의 모드를 사용하여 컨테이너의 IPC 메커니즘을 공유해야 할 수 있습니다 .

```bash
--ipc="MODE"  : Set the IPC mode for the container
```

 * “shareable" : 	Own private IPC namespace, with a possibility to share it with other containers.
 * “container:<_name-or-ID_>"	: Join another (“shareable”) container’s IPC namespace.


pause container의 ipc mode가 “shareable” 인것을 확인할 수 있습니다.
![pause_container002]({{ site.url }}{{ site.baseurl }}/assets/images/pause_container002.png)


그리고 Pod에 Puase외 Container의 IPC Mode를 확인해보면 contaner:<pause container id>로 공유된 것을 확인할 수 있습니다.
![pause_container003]({{ site.url }}{{ site.baseurl }}/assets/images/pause_container003.png)


### Network settings
network=container:<name|id> 로 설정한 경우, 다른 Container의 network를 공유할 수 있습니다.
--network="none" : Connect a container to a network
                      'none': no networking
                      'container:<name|id>': reuse another container's network stack

Pause container(1473a0f09e24) Network Mode가 None이며 같은 Pod의 Container(e7ea6ce64c2c) Network는 Pause network 와 연결되어 있는 것을 볼 수 있습니다. 그리고 Pause container의 NetworkSettings는 docker network에 등록된 network id로 설정되어 있습니다.

![pause_container004]({{ site.url }}{{ site.baseurl }}/assets/images/pause_container004.png)                      

### 결론
이렇게 하면 Container간 namespaces 공유가 가능합니다. Pause 는 namespaces 공유 기능외에도 좀비 프로세스를 괸라합니다. 좀비 프로세스는 다음에 좀 더 알아보도록 하겠습니다.