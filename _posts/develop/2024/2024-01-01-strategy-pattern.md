---
title: 디자인 패턴을 이용하여 반복되는 유사한 서비스를 유연하게 설계하기
date: 2024-01-01 12:00:00 +0800
categories: [Backend, Design pattern ]
tags: [Backend, Design pattern]     # TAG names should always be lowercase
---


## 1.개요

- 반복되는 패턴의 이벤트 구현에 템플릿 메소드 패턴을 적용하여 코드의 통일성을 향상시킨다
- Spring 프레임워크의 빈의 특성과 전략 패턴을 결합하여  객체 간 결합도를 낮춘다

<br>
## 2.배경

저희 회사에서는 소비자들의 만족도와 구매촉진을 위해 다양한 이벤트를 제공하고 있습니다. 그 중에서도 가장 인기있는 이벤트는 단연 선착순 당첨 이벤트입니다. 무료 혹은 저렴한 비용으로 매력적인 아이템을 습득할 수 있어 많은 회원 분들이 적극적으로 참여하십니다.

선착순 당첨 이벤트는 공통적으로 아래 2단계를 거칩니다


> - 1)회원의 응모 가능 여부 판단
> - 2)응모 가능 시 상품 증정 


반면 이벤트 별 구체적인 프로세스는 천차만별입니다. 응모 방법의 경우 ‘하루에 특정 횟수만 응모할 수 있는 이벤트’부터 ‘포인트를 사용해야 응모 가능한 이벤트’까지 다양한 방법이 존재하며 당첨상품도 할인쿠폰, 무료음료쿠폰 등 다양한 형태가 존재합니다.

<br>
## 3.문제상황

분명 이벤트 간 핵심적인 로직은 동일하지만 디테일한 비즈니스 로직이 다른 상황입니다. 때문에 기존 레거시 코드는 공통된 템플릿 없이 각각의 이벤트를 따로 구현했습니다.
이는 아래 2가지 문제점을 가지고 있습니다

- 여러 이벤트 간 공통로직이 명시적으로 드러나지 않는다
    - 별도의 문서 없이 다른 개발자가 코드를 보았을 때 직관적으로 공통로직이 보이지 않습니다
- 구체적인 비즈니스 로직을 담고 있는 Service layer와 이를 사용하는 Controller layer가 강하게 결합되어 있다
    - 추후 기존 이벤트를 삭제하거나 신규 이벤트를 등록 할 때 Controller layer도 변경되어야 합니다. →수정에 닫혀 있음
- Service 클래스가 지나치게 무거워지는 단점이 있음
    - 이벤트가 지속적으로 추가됨에 따라 1개의 클래스가 지나치게 많은 역할을 맡고 있음
    - 이는 테스트코드 혹은 코드 수정을 어렵게 하는 요소임

아래는 당시 상황을 재현한 psedo code입니다.
설명을 위해 간략히 적었기 때문에 위 psedo코드가 간결해 보일 수도 있습니다. 하지만 실제로는 온갖 비즈니스 로직이 섞여 있어 Service 클래스의 길이가 대단히 길고 복잡하며 많은 객체와 강하게 결합되어 있었습니다

