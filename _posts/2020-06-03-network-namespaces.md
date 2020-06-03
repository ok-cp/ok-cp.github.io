---
title: "Namespace Networking"
date: 2020-06-02 18:26:28 +0900
categories: Kubernetes
tags:
  - Container
  - network
  - namespaces
---
Namespace는 Docker와 같은 컨테이너에서 네트워크 격리를 구현하는 데 사용됩니다.

컨테이너를 생성할 때 호스트나 다른 컨테이너로부터 프로세스가 표시되지 않도록 격리해야합니다.

Namespace를 사용하여 호스트에 격리된 공간을 만듭니다.

### **Process Namespace**

컨테이너 내부에서 실행하는 프로세스만보고 자체 호스트가 존재한다고 생각할 수 있습니다.

하지만 기본 호스트에서는 컨테이너 내부에서 실행되는 프로세스를 포함한 모든 프로세스를 볼 수 있습니다.

컨테이너 내부에서 프로세스를 나열 할 때 프로세스 ID가 1 인 단일 프로세스가 표시됩니다. 

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9cf7a362-d935-445f-bf36-83749e6205ab/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9cf7a362-d935-445f-bf36-83749e6205ab/Untitled.png)

호스트의 루트 사용자와 동일한 프로세스를 나열하면, 컨테이너 내에서 실행중인 프로세스와 함께 다른 모든 프로세스가 표시됩니다.

컨테이너 내부와 외부에서 다른 pid로 실행되는 동일한 프로세스를 찾을 수 있습니다.

이것이 네임 스페이스의 작동 방식입니다.

### **Network Namespace**

컨테이너에서 모든 세부 사항을 격리하기 위해 생성 될때 Network Namespace를 만듭니다. 컨테이너 내부에서는 호스트의 네트워크 관련 정보를 볼 수 없습니다. 

호스트에는 근거리 통신망에 연결되는 자체 인터페이스가 있습니다. 그리고 네트워크에 대한 정보가있는 자체 라우팅 및 ARP 테이블이 있습니다. 

Namespace 안에서 컨테이너는 자체 가상 인터페이스 라우팅 및 ARP 테이블을 가지게 됩니다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7a70c630-ed35-400a-84db-597d93082ca0/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7a70c630-ed35-400a-84db-597d93082ca0/Untitled.png)

호스트에서 새로운 Network Namespace 작성은 ip nets add 명령을 통해 실행 가능합니다.

다음과 예를 들어보겠습니다.

이 경우 Network Namespace를 나열하기 위해 두 개의 Network Namespace를 작성하고 ip netns 명령을 실행하여 namespace를 나열하였습니다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cc1cecbf-ac5d-425c-ad94-11ed40874c00/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cc1cecbf-ac5d-425c-ad94-11ed40874c00/Untitled.png)

호스트에서 ip link 명령을 실행하여 루프백 인터페이스와 eth0 인터페이스를 다시 살펴보겠습니다.

방금 만든 Network Namespace 정보와 비교해보겠습니다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0d58ab03-3b09-47b3-be24-46e88150cdbb/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0d58ab03-3b09-47b3-be24-46e88150cdbb/Untitled.png)

Namespace 정보는 다음명령어로 조회가능합니다.

```bash
$ ip nets exec red ip link 또는 ip -n red link
```

red, blue 둘 다 루프백 인터페이스만 나열됩니다.

호스트의 eth0 인터페이스를 볼 수 없습니다.

따라서 Namespace는 컨테이너가 호스트 인터페이스를 볼 수 없습니다. ARP 테이블에서도 마찬가지입니다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8c70d7d7-4199-413b-9745-b0a0c5bc3027/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8c70d7d7-4199-413b-9745-b0a0c5bc3027/Untitled.png)

호스트에서 ARP 명령을 실행하면 항목 목록이 표시됩니다. 그러나 컨테이너 내부에서 실행하면.
라우팅 테이블에 대한 항목이없고 동일합니다.

