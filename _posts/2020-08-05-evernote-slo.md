---
title: "SLO 문화"
date: 2020-08-05 18:26:28 +0900
categories: SRE
classes: wide
tags:
  - SRE

---
## SLO 문화

SRE Workbook 중 evernote와 The Home Depot의 SLO 문화 이야기 간략하게 나옵니다.  서비스 수준 목표 (SLO)는 서비스의 안정성을 위한 목표 수준을 뜻합니다. SLO는 안정성에 대한 데이터 중심의 결정을 내리는 데 중요하므로 SRE의 핵심이라고 정의되어 있습니다. 

## Evernote SLO
Evernote 사례에서 흥미로웠던 점은 GCP를 사용하는데 GCP의 SLO 기준과 Evernote의 SLO 기준은 견해가 다를 수 있기 때문에 글로벌 서비스를 하는 Evernote의 경우에는 작은 문제가 될 수 없다는 것입니다. 사례에서는 Google 엔지니어팀과 소통했다고만 나와있지만 과연 소규모 기업에서는 이러한 문제가 발생한 경우에 어떻게 SLO 기준을 좁혀갈 수 있을지 의문이 들었습니다.

## 지속적인 SLO 안정화를 위한 노력
SRE 역할은 서비스의 안정성을 담당하기 때문에 SRE는 종종 서비스 모니터링 시스템 및 해당 기능에 대해 잘 알고 있어야합니다. 이 지식이 없으면 SRE는 어디를보아야하는지, 비정상적인 행동을 식별하는 방법 또는 비상시 필요한 정보를 찾는 방법을 모를 수 있습니다. 서비스 업종 및 문화별로 SLO 기준은 다르며 그 기준은 SRE가 판단해야하기 때문에 역할의 중요도가 높다는 것을 느꼈습니다.

원본 링크
* https://landing.google.com/sre/workbook/chapters/slo-engineering-case-studies/