---
⛔ **리팩토링 이전 코드**
>
> 1)비즈니스 로직을 호출하는 Controller layer
>
> ```java
> @RestController
> @RequestMapping("/event")
> @RequiredArgsConstructor
> public class BadExampleEventController {
>
>     private final BadExampleEventService badExampleEventService;
>
>     @PostMapping("/eventA/apply")
>     public ResponseEntity<EventResponse> applyEventA() {
>         EventResponse eventResponse = badExampleEventService.applyEventA();
>
>         return ResponseEntity.ok(eventResponse);
>     }
>
>     @PostMapping("/eventB/apply")
>     public ResponseEntity<EventResponse> applyEventB() {
>         EventResponse eventResponse = badExampleEventService.applyEventB();
>
>         return ResponseEntity.ok(eventResponse);
>     }
>
>     @PostMapping("/eventC/apply")
>     public ResponseEntity<EventResponse> applyEventC() {
>         EventResponse eventResponse = badExampleEventService.applyEventC();
>
>         return ResponseEntity.ok(eventResponse);
>     }
> }
> ```
>
>
>
> 2)비즈니스 로직을 수행하는 Service layer
>
> ```java
> @Service
> public class BadExampleEventService {
>
>     public EventResponse applyEventA() {
>         if (checkApplyAbleEventA()) {
>             issueGoodsA();
>         }
>         return new EventResponse();
>     }
>
>     public EventResponse applyEventB() {
>         if (checkApplyAbleEventB()) {
>             issueGoodsB();
>         }
>         return new EventResponse();
>     }
>
>     public EventResponse applyEventC() {
>         if (checkApplyAbleEventC()) {
>             issueGoodsC();
>         }
>         return new EventResponse();
>     }
> }
> ```



## 4.리팩토링 결과

위 두 문제를 해결하기 위해 저는 ‘템플릿 메소드 패턴’ 과 ‘전략 패턴’을 이용하여 리팩토링을 진행하였습니다.

### 1)템플릿 메소드 패턴 적용

**추상클래스**

- 응모과정에 꼭 필요한 추상메소드를 선언하였습니다
- 구상클래스가 구현한 메소드를 이용하여 응모 메소드를 구현하였습니다 (아래 코드에서는 ‘applyEvent()’)

**구상클래스**

- 추상클래스를 상속받으므로 추상메소드를 반드시 구현해야 합니다
- 실제 응모에 필요한 비즈니스 로직을 구현합니다
- protected 접근제한자로 구체적인 메소드를 은닉했습니다
    - 이에 따라 오직 추상클래스를 통해서만 구상클래스의 메소드를 호출할 수 있습니다

---
✅ **템플릿 메소드 패턴 적용**
>
> 1) 템플릿을 정의한 추상 클래스
>
> ```java
> public abstract class EventService {
>
>     abstract boolean isTarget(String eventCode);
>
>     protected abstract boolean isApplyAble();
>
>     protected abstract void issueGoods();
>
>     public final EventResponse applyEvent() {
>         if (isApplyAble()) {
>             issueGoods();
>         }
>         return new EventResponse();
>     }
> }
> ```
>
>
> 2) 실제 비즈니스 로직을 구현한 구상 클래스 (추상 클래스 상속)
>
> ```java
> @Service
> public class EventAService extends EventService {
>
>     private final String EVENT_CODE = "EVENT_A";
>
>     @Override
>     boolean isTarget(String eventCode) {
>         return eventCode.equals(EVENT_CODE);
>     }
>
>     @Override
>     protected boolean isApplyAble() {
>         // do business logic
>         return false;
>     }
>
>     @Override
>     protected void issueGoods() {
>         // do business logic
>     }
> }
> ```


### 2)전략 패턴으로 구상클래스 주입

**Factory 클래스**

- **Spring framework의 문법 상 특정 클래스의 리스트의 주입을 선언하면 특정 클래스 혹은 인터페이스를 상속받는 모든 Bean을 주입받을 수 있습니다**
- 이전 추상클래스에서 강제했던 isTarget 메소드를 이용하여 eventCode에 해당하는 구상클래스를 반환합니다.
- for문 덕분에 템플릿 메소드 패턴이 적용된 구상클래스들의 구성이 바뀌어도 전략 클래스는 변경이 필요 없습니다

**Controller 클래스**

- 추상클래스와 Factory 클래스에만 의존합니다
- 추상클래스의 applyEvent() 메소드만 사용하므로 구체적인 구현에는 신경쓰지 않아도 됩니다.

---
✅ Factory 패턴으로 구상클래스 주입