```bash
# ARP 테이블 조회
$ arp
$ ip nets exec red arp

# Route 테이블 조회
$ route
$ ip nets exec red route
```

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0becce05-0cde-4684-bdd5-2b91322a3dd5/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0becce05-0cde-4684-bdd5-2b91322a3dd5/Untitled.png)

현재 만든 Network Namespace 에는 네트워크 연결할 수 없습니다. 자체 인터페이스가 없으며 기본 호스트 네트워크를 볼 수도 없습니다.

### **Namespace간 Network**

먼저 Namespace간에 Network부터 살펴보겠습니다.

각 머신의 이더넷 인터페이스에 케이블을 사용하여 물리적 머신을 함께 연결하는 방법과 마찬가지로 가상 이더넷 쌍 또는 가상 케이블을 사용하여 Namespace에 함께 연결할 수 있습니다.

양쪽 끝에 두 개의 인터페이스가있는 가상 케이블 또는 파이프라인이라고합니다. 케이블을 만들려면 

ip link add 명령어를 사용합니다. 

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1d19af60-f7b5-4e74-94e2-d78df60d3c40/test.gif](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1d19af60-f7b5-4e74-94e2-d78df60d3c40/test.gif)

```bash
# veth-red와 veth-blue의 가상 인터페이스를 pair 구성한다.
$ ip link add veth-red type veth peer name veth-blue

# 각 인터페이스에 적절한 Namespaces를 연결한다.
$ ip link set veth-red netns red
$ ip link set veth-blue netns blue

# Namespaces에 IP 주소를 할당
$ ip -n red addr add 192.168.15.1 dev veth-red
$ ip -n blue addr add 192.168.15.2 dev veth-blue

# Interface up
$ ip -n red link set veth-red up
$ ip -n blue link set veth-blue up

# Ping test
$ ip netns exec red ping 192.168.15.2
```

ip addr 명령을 사용하여 IP 주소를 할당합니다. 각 namespaces 내에서 red namespaces IP 192.168.15.1을 할당. blue namespaces IP 192.168.15.2입니다.

그 다음 각 namespaces 내 장치에 대해 ip link set up 명령을 사용하여 인터페이스를 불러옵니다. link가 작동하면 namespaces간 서로 도달할 수 있습니다. red namespaces에서 ping을 시도하여 blue IP에 도달할 수 있습니다.

red namespace 에서 ARP 테이블을 보면 MAC 주소로 192.168.1.2에서 blue가 식별 된 것을 볼 수 있습니다. 마찬가지로 blue namespace 에 ARP 테이블을 나열하면 해당 테이블이 red 라는 것을 알 수 있습니다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8f24295d-bcfc-4ed2-84b7-29ede1f9e70b/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8f24295d-bcfc-4ed2-84b7-29ede1f9e70b/Untitled.png)

```bash
# namespace apr 조회
$ ip netns exec red arp
Address       HWtype     HWaddress         Flags Mask    Iface
192.168.15.2  ether   ba:b0:6d:68:09:e9       C          veth-red

$ ip netns exec blue arp
Address       HWtype     HWaddress         Flags Mask    Iface
192.168.15.1  ether   7a:9d:9b:c8:3b:7f       C          veth-blue

# host apr 조회
$ apr
Address       HWtype     HWaddress         Flags Mask    Iface
192.168.1.3   ether  52:54:00:12:35:03       C            eth0
192.168.1.4   ether  52:54:00:12:35:04       C            eth0
```

이것을 호스트의 ARP 테이블과 비교하면 호스트의 ARP 테이블은 방금 만든 새로운 namespace에 대해 전혀 알지 못합니다. 또한 namespace의 인터페이스도 알지 못합니다. 

이렇게 namespace간 통신이 가능합니다. 

### Linux Bridge

이제 호스트 외부에서 접근 가능한 방법을 알아보겠습니다. 일반적으로 호스트 내부에 가상 네트워크를 만드려면 스위치가 필요합니다. 따라서 가상 네트워크를 만들려면 가상 스위치가 필요하므로 호스트 내에 가상 스위치를 만들고 namespaces를 연결합니다.

호스트 내에서 가상 스위치를 생성하는 방법은 사용 가능한 Linux Bridge 및 Open vSwitch 등의 여러 기본 솔루션이 있습니다. 

