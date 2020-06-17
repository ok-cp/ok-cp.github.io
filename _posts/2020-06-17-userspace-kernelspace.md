---
title: "User Space & Kernel Space"
date: 2020-06-17 18:26:28 +0900
categories: Container
classes: wide
tags:
  - Container

---
컨테이너는 때때로 가상 머신처럼 취급되지만 가상 머신과 달리 커널 프로그램과 액세스해야하는 리소스 사이의 유일한 추상화 계층입니다. 

모든 프로세스는 시스템 호출을합니다. 
![up1]({{ site.url }}{{ site.baseurl }}/assets/images/up1.png)

컨테이너는 프로세스이므로 시스템 호출합니다.

![up2]({{ site.url }}{{ site.baseurl }}/assets/images/up2.png)


컨테이너 이미지 안에있는 파일과 프로그램은 User Space 이라고하는 것을 구성합니다. 
컨테이너가 시작되면 컨테이너 이미지에서 프로그램이 메모리에 로드되며 이때 System calls을 통해 Kernel Space에 요청하게 됩니다.
여기서 User space와 Kernel space간의 연결은 매우 중요한 기능을 합니다.

**Uesr Space**

Uesr Space은 운영 체제에서 커널 외부에있는 모든 코드를 나타냅니다. 대부분의 유닉스 계열 운영 체제 (Linux 포함)에는 모든 종류의 유틸리티, 프로그래밍 언어 및 그래픽 도구가 사전 패키지로 제공되는 user space application 입니다. 이것을 "userland"이라고합니다.
Userland application에는 C, Java, Python, Ruby 및 기타 언어로 작성된 프로그램이 포함될 수 있습니다. 컨테이너화에서 이러한 프로그램은 일반적으로 Docker와 같은 컨테이너 이미지 형식으로 제공됩니다. 
System calls은 커널에 요청하여 User Application(컨테이너 또는 비 컨테이너)가 데이터에 액세스 할 수 있습니다. 예를 들어 메모리 할당 또는 파일 열기가 있습니다. 


**Kernel Space**
커널은 보안, 하드웨어 및 내부 데이터 구조에 대한 추상화를 제공합니다. open() system call은 일반적으로 python, C, ruby와 그밖에 다른언어를 가져오는 데 사용됩니다. 프로그램이 XFS 파일 시스템을 비트 수준으로 변경할 수 없도록하려면 커널이 System calls을 제공하고 드라이버를 처리합니다. System calls은 POSIX 라이브러리의 일부입니다.

다음 그림에서 bash 는 자체 프로세스 ID를 요청 하는 getpid () 호출을합니다. 또한 cat 명령 은 file open() 호출 로 /etc/hosts 에 대한 액세스를 요청합니다.

![up3]({{ site.url }}{{ site.baseurl }}/assets/images/up3.png)


User Space 프로그램은 작업을 완료하기 위해 System calls을 통합니다.
```bash
ls
ps
top
bash
```

System calls에 직접적으로 매핑되는 User space 프로그램입니다.
```bash
chroot
sync
mount/umount
swapon/swapoff
```

한 계층을 더 깊이 파고 들면 다음은 위에 나열된 프로그램에 의해 호출되는 System calls의 예입니다. 일반적으로 이러한 함수는 glibc와 같은 라이브러리 또는 Ruby, Python 또는 Java Virtual Machine과 같은 인터프리터를 통해 호출됩니다.
```bash
open (files)
getpid (processes)
socket (network)
```

일반적인 프로그램은 다음과 같이 유사한 추상화 계층을 통해 커널의 리소스에 액세스합니다.
![up4]({{ site.url }}{{ site.baseurl }}/assets/images/up4.png)