> 1) 구상 클래스를 리턴하는 전략 클래스
>
> ```java
> @Component
> @RequiredArgsConstructor
> public class EventServiceFactory {
>
>     private final List<EventService> eventServices;
>
>     public EventService getConcreteEventService(String eventCode) throws Exception {
>         for (EventService eventService : eventServices) {
>             if (eventService.isTarget(eventCode)) {
>                 return eventService;
>             }
>         }
>         throw new Exception();
>     }
> }
> ```
>
> 2) Factory 객체를 통해 구상 클래스를 주입받는 Controller 객체
>
> ```java
> @RestController
> @RequestMapping("/event")
> @RequiredArgsConstructor
> public class GoodExampleEventController {
>
>     private final EventServiceFactory eventServiceFactory;
>
>     @PostMapping("/{eventCode}/apply")
>     public ResponseEntity<EventResponse> applyEventA(@PathVariable String eventCode) throws Exception {
>
>         EventService eventService = eventServiceFactory.getConcreteEventService(eventCode);
>
>         EventResponse eventResponse = eventService.applyEvent();
>
>         return ResponseEntity.ok(eventResponse);
>     }
> }
> ```



### 3-1)장점

- 템플릿 강제
    - 구상클래스는 추상메소드를 반드시 구현해야 하므로 모든 이벤트가 동일한 템플릿을 공유합니다
- OCP 및 DIP 원칙 만족
    - Controller객체와 Factory 객체는 구상클래스의 변경에 영향을 받지 않습니다
    - Controller 객체는 추상클래스에 의존합니다
        - 덕분에 컨트롤러와 서비스 레이어 간의 결합이 느슨해졌습니다.
        - 또한 코드의 재사용성을 늘려 중복코드가 제거되었습니다
- 특정 이벤트를 추가하거나 삭제해야 할 경우 구상 클래스 (Service 클래스)를 추가 혹은 삭제하는 것 만으로 간단히 구현이 가능합니다.

### 3-2)단점

**~~이벤트 마다 배정되는 이벤트 코드는  Unique해야 한다~~**

- ~~Factory 클래스에서는 for 문을 통해 구상클래스를 반환합니다. 이 때 다른 이벤트들이 동일한 이벤트코드를 가질 경우 예상하지 못한 결과를 초래하며 컴파일 단계에서는 문제를 발견할 수 없습니다~~
- ~~저희 회사에서는 DB의 컬럼에 UNIQUE 제약조건을 걸고 관리하고 있습니다. 그러나 DB레벨의 데이터와 구상클래스 간의 관계는 필연적이지 않습니다~~
    - ~~따라서 구상클래스에 이벤트 코드를 선언할 때는 기존 코드와 중복되지 않도록 유의해야 합니다~~
- (24.07) 위 문제는 Factory 클래스에서 eventServices를 주입받을 때 map을 이용하여 해결했습니다.
- 위 방법으로 컨테이너에 빈이 등록되는 시점에 unique 제약조건을 검증할 수 있도록 개선했습니다

✅ 수정된 예시

```java
@Service
public class PaymentInitStrategyFactory {

    private final HashMap<String, PaymentInitStrategy> strategyMap = new HashMap<>();

    public PaymentInitStrategyFactory(List<PaymentInitStrategy> paymentInitStrategyList) {
        for (PaymentInitStrategy paymentInitStrategy : paymentInitStrategyList) {
            if (!strategyMap.containsKey(paymentInitStrategy.getEventType())) {
                strategyMap.put(paymentInitStrategy.getEventType(), paymentInitStrategy);
            } else {
                throw new IllegalArgumentException("Duplicated PaymentInitStrategy is not allowed");
            }
        }
    }

    public PaymentInitStrategy getInitStrategy(String eventType){
        return strategyMap.get(eventType);
    }
}
```


## 추후목표

- ~~Factory 클래스가 빈 컨테이너에 등록되는 시점에 구상클래스 간 중복되는 이벤트코드가 없는지 검증하는 로직을 추가하고자 합니다.~~
    - ~~이를 구현하면 최소한 빈 등록 시점에 문제를 발견할 수 있습니다.~~