여기서에서는 Linux Bridge 옵션을 사용하여 내부 Bridge network를 생성하고 ip link add 명령으로 bridge로 설정하여 호스트에 새 인터페이스를 추가합니다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a286ab76-77ad-4c61-8c2e-14c07beba889/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a286ab76-77ad-4c61-8c2e-14c07beba889/Untitled.png)

```bash
# v-net-0 bridge 생성
$ ip link add v-net-0 type bridge

# 조회
$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state
UNKNOWN mode DEFAULT group default qlen 1000
link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc
fq_codel state UP mode DEFAULT group default qlen 1000
link/ether 02:0d:31:14:c7:a7 brd ff:ff:ff:ff:ff:ff
6: v-net-0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN
mode DEFAULT group default qlen 1000
link/ether 06:9d:69:52:6f:61 brd ff:ff:ff:ff:ff:ff

# v-net-0 bridge up
$ ip link set dev v-net-0 up
```

ip link를 명령를 실행하면 v-net-0을 확인합니다. 그리고 ip link set dev up 명령을 사용하여 올립니다. 이제 v-net-0 인터페이스는 연결할 수있는 스위치와 같습니다.

따라서 호스트의 인터페이스와 namespace 의 스위치로 생각하겠습니다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8a69f0e0-4704-44b9-8c22-d202d60c75ca/test2.gif](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8a69f0e0-4704-44b9-8c22-d202d60c75ca/test2.gif)

다음 단계는 namespace 를 새로운 가상 네트워크 스위치에 연결하는 것입니다.

앞서 두 namespaces를 직접 연결했습니다. 한쪽 끝에 veth-red 인터페이스와 다른쪽에 blue 인터페이스로 케이블 또는 eth pair를 만들었습니다.

이제 Bridge network로 연결하겠습니다. 이를 위해 새로운 케이블이 필요합니다.

이 케이블은 더 이상 의미가 없으므로 ip link del 명령을 사용하여 링크를 삭제하는데 케이블을 삭제되면 다른 쪽 끝이 쌍이므로 자동으로 삭제됩니다. 

```bash
# namespace간 link 삭제
$ ip -n red link del veth-red
```

이제 namespace를 bridge에 연결하기 위해 새 케이블을 만들기 위해 ip link add 명령을 실행하고 앞서 동일하게 한쪽 끝에 veth-red와 pair를 만듭니다. 다른 쪽 끝을 veth-red-br이라고합니다.  

blue namespace도 bridge network에 연결하는 케이블을 만듭니다.

```bash
# veth-red가 veth-red-br와 pair 되도록 한다.
$ ip link add veth-red type veth peer name veth-red-br

# veth-blue가 veth-blue-br와 pair 되도록 한다.
$ ip link add veth-blue type veth peer name veth-blue-br
```

이제 케이블이 준비되었으므로 namespace에 연결해야합니다. 앞서 동일하게 ip link set veth-red netns red 명령을 실행하여 veth-red와 red namespace를 연결합니다. 

다른 쪽 끝을 bridge network에 연결하려면 veth-red-br 끝에서 ip link set 명령을 실행하고 해당 master v-net-0 네트워크로 지정합니다.

```bash
# red namespace와 veth-red 인터페이스를 연결한다.
$ ip link set veth-red netns red
# veth-red-br(red namespace bridge)과 v-net-0(host bridge)를 연결한다.
$ ip link set veth-red-br master v-net-0

# blue namespace와 veth-blue 인터페이스를 연결한다.
$ ip link set veth-blue netns blue
# veth-blue-br(bluenamespace bridge)과 v-net-0(host bridge)를 연결한다.
$ ip link set veth-blue-br master v-net-0
```

blue namespaces 및 blue bridge network에 대한 blue 케이블도 동일하게 진행합니다.

이제 link에 대한 IP 주소를 설정합니다.  이전 동일한 IP 주소 인 192.168.15.1, 192.168.15.2를 사용합니다. 그리고 장치를 올립니다.

