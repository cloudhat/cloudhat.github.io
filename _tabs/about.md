---
# the default layout is 'page'
icon: fas fa-info-circle
order: 4
---


# 김산 (cloudhat)

![share-slack](/assets/img/profile.png){: width="250" .left}



### Contact

Email : cloudhat17@gmail.com

### Channel

Blog : [https://github.com/cloudhat/cloudhat.github.io](https://github.com/cloudhat/cloudhat.github.io)

Github : [https://github.com/cloudhat](https://github.com/cloudhat)

<br>
<br>
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# Introduce.

안녕하세요. 문제해결이 게임처럼 즐거운 4년차 백엔드 개발자 김산입니다.

- 항상 깊이 있게 고민하고 최선의 선택을 하기 위해 노력합니다.
    - 제 고민과 성과를 기술블로그에 정리하고 있습니다. 
- [기술적 복잡도와 생산성의 trade-off를 이해하고 있으며 적절한 수준의 지속가능한 코드를 추구합니다.](https://cloudhat.github.io/posts/sustainable-software/)
    - 누구나 이해할 수 있는 명쾌한 코드를 작성하기 위해 노력하며 오버엔지니어링을 경계합니다.
- [조직의 생산성에도 관심이 많아 사내에서 Project manager 역할도 겸하고 있습니다.](https://cloudhat.github.io/posts/agile-for-agile/)
    - 에자일 프로세스를 도입하여 팀의 작업속도를 3배 이상 향상한 경험이 있습니다.
- 성장에 관심이 많아 매일 꾸준히 학습하고 있으며 지식을 나누는 것을 좋아합니다.



<br>
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# Work Experience.

## 이지태스크 (2024.08 ~ 현재) **(**[🔗](https://easytask.co.kr/)**)**
**동료들로부터 프로덕트에 대한 열정이 진심이며 [특히 비개발자에게 서버 단의 문제를 쉽고 상세하게 풀어낸다는 평가를 받았습니다.](https://cloudhat.github.io/posts/developer-possible-today/)**

사용 기술 
-  Spring boot, JPA, MySQL, Redis, Jenkins, ArgoCD, Kubernetes, AWS

프로젝트
- Backend & Infra
    - [Kafka & Debezium을 이용한 CDC 구현](https://cloudhat.github.io/posts/debezium-kafka-CDC/)
        - 엔티티 변경 로직과 로그 수집 로직을 분리, 로그 수집을 비동기로 처리하여 높은 가용성 보장
        - Retry & DLQ로 안정적인 예외처리 구현
        - Binlog의 file, position 으로 멱등성 보완, Timestamp로 순서보장 보완
    - [레거시 시스템 리팩토링으로 SonarQube 기준 code smell 50개 이상 제거](https://cloudhat.github.io/posts/refactoring/)
        - 비즈니스 로직을 도메인 엔티티에 이관하여 응집성 향상
        - 중복되는 공통 로직을 파사드 패턴으로 간결하게 수정
        - RESTful 원칙에 맞게 API 분리
    - [쿠버네티스 배포 시간 단축 (Pod 생성 속도 2배 이상 향상)](https://cloudhat.github.io/posts/kubernetes-deployment/)
        - slow starting containers를 안전하고 빠르게 실행하기 위해 Startup probe 추가
        - 기존의 Rolling 배포전략을 Blue/Green과 유사하게 작동하도록 변경
        - 명시적으로 warming up을 안정적으로 수행
    - [ELK Stack을 이용한 로그 모니터링 시스템 구축](https://cloudhat.github.io/posts/ELK-in-local-env/)
    - [CQRS를 이용한 쿼리 최적화 관련 글 사내 공유](https://cloudhat.github.io/posts/CQRS-concept/)
- ETC
    - [PM 역할 담당 및 팀 내에 에자일 프로세스 도입을 통해 작업속도 3배 이상 향상](https://cloudhat.github.io/posts/agile-for-agile/)
        - 스프린트 및 회고를 통해 생산성 향상
        - 칸반보드, 간트차트로 일정 시각화
    - [개발자 채용 담당, 채용 공고 개선으로 지원율 10배 이상 증가 달성](https://cloudhat.github.io/posts/recruitment-process-improvement/)

  
---

## 리미티드 포티 (2022.05 ~ 2024.08) **(**[🔗](https://www.limited40.com)**)**
**선배들로부터 버그 없이 안정적으로 기능을 구현하며 연차에 비해서도 많은 지식을 학습했다는 평가를 받았습니다.**

주요 프로젝트 
- [코카-콜라 공식 스토어 개발 및 운영 ](https://cokeplay.cocacola.co.kr/main)
    - 추천시스템, 주문, 배송 시스템 개발 및 유지보수
    - 선착순 이벤트 기획 및 개발
    - 백오피스 기획, 개발, 유지보수


사용 기술 
  - Spring boot, MyBatis ,JSP ,MySQL ,Redis


프로젝트
- 추천시스템, 결제시스템, 관리자 페이지 기능 등 이커머스 서비스 운영 전반에 대해 개발 및 레거시 리팩토링
- [초당 1천 이상의 Request를 수용 가능한 선착순 시스템 구현](https://cloudhat.github.io/posts/distributed-lock-redis-FCFS/)
    - Redis Redisson을 이용한 Distributed Lock을 AOP로 간결하게 구현
    - Redis를 사용하여 SPOF였던 RDB의 부담을 경감
    - In-memory DB인 Redis의 특성을 이용하여 비즈니스 로직 속도 향상
- [인덱싱, 캐싱 등의 방법으로 초당 1만 이상의 Request를 안정적으로 처리](https://cloudhat.github.io/posts/optimization/)
    - CQRS을 고려하여 부하가 많이 걸리는 Query 모델을 분리 및 비동기 처리
- [객체지향 관점에서 레거시 코드 리팩토링](https://cloudhat.github.io/posts/strategy-pattern/)
    - 템플릿 메소드 패턴과 전략패턴으로 OCP 및 DIP 원칙에 부합하는 코드 작성    
- 테스트 코드가 전무한 상황에서 테스트 작성 및 테스트 코드 작성 문화 전파 

<br>
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Skill.

## Backend

▪️Spring Boot  ▪️Spring MVC  ▪️JPA ▪️QueryDSL ▪️JUnit

▪️MySQL  ▪️Redis  ▪️Kafka  ▪️Docker ▪️Python 

## Infra

▪️Jenkins  ▪️ArgoCD  ▪️Kubernetes  ▪️AWS


━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

# Side project.

##  **[이커머스 주문-결제 아키텍쳐](https://cloudhat.github.io/posts/payment-system/)**


(2024.04 ~2024.05)

- 토이프로젝트로 주문-결제 시스템 설계 서비스 구현
- 지난 2년 간의 이커머스 운영으로 얻은 노하우 정리
- 새로운 비즈니스 요구에 맞게 유연하게 확장 가능하도록 설계
- 장애상황에 안정적으로 대응 가능하도록 설계
- 사용기술 : Spring Boot, JPA, H2 Database

##  **[글또 10기 참여](https://geultto.github.io/blog/geultto-summary/)**


(2023.12 ~2024.05)

- 개발자 글쓰기 모임 ‘글또’ 참여
- 글또 내에서 책 ‘클린코드’ 독서 모임 주최
- 기간 동안 총 7개의 글 작성 및 꾸준히 글을 작성하는 습관 획득

## 공적인 사적모임 홈페이지 제작

(2023.03 ~2023.08)

- 국제개발협력 청년  모임 ‘공사모’의 홈페이지 제작
- 사용기술 : Spring Boot, JPA, MySQL ,Redis, AWS
- 백엔드 팀 리드
    - 코드리뷰를 통해 팀원이  N+1문제를 해결하도록 도우는 등 ORM을 적절히 사용할 수 있도록 보조함.
    
- AWS 인프라 세팅

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# Education.

### **NestStep ATDD, 클린 코드 with Spring 7기 수강 (**[🔗](https://edu.nextstep.camp/c/R89PYi5H/)**)**

(2023.06 ~ 2023.08) : 인수 테스트 주도 개발 및 객체지향 프로그래밍 학습

### 중앙대학교 심리학과 수료

(2015.03 ~ 2023.06) : **알고리즘,네트워크 등** **컴퓨터공학 전공 8과목 수강**
