---
title: 쿠버네티스 배포 속도 향상
date: 2025-04-23 20:00:00 +0800
categories: [Infra, Kubernetes]
tags: [Infra, Kubernetes, Deployment]    
---

<br>
## 개요

- 저희 회사는 어플리케이션 관리를 위해 쿠버네티스를 사용하고 있습니다.
- 이번에 몇 가지 설정 변경으로 안정적으로 배포 속도를 향상시켰습니다.
    - 특히 Pod 생성 속도가 약 2배 빨라졌습니다.

---
<br>
## 변경사항

### Probe 설정 변경

**1) slow starting containers를 안전하고 빠르게 실행하기 위해 Startup probe 추가**

문제점

- 저희 회사는 백엔드 서버로 Spring을 사용하고 있습니다.
- 기존 Deployment 오브젝트는 Liveness, Readiness Probe만 설정되어 있었으며 Spring의 slow starting을 대비하여 initialDelaySeconds 옵션이 100초로 지정되어 있었습니다.
- Liveness, Readiness Probe도 failureThreshold 및 periodSeconds가 default로 적용되기 때문에 위 방식으로도 배포 자체는 가능합니다. 그러나 Liveness, Readiness Probe가 Startup probe의 역할을 대신 수행한다는 점에서 바람직하지 않습니다.

개선

- 쿠버네티스 공식문서에서는 slow starting containers 를 보호하기 위해 Startup probe를 사용하는 방식을 제시하고 있습니다.
- 공식문서가 제시한 방법에 따라 Startup probe을 추가하였고 periodSeconds 를 10초로 설정하였습니다.

결과

- 기존 initialDelaySeconds 옵션을 제거한 덕분에 Pod 생성속도가 약 2배 증가했습니다.
- 또한 Liveness, Readiness Probe는 각자의 역할인 파드 재시작, 트래픽 연결에 집중할 수 있게 되었습니다.

**2) tcpSocket 방식을 httpGet 방식으로 변경**

- 기존 health check 방식을 tcpSocket 에서 httpGet으로 변경했습니다.
- port 연결 여부 보다는 정상적인 http 통신이 server의 health 체크에 적합하다고 판단했기 때문입니다.

<br>


### 기존의 Rolling 배포전략을 Blue/Green과 유사하게 작동하도록 변경

- 배포전략으로 RollingUpdate 방식을 사용하고 있습니다.
- Blue/Green 과 유사하게 작동하도록 maxSurge를 100%, maxUnavailable 를 0%로 변경했습니다.
    - 위 방식으로 세팅할 경우 replicas 수 만큼 신규 Pod가 생성되며 각각의 신규 Pod가 생성 완료될 때마다 즉각 기존의 Pod가 종료됩니다.
- 위 방식을 적용한 이유는 아래와 같습니다.
    - 서비스의 특성 상 replicas의 수가 작기 때문에 동시에 2배의 Pod가 존재해도 리소스 부담이 거의 없고
    - 동시에 모든 신규 Pod를 생성하기 때문에 더 빠른 배포속도를 기대할 수 있기 때문입니다.

---
<br>
## 배움과 도전의 즐거움

비록 큰 변경은 아니지만 이번 작업을 통해 배포 파이프라인을 조금 더 효율적으로 개선했습니다. 덕분에 입사 후 열심히 쿠버네티스를 공부한 보람을 소소하게 느꼈습니다.

대기업의 경우 이미 최적화가 완료되었거나 권한 상의 문제로 인프라를 건들 수 없는 경우가 적지 않은 것으로 알고 있습니다. 그런 점에서 스타트업은 다양한 영역을 주도적으로 작업할 수 있는 매력이 있는 것 같습니다.

---
<br>
## 참고자료

- [Kubernetes : Protect slow starting containers with startup probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-startup-probes)