```bash
# 각 namespace의 인터페이스에 IP주소를 설정한다.
$ ip -n red addr add 192.168.15.1 dev veth-red
$ ip -n blue addr add 192.168.15.2 dev veth-blue

# veth-red, veth-blue up
$ ip -n red link set veth-red up
$ ip -n blue link set veth-blue up
```

컨테이너는 이제 네트워크를 통해 서로 연결할 수 있습니다. 따라서 동일한 절차에 따라 나머지 두 namespace를 동일한 네트워크에 연결할 수 있습니다. 

내부 분기 네트워크에 4개의  namespace가 모두 연결되어 있으며 서로 통신 할 수 있습니다.

모든 IP 주소는 192.168.15.1, 2, 3, 4입니다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1c296d4e-9b44-42a4-a7c2-da543ea4003b/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1c296d4e-9b44-42a4-a7c2-da543ea4003b/Untitled.png)

하지만 아직도 호스트와 namespace간 연결은 할 수 없습니다. 호스트와 namespace는 다른 네트워크에 있습니다. 호스트와 namespace에 연결을 위해서 어떻게 할까요?

bridge 스위치는 실제로 호스트의 네트워크 인터페이스라고 했습니다. 따라서 호스트의 192.168.15 네트워크에 인터페이스가 있습니다.

bridge 인터페이스에 IP 주소를 지정하면 이 인터페이스를 통해 namespaces에 도달 할 수 있습니다. ip addr 명령을 실행하여 IP 192.168.15.5/24를 설정합니다.

```bash
# v-net-0 bridge 에 IP 설정
ip addr add 192.168.15.5/24 dev v-net-0

# Ping test
$ ping 192.168.15.1
```

이제 v-net-0 인터페이스는 호스트에서 namespace로 통신 가능합니다.

### Namespace에서 다른 호스트간 Network

전체 네트워크는 여전히 비공개이며 namespace는 호스트 내에서 제한됩니다.
주소가 192.168.1.3 인 LAN 네트워크에 다른 호스트가 연결되어 있다고 가정해보겠습니다.

Namespace 내에서 이 호스트에 어떻게 연결할 수 있을까요?

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/25358b90-e471-4e8d-9acb-a101ca5cf739/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/25358b90-e471-4e8d-9acb-a101ca5cf739/Untitled.png)

blue namespace에서 다른 호스트로 ping 시도를 하게 된다면 blue namespace는 현재 네트워크 192.168.15.0와 다른 192.168.1.0의 네트워크에 도달하려고합니다. 라우팅 테이블을 보고 해당 네트워크를 찾는 것입니다.

하지만 라우팅 테이블에는 다른 네트워크에 대한 정보가 없으므로 네트워크에 연결할 수 없다는 메시지가 표시됩니다.

라우팅 테이블에 항목을 추가하여 외부 호스트 연결에 필요한 게이트웨이를 제공해야합니다.

Namespace에서 호스트와 통신하기 위한 논리적 구조는 다음과 같습니다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/561fd933-d0f3-4bdb-b969-7ba368c23a1f/test3.gif](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/561fd933-d0f3-4bdb-b969-7ba368c23a1f/test3.gif)

호스트는 두 네트워크를 함께 연결하는 게이트웨이입니다.

blue namespace route에 추가하여 모든 트래픽을 192.168.1.0/24로 라우팅 할 수 있습니다.

```bash
# Bridge(192.168.15.5,source)가 eth0(192.168.1.0/24,destination)로 나갈수 있도록 route 지정한다.
$ ip netns exec blue ip route add 192.168.1.0/24 via 192.168.15.5

# Ping Test 
$ ip netns exec blue ping 192.168.1.3
PING 192.168.1.3 (192.168.1.3) 56(84) bytes of data.
```

이제 호스트에는 192.168.15.5의 bridge 네트워크와 192.168.1.2의 외부 네트워크에 각각 두 개의 IP 주소가 있습니다. 

하지만 blue namespaces는 192.168.15.5의 로컬 네트워크에있는 게이트웨이에만 도달 할 수 있기 때문입니다. 지금 Ping을 시도하면 더 이상 네트워크에 연결할 수 없다는 메시지가 표시되지 않습니다.

