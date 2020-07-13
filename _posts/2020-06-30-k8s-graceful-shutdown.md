---
title: "Kubernetes - Graceful shutdown"
date: 2020-06-30 18:26:28 +0900
categories: Container
classes: wide
tags:
  - Kubernetes

---
## Graceful shutdown
Kubernetes에서 rollingupdate를 사용한다면 무중단 배포를 위해 Graceful shutdown은 굉장히 중요합니다.
무중단 배포를 위해서는 Graceful shutdown 외에도 Readiness/liveness Probe 도 꼭 필요합니다만 이번엔 Graceful shutdown 을 집중적으로 다루어보도록 하겠습니다.

### Pod lifecycle
Pod 종료됨과 동시에 Service의 Endpoint 목록에서 삭제되며 새로운 request/response를 중지하게 됩니다. 
배포 프로세스 중 다운 타임을 피하려면 Pod가 종료되기 전, 애플리케이션이 정상적으로 종료되어야합니다.

Pod가 종료되기 까지 기본적으로 30초가 주어집니다. 30초가 지나면 Pod의 모든 프로세스가로 종료됩니다.
종료되기 전에 응용 프로그램은 모든 요청을 완료해야합니다.

![kubelet]({{ site.url }}{{ site.baseurl }}/assets/images/kubelet.png)

Endpoint에서 Pod를 제거하고 SIGTERM 신호를 보내는 것은 순차적이 아니라 동시에 발생합니다. 
하지만, Ingress Controller는 약간의 지연으로 업데이트된 Endpoint 목록을 가져오고 새 트래픽은 계속 Pod로 전송하게 되고 클라이언트가 Pod의 종료시 5XX 오류를 얻게 됩니다.
HTTP 애플리케이션의 경우에는 response Header에 Connection: close 을 보내면 됩니다. 


### NGINX Test
Nginx의 경우, master proccess와 worker를 nginx -s <SIGNAL> 명령어를 통해 프로세스를 즉시 종료할 수 있습니다.
Nginx Pod의 Graceful shutdown을 위해 SIGNAL를 보내는 명령을 preStop에 추가해야합니다.

```yaml
lifecycle: 
      preStop : 
        exec : 
          command : 
          - /usr/sbin/nginx 
          - -s 
          - quit
```

#### Nginx Container log
```bash
2020/06/25 11:55:30 [notice] 1#1: signal 3 (SIGQUIT) received, shutting down
2020/06/25 11:55:30 [notice] 11#11: gracefully shutting down
```
NGINX는 요청이 완료 될 때까지 기다렸다가 프로세스를 종료합니다. 
물론 간혹 모든 소켓이 닫히지 않고 종료되는 경우도 발생합니다.


### 부하 테스트를 통한 검증
부하 테스트를 통해서 컨테이너가 제대로 작동하는지 확인할 수 있습니다.
시나리오는 여러 사용자가 동시에 사이트를 방문한다고 가정합니다.
가장 중요한 것은 단계적으로 변경 사항을 테스트하는 것 입니다. 새로운 수정 사항으로 시작하여 구현하고 테스트를 실행한 후 결과가 이전 실행과 다른지 확인해야합니다.
또한, 컨테이너 로그 종료와 관련된 컨테이너 로그를 분석하는 것이 좋습니다. 

첫 번째 테스트에서는 lifecycle 를 사용하지 않고 테스트합니다.

![shutdown_test001]({{ site.url }}{{ site.baseurl }}/assets/images/shutdown_test_001.png)


두번째는 lifecycle을 설정하고 하위 프로세스를 Graceful shutdown 이 되는지 살펴 보겠습니다. 

![shutdown_test003]({{ site.url }}{{ site.baseurl }}/assets/images/shutdown_test_003.png)

에러율이 줄어든 것을 확인할 수 있습니다. 그래도 에러가 발생하는 이유는 Pod 실행시 nginx 프로세스가 실행되기 전에 들어오는 요청이 있기 때문입니다. 이 문제는 Probe로 해결 가능합니다.
하지만 Ingress Controller를 사용하는 경우, 오류가 발생할 수 있습니다. 
방지하기 위해 preStop 에 <sleep> 명령을 통해 지연하여 오류를 피할 수 있습니다.

### 결론
Graceful shutdown 을 위해서는 다음 단계가 필요합니다.
1. 응용 프로그램은 몇 초 정도 기다렸다가 새 연결 수락을 중지해야합니다.
2. 응용 프로그램은 모든 요청이 완료 될 때까지 대기하고 모든 유휴 연결 유지 연결을 닫아야합니다.
3. 자체 프로세스를 종료해야합니다.
4. preStop를 추가하고 관련 매개 변수를 선택합니다.
