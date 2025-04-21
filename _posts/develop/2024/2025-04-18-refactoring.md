---
title: 업무요청 기능 리팩토링 
date: 2025-04-17 20:00:00 +0800
categories: [Retrospective, Refactoring]
tags: [Retrospective, Refactoring]    
---
<br>




## 배경

업무요청은 [이지태스크](https://easytask.co.kr/)의 핵심 서비스인 ‘온라인 업무보조’를 사용하기 위한 첫 단계로 고객의 요구사항을 다루는 중요한 로직입니다.  

그러나 그 중요도와 별개로 상당한 기술부채가 쌓여 있어 서비스를 개선하기 어려웠으며 작은 변경도 장애로 이어질 수 있는 상황이었습니다. 

마침 최근 업무요청 관련 기능추가 프로젝트에 참여한 것을 계기로 짬을 내어 리팩토링을 신규개발과 병행했습니다.

비록 가시적인 비즈니스 임팩트를 보일 수 있는 작업은 아니었지만 묵묵히 리팩토링을 진행한 결과 기술부채를 상당 부분 해소하는 성과를 얻었습니다.

<br>
## 기술부채 및 리팩토링 상세

### 난잡한 코드를 통일성 있게 간소화

- **문제**
    - 단기업무와 정기업무의 요청은 공통적으로 20여가지의 로직을 동일한 순서로 수행합니다
        - 예시 : 업무요청서 저장 , 포인트 사용, 채팅방 생성, 업무제안 생성 등
    - 전해 듣기로는, 과거에 단기업무가 먼저 구현 되었고 정기업무는 그 후에 퀄리티를 고려하지 못하고 단기업무 코드에 급하게 덧붙이는 형태로 개발되었다고 합니다. 기술부채가 체계적으로 관리되지 못했던 탓에  비즈니스 로직이 각종 클래스에 무분별하게 구현되어 있어 응집성과 가독성이 매우 떨어지는 상태였습니다.
- **개선**
    - request dto 통일성 있게 간소화
        - 저는 request dto 또한 클라이언트 - 서버 간의 인터페이스 역할을 하는 중요한 요소라고 생각합니다.
        - request dto의 역할을 간결하고 명확하게 드러내기 위해 단기, 정기업무가 공통으로 사용하는 필드를 모아 단일한 클래스로 생성하고 정기업무에서만 사용하는 필드는 별도로 다루도록 request dto를 구현했습니다
        
        ```java
        
        //공통으로 사용하는 클래스
        public class 공통_클래스 {
        
                private String 업무요청_제목;
                private String 업무요청_상세;
                .
                .
                .
            }
            
        public class 단기업무_요청_dto {
        
                private 공통_클래스 공통클래스;
                .
                .
                .
            }
            
        public class 정기업무_요청_dto {
        
                private 공통_클래스 공통클래스;
                
                private LocalDate 종료일;
                private List<DayOfTheWeekType> 업무 요일
                .
                .
            }
        ```
        

### 공통으로 사용하는 로직을 파사드 패턴으로 묶기

- 상술한대로 단기업무와 정기업무 요청은 공통적으로 20여가지의 서브 로직을 동일한 순서로 수행합니다
- 해당 특징을 더 유연하게 반영하기 위해 아래의 3가지 디자인패턴을 고민했고 결과적으로 퍼사드 패턴만 적용했습니다.
    - 채택한 디자인 패턴
        - 퍼사드 패턴
            - 복잡한 서브시스템을 단순한 인터페이스로 제공하기 위해 채택했습니다
            - 컨트롤러는 퍼사드 클래스만 호출하고 퍼사드 클래스가 일련의 로직을 호출하도록 구현하였습니다.
    - 기각한 디자인 패턴
        - 전략 패턴
            - 업무요청의 타입이 단기와 정기 외에 더 이상 늘어나지 않을 것으로 판단하여 기각했습니다.
        - 템플릿 메소드 패턴
            - 템플릿 메소드 패턴을 사용하려면 서브 클래스에서 상세 동작을 구현해야 합니다.
            - 그러나 단기, 정기업무 모두 거의 동일한 상세 동작을 하므로 굳이 각 서브 클래스를 구현할 메리트가 없어 기각했습니다.
    - 공통 로직을 private method로 묶기
        - 특별한 구조는 아니지만 간결한 코드를 위해 공통 로직을 private method로 묶고 flag 변수로 구분하도록 하였습니다.
        
        ```java
        public class 업무요청 {
        
                public void 단기_업무요청_생성(){
        		        private boolean isRecurring = false;
        		        
        		        this.업무요청서_저장(isRecurring);
        		        this.포인트_사용(isRecurring);
        		        .
        		        .
        		        .
                }
                
                public void 정기_업무요청_생성(){
        		        private boolean isRecurring = true;
        		        
        		        this.업무요청서_저장(isRecurring);
        		        this.포인트_사용(isRecurring);
        		        .
        		        .
        		        .
                }
                
                private void 업무요청서_저장(boolean isRecurring){
        		        //업무요청서 저장
                }
                private void 포인트_사용(boolean isRecurring){
        		        //포인트 사용
                }
            }
        ```
        

### 데이터 정합성을 위한 트랜잭션 처리

- 문제
    - 정기업무에 사용되는 엔티티는 크게 1)’업무요청 내용’ 엔티티 2)’정기업무 스케쥴’ 엔티티, 2가지가 있습니다. 2개의 엔티티는 정합성을 위해 반드시 함께 원자적으로 처리되어야 합니다.
    - 기존의 정기업무 요청 서비스는 ’업무요청 내용’ 엔티티와 ’정기업무 스케쥴’ 엔티티를 생성하는 API가 분리되어 있고 이를 순차적으로 호출하는 형태였습니다.
    - 위의 형태는 ’업무요청 내용’ 엔티티 API 성공 후 ’정기업무 스케쥴’ 엔티티 API가 실패할 경우 정합성이 깨지는 심각한 문제점을 가지고 있습니다
