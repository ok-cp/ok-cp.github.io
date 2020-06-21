---
title: "Kubernetes docker suid/guid"
date: 2020-06-21 18:26:28 +0900
categories: Container
classes: wide
tags:
  - Kubernetes
  - Docker

---
## Privilege Escalation
이 옵션은 allowPrivilegeEscalation 컨테이너 옵션을 제어합니다. 이 옵션은 컨테이너 프로세스에서 no_new_privs 플래그가 설정되는지 여부를 직접 제어합니다. 이 플래그는 setuid 바이너리가 유효 사용자 ID를 변경하지 못하게하고 파일이 추가 기능을 활성화하지 못하게합니다 (예 : 핑 도구 사용을 방해 함). MustRunAsNonRoot를 효과적으로 적용하려면이 동작이 필요합니다.

## AllowPrivilegeEscalation
사용자가 컨테이너의 Security Context, allowPrivilegeEscalation = true로 설정할 수 있는지 여부를 지정합니다. true는 setuid 바이너리를 기본적으로 허용됩니다. false로 설정하면 컨테이너의 하위 프로세스가 상위보다 더 많은 권한을 얻을 수 없습니다.

### kubernetes
```yaml
kind: …
apiVersion: …
metadata:
  name: …
  spec:
…
   containers:
    – name: …
      image: ….
      securityContext:
  …
      allowPrivilegeEscalation: false
  …
```

### docker
Docker에서는 security-opt=no-new-privileges 옵션을 통해 사용할 수 있습니다.
```bash
docker run –security-opt=no-new-privileges …
```