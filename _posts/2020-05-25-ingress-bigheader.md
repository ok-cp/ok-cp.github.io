---
title: "Nginx ingress controller에서 'upstream sent too big header error' 처리"
date: 2020-05-25 08:26:28 +0900
categories: Kubernetes
classes: wide
tags:
  - kubernetes 
  - nginx ingress
---
## The error: "upstream sent too big header while reading response header from upstream"
Kubernetes에서 Nginx Ingress controller를 사용할 경우, 자주 접하게 되는 에러입니다. 
브라우저에서는 502 Bad Gateway 메세지가 보이며 nginx ingress controller 로그에는 "upstream sent too big header while reading response header from upstream" 다음 메세지를 출력합니다.

이것은 nginx에서도 발생하는 일반적인 header size 문제입니다.
nginx에서 header size를 조정할 수 있도록 설정을 제공하며 ingress controller에도 적용할 수 있습니다.

ingress annotaion에 다음과 같이 설정합니다.
```yaml
nginx.ingress.kubernetes.io/proxy-buffers-number: "4"
nginx.ingress.kubernetes.io/proxy-buffer-size: "8k"
```

## buffer size Tip


### proxy-buffer-size(proxy_buffer_size)
보통은 로그인을 하는 순간 Set-Cookie로 인해 HTTP Header size가 커지게 됩니다.

아래 방법으로 Response header size를 가져올 수 있지만 Set-Cookie가 포함되지 않을 수 있기 때문에 로그인 자격증명이 post method를 통해 자격증명된 url로 조정해야합니다.
```yaml
curl -s -w \%{size_header} -o /dev/null https://google.com
```

여기서 나오는 header size는 byte이며 이 값은 memory page size와 정렬되어야합니다.
linux에서 getconf PAGESIZE 를 통해 page size를 알 수 있으며 대개 4k로 설정되어있습니다.
예를 들어 header size가 9000이라면 proxy_buffer_size 12k 로 설정되어야합니다.


### proxy-buffers-number(proxy_buffers)
버퍼 사이즈를 작게 설정할 경우, 디스크에 저장되기때문에 디스크 i/o를 피하고 버퍼만 사용하여 빠르게 응답하기 위해 적절한 사이즈로 조정해야합니다.
애플리케이션 reponse body size와 압축된 리소스를 요청여부를 고려하여 사이즈를 조정해야합니다.

응답 리소스 사이즈는 다음 명령어를 통해 알아볼 수 있습니다.
```yaml
curl -so /dev/null https://google.com/ -w '%{size_download}'
```
예를 들어 압축되지 않은 body size가 512k인 경우, proxy-buffers-number 128 로 설정합니다. 
proxy_buffers는 성능 최적화를 위한 설정이기때문에 서버 리소스(Mem,Disk)에 맞게 적용하는 것이 중요합니다.




