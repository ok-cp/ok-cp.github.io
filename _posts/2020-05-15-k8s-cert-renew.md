---
title: "kubernetes 인증서"
date: 2020-05-15 08:26:28 -0400
categories: Kubernetes
---
v1.14.x 
마스터 노드에서 실행합니다.

1. Backup

$ cp -r /etc/kubernetes/pki /etc/kubernetes/pki_backup
$ mkdir /etc/kubernetes/conf_backup
$ cp -r /etc/kubernetes/*.conf /etc/kubernetes/conf_backup

2. Delete Cert & Config

$ mv /etc/kubernetes/pki/{apiserver.crt,apiserver-etcd-client.key,apiserver-kubelet-client.crt,front-proxy-ca.crt,front-proxy-client.crt,front-proxy-client.key,front-proxy-ca.key,apiserver-kubelet-client.key,apiserver.key,apiserver-etcd-client.crt} /etc/kubernetes/pki_backup/

$ rm -rf /etc/kubernetes/pki/etcd/{healthcheck-client.crt,healthcheck-client.key,peer.crt,peer.key,server.crt,server.key}

$ rm -rf /etc/kubernetes/*.conf

3. Cretate Certs & Config

kubeadm alpha phase certs all –config=/etc/kubernetes/addon/kubeadm-config.yaml
kubeadm alpha phase kubeconfig all –config=/etc/kubernetes/addon/kubeadm-config.yaml

4. Copy admin.conf

cp /etc/kubernetes/admin.conf /root/.kube/config
export KUBECONFIG=/root/.kube/config

5. master nodef Restart Docker & kubelet

systemctl stop kubelet
systemctl stop docker (docker를 내리지 않으면, config가 적용되지 않은 apiserver가 올라와있다)
systemctl start docker
systemctl start kubelet


v1.15 이상부터 kubeadm alpha certs renew 명령어를 사용하여 간단하게 인증서를 리뉴얼할 수 있다.
인증서 갱신은 만료기간 상관없이 renew 실행한 시간부터 다시 설정됩니다.

인증서 만료 확인
kubeadm alpha certs check-expiration 

인증서 갱신
kubeadm alpha certs renew all 
