---
title: "Kubernetes Time based Scaling"
date: 2020-07-06 18:26:28 +0900
categories: Container
classes: wide
tags:
  - Kubernetes

---
## Kubernetes AutoScale
Kubernetes에서는 CPU 사용률에 따라 확장되는 HorizontalPodAutoscaler를 사용하여 자동확장을 할 수 있습니다.
일반적인 상황에서는 문제없이 작동할 수 있지만 예상된 트래픽보다 몰리는 상황에서는 대응하지 못할 수 있습니다.
HPA에 의해 Pod가 생성되는 시간이 길진 않지만 트래픽을 수용할 수 있는 시간보다는 길 수 있기 때문입니다.
이러한 대규모 트래픽이 예상되는 이벤트가 있을 경우, HPA보다는 Timebased의 Pod 확장이 필요합니다.


## Job/CronJob
Kubernetes의 Job/CronJob을 사용하면 Timebased로 Pod를 확장할 수 있습니다.
그리고 args에 kubectl을 이용하여 scale 조정을 합니다.
Container에서 kubectl을 사용하기 위해서는 사전에 kubectl CLI를 Dockerfile를 통해서 추가시켜줘야합니다.

  * kubectl 설치 방법(https://kubernetes.io/ko/docs/tasks/tools/install-kubectl/)

그리고 kubectl를 사용하기 위한 kubeadm config가 필요합니다.

### Dockerfile
```bash
FROM debian:buster-slim

RUN apt-get update && apt-get install curl -y; \
    curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.16.0/bin/linux/amd64/kubectl; \
    chmod +x ./kubectl; \
    mv ./kubectl /usr/local/bin/kubectl; \
    kubectl version --client

```

### SclaeUp
```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: nginx-scaleup
  namespace: demo
spec:
  failedJobsHistoryLimit: 1
  jobTemplate:
    metadata:
      creationTimestamp: null
    spec:
      template:
        metadata:
          creationTimestamp: null
        spec:
          containers:
          - command:
            - /bin/sh
            - -c
            - kubectl scale deploy nginx -n demo --replicas=3
            image: okcp/kubectl:latest
            imagePullPolicy: Always
            name: scaleup
            volumeMounts:
              - mountPath: /root/.kube
                name: kube-config            
          dnsPolicy: ClusterFirst
          restartPolicy: OnFailure
          schedulerName: default-scheduler
          volumes:
            - name: kube-config
              secret:
                defaultMode: 420
                secretName: kube-config          
  schedule: 30 14 * * *

```
이렇게 하면 Timebased 의 Scaleup뿐만 아니라 ScaleDown도 가능합니다.
