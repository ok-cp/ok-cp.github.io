---
title: "kubernetes 인증서"
date: 2020-05-15 08:26:28 +0900
categories: Kubernetes
classes: wide
---
## kubernetes 인증서 종류
* kubelet에서 API 서버 인증서를 인증시 사용하는 클라이언트 인증서
* API 서버 엔드포인트를 위한 서버 인증서
* API 서버에 클러스터 관리자 인증을 위한 클라이언트 인증서
* API 서버에서 kubelet과 통신을 위한 클라이언트 인증서
* API 서버에서 etcd 간의 통신을 위한 클라이언트 인증서
* 컨트롤러 매니저와 API 서버 간의 통신을 위한 클라이언트 인증서/kubeconfig
* 스케줄러와 API 서버간 통신을 위한 클라이언트 인증서/kubeconfig
* front-proxy를 위한 클라이언트와 서버 인증서

### 인증서를 저장하는 위치
```bash
[root@bskim-master ~]# ls /etc/kubernetes/pki/
apiserver.crt              apiserver-etcd-client.key  apiserver-kubelet-client.crt  ca.crt  ca.srl  etcd                front-proxy-ca.key  front-proxy-client.crt  sa.key
apiserver-etcd-client.crt  apiserver.key              apiserver-kubelet-client.key  ca.key  conf    front-proxy-ca.crt  front-proxy-ca.srl  front-proxy-client.key  sa.pub
```
### v1.14.x 
마스터 노드에서 실행합니다.

#### 1. Backup
```yaml
$ cp -r /etc/kubernetes/pki /etc/kubernetes/pki_backup
$ mkdir /etc/kubernetes/conf_backup
$ cp -r /etc/kubernetes/*.conf /etc/kubernetes/conf_backup
```

#### 2. Delete Cert & Config
```yaml
$ mv /etc/kubernetes/pki/{apiserver.crt,apiserver-etcd-client.key,apiserver-kubelet-client.crt,front-proxy-ca.crt,front-proxy-client.crt,front-proxy-client.key,front-proxy-ca.key,apiserver-kubelet-client.key,apiserver.key,apiserver-etcd-client.crt} /etc/kubernetes/pki_backup/
$ rm -rf /etc/kubernetes/pki/etcd/{healthcheck-client.crt,healthcheck-client.key,peer.crt,peer.key,server.crt,server.key}
$ rm -rf /etc/kubernetes/*.conf
```

#### 3. Cretate Certs & Config
```yaml
kubeadm alpha phase certs all –config=/etc/kubernetes/addon/kubeadm-config.yaml
kubeadm alpha phase kubeconfig all –config=/etc/kubernetes/addon/kubeadm-config.yaml
```

#### 4. Copy admin.conf
```yaml
cp /etc/kubernetes/admin.conf /root/.kube/config
export KUBECONFIG=/root/.kube/config
```

#### 5. master nodef Restart Docker & kubelet
```yaml
systemctl stop kubelet
systemctl stop docker (docker를 내리지 않으면, config가 적용되지 않은 apiserver가 올라와있다)
systemctl start docker
systemctl start kubelet
```

### v1.15 이상부터 kubeadm alpha certs renew 명령어를 사용하여 간단하게 인증서를 리뉴얼할 수 있다.
인증서 갱신은 만료기간 상관없이 renew 실행한 시간부터 다시 설정됩니다.

#### 인증서 만료 확인
kubeadm alpha certs check-expiration 

#### 인증서 갱신
kubeadm alpha certs renew all 

### 참조
https://kubernetes.io/ko/docs/setup/best-practices/certificates/