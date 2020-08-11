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
Harbor는 Image에 대한 취약점 스캔을 지원하고 있습니다. 취약점 스캔은 Clair, Trivy, Anchore 방식을 지원하고 있습니다.

| OSS CVE Scanner  | Vendor  
| --------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|
| `Clair` | RedHat  
| `Trivy` | Aquasec  

![harbor_sc004]({{ site.url }}{{ site.baseurl }}/assets/images/harbor_sc004.png)
![harbor_sc008]({{ site.url }}{{ site.baseurl }}/assets/images/harbor_sc008.png)


### Alpine
#### Clair
![harbor_sc001]({{ site.url }}{{ site.baseurl }}/assets/images/harbor_sc001.png)

#### Trivy
![harbor_sc006]({{ site.url }}{{ site.baseurl }}/assets/images/harbor_sc006.png)


### Python:3.8
#### Clair
![harbor_sc002]({{ site.url }}{{ site.baseurl }}/assets/images/harbor_sc002.png)

#### Trivy
![harbor_sc005]({{ site.url }}{{ site.baseurl }}/assets/images/harbor_sc005.png)


### OpenJDK
#### Clair
![harbor_sc003]({{ site.url }}{{ site.baseurl }}/assets/images/harbor_sc003.png)


#### Trivy
![harbor_sc007]({{ site.url }}{{ site.baseurl }}/assets/images/harbor_sc007.png)