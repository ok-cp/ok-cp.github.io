---
title: "Harbor - Clair vs Trivy"
date: 2020-08-11 18:26:28 +0900
categories: Job
classes: wide
tags:
  - kubernetes
  - harbor
  - vulnerabilityScanner

---
## Harbor 
Harbor는 정책 및 역할 기반 액세스 제어로 아티팩트를 보호하고 이미지를 스캔하고 취약성이 없는지 확인하며 이미지를 신뢰할 수있는 것으로 서명하는 오픈 소스 레지스트리입니다. CNCF 프로젝트인 Harbor는 규정 준수, 성능 및 상호 운용성을 제공하여 Kubernetes 및 Docker와 같은 클라우드 네이티브 컴퓨팅 플랫폼에서 아티팩트를 일관되고 안전하게 관리 할 수 ​​있도록 지원합니다.

## Vulnerability Scanner
Harbor는 Image에 대한 취약점 스캔을 지원하고 있습니다. 취약점 스캔은 Clair, Trivy, Anchore 방식을 지원하고 있습니다. helm 으로 설치한 Harbor는 Trivy, Clair 을 포함하고 있으며 Trivy을 기본으로 선택하고 있습니다. 테스트는 3가지 이미지를 준비하여 스캐너별로 취약점 발견차이를 알아보고자 하였습니다. 


| OSS CVE Scanner  | Vendor  | Version  | 취약점 정밀도 |
| --------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|
| [Clair](https://github.com/arminc/clair-scanner) | RedHat | v2.x  | CoreOS
| [Trivy](https://github.com/aquasecurity/trivy) | Aquasec | v0.9.1 | Alpine Linux 및 RHEL / CentOS

테스트 환경에서 사용한 Harbor Helm Chart - [ok-cp/harbor-helm](https://github.com/ok-cp/harbor-helm)



### Alpine
Alpine 이미지에서는 두 방식에서 취약점이 발견되지 않았습니다.
#### Clair
![harbor_sc001]({{ site.url }}{{ site.baseurl }}/assets/images/harbor_sc001.png)

#### Trivy
![harbor_sc006]({{ site.url }}{{ site.baseurl }}/assets/images/harbor_sc006.png)


### Python:3.8
Python 이미지에서는 Trivy에서 나오지 않은 Critical 취약점이 Clair에서는 나오지 않았습니다. 
#### Clair
![harbor_sc002]({{ site.url }}{{ site.baseurl }}/assets/images/harbor_sc002.png)
Critical 취약점은 [CVE-2019-19816](https://security-tracker.debian.org/tracker/CVE-2019-19816), [CVE-2019-19814](https://security-tracker.debian.org/tracker/CVE-2019-19814) 이며 	Linux 커널 5.0.21에서 만들어진 btrfs, f2fs 파일시스템에 대한 취약점입니다.

#### Trivy
![harbor_sc005]({{ site.url }}{{ site.baseurl }}/assets/images/harbor_sc005.png)


### OpenJDK
OpenJDK 이미지는 두방식에서 Critical 취약점이 발견되지 않았지만 Medium, Low 단계의 취약점 발견수가 많은 차이를 보였습니다.
#### Clair
![harbor_sc003]({{ site.url }}{{ site.baseurl }}/assets/images/harbor_sc003.png)


#### Trivy
![harbor_sc007]({{ site.url }}{{ site.baseurl }}/assets/images/harbor_sc007.png)