- 해결
    - 단일 API 내에서 ’업무요청 내용’ 엔티티와 ’정기업무 스케쥴’ 엔티티 생성을 처리하도록 하고 트랜잭션으로 묶어서 처리했습니다

### 리치 도메인 모델

- 레거시 코드는 각종 비즈니스 로직이 service layer에 흩어져 있었고 중복되는 코드도 많았습니다.
- 코드의 응집성을 높이기 위해 도메인 객체에 비즈니스 로직을 옮기고 service layer는 도메인 객체의 메소드를 호출하는 방식으로 수정했습니다.

### RESTful API 원칙 준수

- 레거시 코드는 하나의 API에 flag 값을 이용해서 POST와 PATCH 역할을 둘 다 수행하거나
- ‘임시저장’과 ‘업무요청서’ 엔티티는 아예 다른 자원임에도 동일한 API에서 처리하는 등 Restful하지 않은 구조였습니다.
- 이는 직관적이지 않고 유지보수도 어려운 구조이므로 RESTful 원칙에 맞게 API를 분리했습니다.

### 스윔 레인 다이어그램

- 업무 요청 후 포인트 정산, 매칭 성공 or 실패, 그에 따른 알림톡 등 다양한 형태의 후속작업이 발생합니다.
- 후속작업은 고객 뿐만 아니라 여러 명의 프리랜서가 개입하므로 정책 상 여러 case가 복잡하게 얽혀있습니다.
- 스윔 레인 다이어그렘을 작성하여 후속작업의 다양한 case를 명확하게 시각화했습니다.

### 모듈 의존성을 고려하여 임시저장 내용을 문자열로 직렬화

**문제**

- 임시저장 엔티티는 Redis, 업무요청 엔티티는 RDB에 저장하고 있습니다..
- 업무요청 엔티티에 새로운 필드가 추가될 때 마다 임시저장 엔티티에도 필드를 추가해줘야 하는 상황이었습니다.
    - 업무요청과 임시저장 둘 다 동일한 필드를 가지고 있어야 하나 기존 구조 상 Redis 모듈과 RDB 모듈이 서로 분리되어 있어 서로 클래스를 공유할 수가 없기 때문입니다
