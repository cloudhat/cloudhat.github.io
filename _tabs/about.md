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

안녕하세요. [문제해결이 삶의 행복인 개발자](https://cloudhat.github.io/posts/coding-is-passion/#%EB%82%AD%EB%A7%8C%EC%A0%81-%EC%9D%B4%EC%9C%A0-2%EA%B0%80%EC%A7%80) 김산입니다.
- 저는 한정된 상황에서 trade-off를 고려하여 최선의 결과를 내기 위해 노력하는 개발자입니다.
  - 항상 깊이 있게 고민하고 있으며 회고를 저의 자산으로 삼고 있습니다. 
  - **제 고민과 성과를 기술블로그에 정리하고 있습니다. 자기소개에서 하이퍼링크(밑줄)이 적용된 문장을 클릭하시면 관련 글을 보실 수 있습니다.**
- 저는 강한 책임감을 가진 개발자입니다.
  - 깊이 있는 고민과 별개로 기한 내에 반드시 업무를 완료합니다. 
  - 365일, 주말 아침 6시를 Slack 에러 채널을 확인하며 시작합니다.
- 저는 능동적으로 회사에 기여하는 개발자입니다.
  - 로그, 데이터, Voc를 분석하여 능동적으로 백엔드 서버를 개선하고 있습니다.
  - PM을 맡아 개발속도를 3배 이상 향상시켜 프로젝트를 성공적으로 이끈 경험이 있습니다.
  - 고객에게 좋은 유저 경험을 제공하고 싶어 동료들과 적극적으로 기획을 완성하고 있습니다.


<br>
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# Work Experience.

## 이지태스크 (2024.08 ~ 2025.11)  **(**[🔗](https://easytask.co.kr/)**)**

이지태스크는 70,000명의 회원을 보유한 프리랜서 매칭 및 온라인 사무보조 플랫폼 스타트업입니다. 

저는 아키텍쳐를 설계 및 구현하고 기획을 능동적으로 주도하여 회사에 기여했습니다.

<br>

### 사용 기술 
-  Spring boot, JPA, MySQL, Redis, Jenkins, ArgoCD, Kubernetes, AWS, Kafka

### 프로젝트
- Backend & Infra
    - [최종적 일관성과 유저경험을 고려한 기프티콘 주문&결제 아키텍쳐 설계 및 구현](https://cloudhat.github.io/posts/gifticon-payment-architecture/)
        - 상황
          - 기프티콘 결제를 위해 외부 API를 사용하므로 장애 시 일관성이 보장되지 않는 문제 발생 가능성 존재
        - 해결
          - 재처리, 멱등성을 고려한 아키텍쳐를 설계하여 최종적 일관성 보장
          - 적절한 예외처리로 유저에게 응답을 제공하여 유저경험 보장
    - [EntityListeners를 이용한 엔티티 변경 로그 수집](https://cloudhat.github.io/posts/debezium-kafka-CDC/)
        - 상황
          - 기존 구조 상 ‘로그 수집 로직’을 모든 ‘엔티티 변경 로직’에 붙이는 방식으로 처리하고 있어 누락 가능
          - 짧은 개발 시간 내에 누락 없이 완벽하게 모든 로그를 수집해야 하는 상황 
        - 해결
          - 1차 개선 :  Kafka & Debezium을 이용하여 CDC로 모든 로그 수집
            - Retry & Dead letter queue로 안정적인 예외처리 구현, Binlog의 file, position 으로 멱등성 보완
          - 2차 개선 : EntityListeners를 이용하여 모든 로그를 자동으로 수집하도록 변경, 스트랭글러 패턴으로 점진적으로 CDC 구현을 제거
    - [레거시 시스템 리팩토링으로 SonarQube 기준 code smell 50개 이상 제거](https://cloudhat.github.io/posts/refactoring/)
        - 비즈니스 로직을 도메인 엔티티에 이관하여 응집성 향상
        - 중복되는 공통 로직을 파사드 패턴으로 간결하게 수정
        - RESTful 원칙에 맞게 API 분리
    - [쿠버네티스 배포 시간 단축 (Pod 생성 속도 2배 이상 향상)](https://cloudhat.github.io/posts/kubernetes-deployment/)
        - slow starting containers를 안전하고 빠르게 실행하기 위해 Startup probe 추가
        - 기존의 Rolling 배포전략을 Blue/Green과 유사하게 작동하도록 변경
        - 명시적으로 warming up을 안정적으로 수행
    - [ELK Stack을 이용한 로그 모니터링 시스템 구축](https://cloudhat.github.io/posts/ELK-in-local-env/)
- ETC
    - [PM 역할 담당 및 팀 내에 에자일 프로세스 도입을 통해 작업속도 3배 이상 향상](https://cloudhat.github.io/posts/agile-for-agile/)
        - 상황
          - 워터폴 방식으로 진행되었으며 일정이 조율되지 않아 일정에 차질 발생
        - 해결
          - PM을 자처하여 직접 에자일을 도입
        - 결과
          - 스프린트 및 회고를 통해 생산성 향상, 칸반보드, 간트차트로 일정 시각화
    - [사용자 경험을 중심으로 리워드 포인트 시스템 기획 리드](https://cloudhat.github.io/posts/point-system-service-design/)
    - [개발자 채용 담당, 채용 공고 개선으로 지원율 10배 이상 증가 달성](https://cloudhat.github.io/posts/recruitment-process-improvement/)

<br>

---

## 리미티드 포티 (2022.05 ~ 2024.08) **(**[🔗](https://www.limited40.com)**)**

리미티드 포티는 다양한 서비스를 개발 및 운영하는 SI회사입니다. 

저는 이커머스 서비스의 시스템을 최적화하고 안정적으로 트래픽을 처리하여 회사에 기여했습니다.

<br>

### 담당 업무 
- 코카-콜라 공식 스토어 개발 및 운영   [(🔗)](https://cokeplay.cocacola.co.kr/main)
    - 추천시스템, 주문, 배송 시스템 개발 및 유지보수
    - 선착순 이벤트 기획 및 개발
    - 백오피스 기획, 개발, 유지보수


### 사용 기술 
  - Spring boot, MyBatis, JSP, MySQL, Redis


### 프로젝트
- 추천시스템, 결제시스템, 관리자 페이지 기능 등 이커머스 서비스 운영 전반에 대해 개발 및 레거시 리팩토링
- [1천 RPS(Request Per Second) 이상의 트래픽을 수용 가능한 선착순 쿠폰 시스템 구현](https://cloudhat.github.io/posts/distributed-lock-redis-FCFS/)
  - 상황
    - 기존 구현 방식은 동시성 문제를 고려하지 않아 초과발급 문제가 발생
    - 인프라 상황 상 RDB가 대부분의 부하를 감당하여 Single point of failure로 작용
  - 해결
    - Redisson 라이브러리를 이용한 Pessimistic Lock을 AOP로 간결하게 구현 및 동시성 문제 해결
    - Redis로 Lock을 처리하여 RDB에 가해지는 부하를 분산
    - In-memory DB, Single Thread인 Redis의 특성을 이용하여 비즈니스 로직 속도 향상
  - 결과
    - 실제 이벤트에서 1천 RPS 이상의 트래픽을 안정적으로 처리
- [인덱싱, 캐싱 등의 방법으로 1만 RPS(Request Per Second) 이상의 트래픽을 안정적으로 처리](https://cloudhat.github.io/posts/optimization/)
  - 상황
    - 이벤트 시 대량의 트래픽이 유입되어 RDB가 Single point of failure으로 작용
  - 해결
    - CQRS을 고려하여 기존 실시간 집계 Query 모델을 비동기 집계 방식으로 변경
    - Read replica를 사용하여 SELECT 부하 분산
    - Selectivity를 고려하여 Index설정
  - 결과
    - 1만 RPS 이상의 트래픽을 안정적으로 처리
- [객체지향 관점에서 레거시 코드 리팩토링](https://cloudhat.github.io/posts/strategy-pattern/)
    - 템플릿 메소드 패턴과 전략패턴으로 OCP 및 DIP 원칙에 부합하는 코드 작성    
- 테스트 코드가 전무한 상황에서 테스트 작성으로 안정적인 개발환경 구성


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

##  **[글또 9기 참여](https://geultto.github.io/blog/geultto-summary/)**


(2023.12 ~2024.05)

- 개발자 글쓰기 모임 ‘글또’ 참여
- 글또 내에서 책 ‘클린코드’ 독서 모임 주최
- 기간 동안 총 7개의 글 작성 및 꾸준히 글을 작성하는 습관 획득

## 공적인 사적모임 홈페이지 제작

(2023.03 ~2023.08)

- 국제개발협력 청년  모임 ‘공사모’의 홈페이지 제작
- 사용기술 : Spring Boot, JPA, MySQL ,Redis, AWS
- 백엔드 팀 리드
  - 백엔드 아키텍쳐 설계
  - 코드리뷰를 통해 팀원이 N+1문제를 해결하도록 도우는 등 ORM을 적절히 사용할 수 있도록 보조함.
  - Redis를 이용하여 조회 수 집계를 처리하도록 설계    
- AWS 인프라 세팅

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# Education.

### **NestStep ATDD, 클린 코드 with Spring 7기 수강 (**[🔗](https://edu.nextstep.camp/c/R89PYi5H/)**)**

(2023.06 ~ 2023.08) : 인수 테스트 주도 개발 및 객체지향 프로그래밍 학습

### 중앙대학교 심리학과 수료

(2015.03 ~ 2023.06) : **자료구조, 알고리즘, 네트워크 등** **컴퓨터공학 전공 8과목 수강**
