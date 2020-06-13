---
title: "Wildfly Thread/Request connection pool 설정"
date: 2020-06-04 18:26:28 +0900
categories: Wildfly
classes: wide
tags:
  - Wildfly
---
Wildfly가 web subsystem을 undertow로 변경하면서 설정에도 변화가 생겼습니다. Thread, request connection pool을 설정하는 subsystem과 설정값등이 변경되었습니다.
Tomcat과 다르게 Wildfly는 j2e 기능을 지원하기때문에 설정값이 다양하여 초기 설정에 어려운 부분이 있습니다.
설정할때 subsystem 이름만 주의해서 본다면서 헤매지 않고 설정할 수 있습니다.

### 1. Thread
thread는 io subsystem에 설정합니다. wildfly 기본 Web container는 Undertow이기때문에 기본적으로 모든 리스너가 io subsystem에 설정된 worker를 사용합니다. 
worker는 리스너(AJP/HTTP/HTTPS) IO thread를 관리합니다.

- io-threads:  worker 생성을 위한  I/O thread 수를 지정합니다. 지정하지 않으면 기본값이 선택되어 다음과 같이 계산됩니다. (cpu core * 2)
- stack-size: stack size (in bytes) worker thread의 stack size 를 설정합니다. 기본값 0
- task-keepalive: non-core worker thread 유지시간. 밀리코어 단위 기본값 60000
- task-max-threads: 작업자 태스크 스레드 풀의 최대 스레드 수를 지정하십시오. ( cpucore * 16 )

```xml
  <subsystem xmlns="urn:jboss:domain:io:3.0">
      <worker name="default" task-keepalive="600000" task-max-threads="250" stack-size="10"/>
      <buffer-pool name="default"/>
  </subsystem>
```

### 2. Request connection pool
Request connection pool은 undertow subsystem에 설정합니다.
filters에 name과 max-concurrent-requests, queue-size를 설정합니다. 그리고 host에 filter-ref을 설정하여 filter의 이름을 설정합니다.
참고로 predicate을 사용하면 특정 Path와 확장자(*.jsp 등등)에 대해서만 처리할 수도 있습니다.

```xml
  <subsystem xmlns="urn:jboss:domain:undertow:8.0" default-server="default-server" default-virtual-host="default-host" default-servlet-container="default" default-security-domain="other" statistics-enabled="${wildfly.undertow.statistics-enabled:${wildfly.statistics-enabled:false}}">
      <buffer-cache name="default"/>
      <server name="default-server">
          <http-listener name="default" socket-binding="http" redirect-socket="https" enable-http2="true"/>
          <https-listener name="https" socket-binding="https" security-realm="ApplicationRealm" enable-http2="true"/>
          <host name="default-host" alias="localhost">
              <location name="/" handler="welcome-content"/>
              <filter-ref name="mylimit" predicate="regex('/example/(.*).jsp')"/>
              <http-invoker security-realm="ApplicationRealm"/>
          </host>
      </server>

      (중략)

      <filters>
          <request-limit name="mylimit" max-concurrent-requests="512" queue-size="10"/>
      </filters>
  </subsystem>
```