아직 해결하지 못한 문제가 더 있습니다.

iptable을 사용하여 호스트에 nat를 추가해야합니다.

POSTROUTING 체인의 NAT iptable에 새 규칙을 추가하여 소스 네트워크 192.168.15.0/24에서 오는 모든 패킷의 발신 주소를 고유 한 IP 주소로 바꿉니다.

```bash
# Namespace 에서 Bridge 로 통해 나갈때 내부아이피를 외부아이피로 변경한다.
$ iptables -t nat -A POSTROUTING -s 192.168.15.0/24 -j MASQUERADE

# Ping test
$ ip netns exec blue ping 192.168.1.3
64 bytes from 192.168.1.3: icmp_seq=1 ttl=63 time=0.587 ms
64 bytes from 192.168.1.3: icmp_seq=2 ttl=63 time=0.466 ms
```

그렇게하면 네트워크 외부에서 이러한 패킷을받는 사람은 Namespace가 아닌 호스트에서 온 것으로 생각합니다.

Ping 시도하면 다른 호스트로 접근가능한 것을 볼 수 있습니다.

### Default gateway

만약, LAN이 인터넷에 연결되어있다면 namespaces는 인터넷에 도달할 수 있어야합니다.

blue namespace에서 인터넷의 서버를 8.8.8.8로 Ping하려고합니다. 하지만 네트워크 접근할 수 없습니다. 

 그 이유는 라우팅 테이블을 보면 알수 있습니다. 네트워크 192.168.1에 대한 경로는 있지만 다른 것은 없습니다. namespaces는 호스트가 도달 할 수있는 모든 네트워크에 도달 할 수 있습니다. 따라서 호스트를 지정하는 default gateway를 추가합니다.

```bash
# 인터넷 서버로 접근할 수 없음.
$ ip netns exec blue ping 8.8.8.8
Connect: Network is unreachable

# blue namespaces route 확인
$ ip netns exec blue route
Destination    Gateway        Genmask        Flags Metric Ref Use Iface
192.168.15.0   0.0.0.0        255.255.255.0  U     0      0   0   veth-blue
192.168.1.0    192.168.15.5   255.255.255.0  UG    0      0   0   veth-blue

# blue namespace route에 default gateway 추가
$ ip netns exec blue ip route add default via 192.168.15.5
Destination    Gateway        Genmask        Flags Metric Ref Use Iface
192.168.15.0   0.0.0.0        255.255.255.0  U     0      0   0   veth-blue
192.168.1.0    192.168.15.5   255.255.255.0  UG    0      0   0   veth-blue
Default        192.168.15.5   255.255.255.0  UG    0      0   0   veth-blue

# Ping test
$ ip nets exec blue ping 8.8.8.8
64 bytes from 8.8.8.8: icmp_seq=1 ttl=63 time=0.587 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=63 time=0.466 ms
```

### 외부 호스트에서 Namespace networking

예를 들어 blue namespace가 port 80에서 웹애플리케이션을 호스팅했다고 가정합니다.

Namespace는 내부 Private 네트워크에 있으며 외부 Public 네트워크에서는 그것에 대해 알지 못합니다.

Ping을 시도하면 호스트 자체에서만 접근가능하며 외부에서는 Namespace에 접근하지 못합니다.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cca1a0f5-5cb5-43af-8c67-d04402e61025/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cca1a0f5-5cb5-43af-8c67-d04402e61025/Untitled.png)

iptables을 사용하여 포트 전달 규칙을 추가하여 로컬 호스트의 포트 80으로 들어오는 트래픽이 blue namespaces에 할당 된 IP의 포트 80으로 전달되도록하는 것입니다.

```bash
# HostIP:80 -> 192.168.15.2:80(namespace) 허용
$ iptables -t nat -A PREROUTING --dport 80 --to-destination 192.168.15.2:80 -j DNAT
```

이렇게 웹애플리케이션 컨테이너는 외부 네트워크를 통해 접근할 수 있습니다.

지금까지 Namespace networking을 알아보았습니다.