- 위 제약조건 때문에 업무요청 엔티티를 임시저장 엔티티로 변환해주는 40줄이 넘는 끔찍한 Builder 코드가 존재했습니다.
    - *예시 : 실제 필드 이름은 보안을 위해 알파벳으로 대체했습니다
        
        ```java
        public static 임시저장_엔티티 create_임시저장_엔티티() {
            return 임시저장_엔티티.builder()
                    .a(this.getA())
                    .b(this.getB())
                    .c(this.getC())
                    .d(this.getD())
                    .e(this.getE())
                    .f(this.getF())
                    .g(this.getG())
                    .h(this.getH())
                    .i(this.getI())
                    .j(this.getJ())
                    .k(this.getK())
                    .l(this.getL())
                    .m(this.getM())
                    .n(this.getN())
                    .o(this.getO())
                    .p(this.getP())
                    .q(this.getQ())
                    .r(this.getR())
                    .s(this.getS())
                    .t(this.getT().stream()
                            .map(x -> 임시저장_엔티티.A.builder()
                                    .b(임시저장_엔티티.B.builder()
                                            .c(x.getC().getC())
                                            .d(x.getC().getD())
                                            .e(x.getC().getE())
                                            .f(x.getC().getF())
                                            .g(x.getC().getG())
                                            .build())
                                    .h(x.getH().stream()
                                            .map(y -> this.B.builder()
                                                    .c(y.getC())
                                                    .d(y.getD())
                                                    .e(y.getE())
                                                    .f(y.getF())
                                                    .g(y.getG())
                                                    .build())
                                            .collect(Collectors.toList()))
                                    .build())
                            .collect(Collectors.toList()))
                    .u(this.getU())
                    .v(LocalDateTime.now())
                    .w(this.getW())
                    .x(this.getX())
                    .y(this.getY())
                    .z(Optional.ofNullable(this.getZ())
                    .build();
        }
        
        ```
        

**해결**

- 임시저장 엔티티는 문자열 필드를 가지도록 구현했습니다. 그리고 임시저장 시 업무요청 클래스를 문자열로 직렬화하여 임시저장 엔티티의 문자열 필드에 할당하도록 구현했습니다.
- 덕분에 업무요청 엔티티가 변경되더라도 임시저장 엔티티는 변경할 필요가 없어졌습니다.
- 예시
    
    ```java
    @Getter
    @NoArgsConstructor(access = AccessLevel.PROTECTED)
    @AllArgsConstructor
    @RedisHash(value = "임시저장_엔티티", timeToLive = N)
    public class 임시저장_엔티티 {
    
        @Id
        private String id;
    
        private String 직렬화된_업무요청_엔티티;
    
        private LocalDateTime createdAt;
    
        private LocalDateTime updatedAt;
    }
    
    @Getter
    @AllArgsConstructor
    @NoArgsConstructor(access = AccessLevel.PROTECTED)
    public class 업무요청_엔티티 {
    		
    		@Id
        private String id;
        .
        .
        .
        public String serialize(업무요청_엔티티 업무요청엔티티) {
            try {
                return mapper.writeValueAsString(업무요청엔티티);
            } catch (Exception e) {
                throw new RuntimeException("직렬화 실패", e);
            }
        }
    
        public static 업무요청_엔티티 deserialize(String 직렬화된_업무요청_엔티티) {
            try {
                return mapper.readValue(직렬화된_업무요청_엔티티, 업무요청_엔티티.class);
            } catch (Exception e) {
                throw new RuntimeException("역직렬화 실패", e);
            }
        }
        
    }
    ```
    

### 기타

- 주요 DB 테이블에 선제적으로 인덱스 추가
- 모든 Controller, DTO에 swagger 명시
- 유저 pk를 얻기 위해 Spring security의 SecurityContextHolder를 무분별하게 Service layer에서 호출하지 않고 컨트롤러 Layer에서 파라미터로 받도록 수정
    - 추후 쉽게 테스트코드를 작성하기 위함

<br>
## 결과

이번 개선으로 업무요청 기능이 간결한 코드와 더불어 스웨거, 플로우 차트 덕분에 누구나 쉽게 이해하고 변경할 수 있는 로직이 되었습니다.

리팩토링은 당장 극적인 임팩트를 낼 수 없다는 이유로 등한시 되는 경우도 있습니다. 그러나 저는 체계적 기술부채 관리의 관점에서 반드시 리팩토링이 꾸준히 진행되어야 한다고 생각합니다.

앞으로도 꾸준히 점진적 리팩토링을 병행하고자 합니다.

<br>
## 향후 과제

- 테스트 코드를 추가하여 서비스의 안정성을 높일 예정입니다
- CI 파이프라인에 자동화된 테스트 코드 실행을 적용할 예정입니다.
