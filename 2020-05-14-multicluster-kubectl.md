2020-05-14
멀티 클러스터 관리 - (kubeconfig 설정)
멀티 클러스터가 구성되어있는 경우, kubeconfig 설정을 통해서 kubectl이 멀티클러스터를 관리할 수 있도록 할 수 있다.

kubeconfig는 마스터 노드에 있다.
master $ ls /root/.kube/config
/root/.kube/config

클러스터 추가는 다음과 같이 한다.
kubectl config set-cluster development --server=https://1.2.3.4 --certificate-authority=/etc/kubernetes/pki/ca.crt
kubectl config set-credentials development --client-certificate=/etc/kubernetes/pki/users/dev-user/developer-user.crt --client-key=/etc/kubernetes/pki/users/test-user/test-user.key
kubectl config set-context development --cluster=development --namespace=storage --user=dev-user

kubectl config view 명령어를 통해 config 를 조회할 수 있다.
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://1.2.3.4
  name: development
- cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: KUBE_ADDRESS
  name: kubernetes-on-aws
- cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: KUBE_ADDRESS
  name: production
- cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: KUBE_ADDRESS
  name: test-cluster-1
contexts:
- context:
    cluster: kubernetes-on-aws
    user: aws-user
  name: aws-user@kubernetes-on-aws
- context:
    cluster: test-cluster-1
    user: dev-user
  name: research
- context:
    cluster: development
    user: test-user
  name: test-user@development
- context:
    cluster: production
    user: test-user
  name: test-user@production
current-context: test-user@development
kind: Config
preferences: {}
users:
- name: aws-user
  user:
    client-certificate: /etc/kubernetes/pki/users/aws-user/aws-user.crt
    client-key: /etc/kubernetes/pki/users/aws-user/aws-user.key
- name: dev-user
  user:
    client-certificate: /etc/kubernetes/pki/users/dev-user/developer-user.crt
    client-key: /etc/kubernetes/pki/users/dev-user/dev-user.key
- name: test-user
  user:
    client-certificate: /etc/kubernetes/pki/users/test-user/test-user.crt
    client-key: /etc/kubernetes/pki/users/test-user/test-user.key


만약 test-cluster-1 클러스터를 사용하고 싶다면 다음과 같이 명령어를 입력한다.
kubectl config --kubeconfig=/root/my-kube-config use-context research
