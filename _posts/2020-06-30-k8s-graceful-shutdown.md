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

### Pod lifecycle
Pod 종료됨과 동시에 Service의 Endpoint 목록에서 삭제되며 새로운 request/response를 중지하게 됩니다. 
배포 프로세스 중 다운 타임을 피하려면 Pod가 종료되기 전, 애플리케이션이 정상적으로 종료되어야합니다.

Pod가 종료되기 까지 기본적으로 30초가 주어집니다. 30초가 지나면 Pod의 모든 프로세스가로 종료됩니다.
종료되기 전에 응용 프로그램은 모든 요청을 완료해야합니다.

[그림]

Endpoint에서 Pod를 제거하고 SIGTERM 신호를 보내는 것은 순차적이 아니라 동시에 발생합니다. 
하지만, Ingress Controller는 약간의 지연으로 업데이트된 Endpoint 목록을 가져오고 새 트래픽은 계속 Pod로 전송하게 되고 클라이언트가 Pod의 종료시 5XX 오류를 얻게 됩니다.
HTTP 애플리케이션의 경우에는 response Header에 Connection: close 을 보내면 됩니다. 


### NGINX Test
Nginx의 경우, master proccess와 worker를  nginx -s <SIGNAL> 명령어를 통해 프로세스를 즉시 종료할 수 있습니다.
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
2018/01/25 13:58:31 [notice] 1#1: signal 3 (SIGQUIT) received, shutting down
2018/01/25 13:58:31 [notice] 11#11: gracefully shutting down
```
NGINX는 요청이 완료 될 때까지 기다렸다가 프로세스를 종료합니다. 

#### 적용시 발생하는 이슈
여러 경우에 NGINX가 SIGTERM대신에 전송 될 때 이상한 문제가 발생했습니다 SIGQUIT. 그로 인해 요청이 정상적으로 완료되지 않았습니다.
불행히도 우리는이 이상한 행동의 원인이 무엇인지 알 수 없었습니다. 
NGINX 컨테이너의 로그에서“ 5 번 소켓에 열린 소켓 # 10이 남았습니다 ”와 같은 메시지 가 표시되고 포드가 종료되었습니다.
해당 Ingress에서 응답을 연구하면 문제가 분명해집니다.

[그림]

배포 프로세스 중 HTTP 상태 코드
이 경우 503 오류는 수신 자체에서 발생합니다. 컨트롤러가 NGINX 컨테이너에 연결할 수 없습니다 (사용할 수 없음). NGINX 컨테이너의 로그는 다음과 같습니다.

```bash
[alert] 13939#0: *154 open socket #3 left in connection 16
[alert] 13939#0: *168 open socket #6 left in connection 13
```

바꿀 경우, SIGTERM로 SIGQUIT, 컨테이너는 503 오류의 부재에 의해 확인되고, 정상적으로 종료하기 시작합니다.


### 부하 테스트를 통한 검증
부하 테스트를 통해서 컨테이너가 제대로 작동하는지 확인할 수 있습니다.
시나리오는 여러 사용자가 동시에 사이트를 방문한다고 가정합니다.
가장 중요한 것은 단계적으로 변경 사항을 테스트하는 것 입니다. 새로운 수정 사항으로 시작하여 구현하고 테스트를 실행한 후 결과가 이전 실행과 다른지 확인해야합니다.
또한, 컨테이너 로그 종료와 관련된 컨테이너 로그를 분석하는 것이 좋습니다. 

첫 번째 테스트에서는 lifecycle 를 사용하지 않고 테스트합니다.

[그림]

배포 중에 발생하여 평균 5 초 동안 지속되는 502 오류의 급증을 볼 수 있습니다. 
이전 포드가 종료되는 동안 요청이 실패했기 때문이라고 생각합니다.
그 후, 연결이 끊어진 NGINX 컨테이너 중지로 인해 503 오류가 급증했습니다.

두번째는 lifecycle을 설정하고 하위 프로세스를 Graceful shutdown 이 되는지 살펴 보겠습니다. 

[그림]

배치 중에 더 이상 5XX 오류가 없습니다! 프로세스가 계획대로 진행되고 정상적으로 종료됩니다.

하지만 Ingress Controller를 사용하는 경우, 오류가 발생할 수 있습니다. 
방지하기 위해 preStop 에 <sleep> 명령을 통해 지연하여 오류를 피할 수 있습니다.

### 결론
Graceful shutdown 을 위해서는 다음 단계가 필요합니다.
1. 응용 프로그램은 몇 초 정도 기다렸다가 새 연결 수락을 중지해야합니다.
2. 응용 프로그램은 모든 요청이 완료 될 때까지 대기하고 모든 유휴 연결 유지 연결을 닫아야합니다.
3. 자체 프로세스를 종료해야합니다.
4. preStop를 추가하십시오.
5. 백엔드 구성을 분석하고 관련 매개 변수를 선택하십시오.
