---
title: 이커머스 주문-결제 시스템
date: 2024-05-01 12:00:00 +0800
categories: [Backend, Architecture]
tags: [Backend, Architecture ,Design pattern, OOP, Test code]     
---
## 1. 깃헙 링크

[https://github.com/cloudhat/paymentSystem](https://github.com/cloudhat/paymentSystem)

---

<br>
## 2. 개요

아래 6단계로 이루어진 이커머스 결제 서비스를 구현한 토이프로젝트입니다.

- 회원 가입 및 로그인
- 장바구니 생성
- 주문 옵션 선택(결제수단, 배송지, 할인수단 등)
- 결제
- 주문내역 조회
- 주문 취소

---

<br>
## 3. 배경 및 목적

저는 2년 동안 결제파트를 포함하여 이커머스 사이트 운영 전반에 참여하고 있습니다. 기존 레거시는 소위 ‘Bad smell’이 가득한 나쁜 코드로 가득차 있었고,  각종 장애가 시스템 전체에 전파되는 장애에 취약한 아키텍쳐를 가지고 있었습니다. 이전에는 트래픽이 적어 문제가 발생하지 않았지만 서비스가 성장함에 따라 장애가 발생하기 시작했습니다. 특히 결제시스템의 경우 결제요청이 조금만 증가해도 장애가 발생하였고, 고객의 돈이 지출되었음에도 결제내역이 DB에 기록되지 않는 등 심각한 문제가 발생했습니다.

결국 고통스러운 과정을 거쳐 레거시 시스템을 개선하였고 그 과정에서 많은 것을 배웠습니다. 그 때 얻은 소중한 경험을 코드 및 글로 정리하고자 토이프로젝트로 주문-결제 시스템을 구현했습니다.

**최선을 다했지만 아직 부족한 점이 많습니다. 피드백은 언제든지 환영입니다.**

---

<br>
## 4. 기술적 목표

### **토이프로젝트의 기술적 목표는 아래와 같습니다.**

- 1)새로운 비즈니스 요구에 맞게 유연하게 확장 가능하도록 설계
- 2)장애상황에 안정적으로 대응 가능하도록 설계

### **위 목표를 달성하기 위해 아래 3가지 기준을 중심으로 구현했습니다.**

**1)예외처리**

- 장애는 언제든지 다양한 요인으로 발생할 수 있습니다.
- 특히 주문-결제 서비스의 경우 다양한 계층에서 예외 케이스가 발생할 수 있으며 이는 사용자 경험에 큰 악영향으로 작용합니다.
- 이에 따라 가능한 사용자 경험에 악영향이 없도록 예외처리를 구현했습니다.

**2)객체지향**

- 객체지향이 항상 최고의 방법론은 아닙니다.
- 하지만 아래 2가지의 장점을 고려하여 이번 토이프로젝트에서는 객체지향 프로그래밍을 적극적으로 추구했습니다.
    - 1)안정적으로 변화에 대응할 수 있습니다.
        - 주문 및 결제기능은 고객의 지갑과 직접적으로 연관되어 있어 특히 안정성이 요구됩니다.
        - 객체지향 프로그래밍은 객체에게 역할을 적절하게 분배합니다. 덕분에 기존의 코드를 가능한 적게 수정하면서도 새로운 기능을 안정적으로 추가할 수 있습니다.
    - 2)이번 토이프로젝트에서 선택한 기술 Spring - JPA에 가장 적합한 패러다임입니다.

**3)시나리오 기반 E2E 테스트 코드 작성**

- 실제 주문 및 결제 서비스 호출 시 발생하는 프로세스와 동일하도록 시나리오에 따라 E2E 테스트로 코드를 검증하도록 구현했습니다.
    - 이는 각기 분리되어 있는 주문 및 결제 프로세스가 Production 환경에서도 안정적으로 작동하도록 신뢰성을 높이기 위함입니다.
- 복잡한 로직의 경우 unit 테스트로 보완하였습니다.

---

<br>
## 5. 구현 포인트

### 1)리치 도메인 모델

- 가능한 비즈니스 로직을 도메인에게 위임하여 리치 도메인 모델을 추구했습니다

### 2)테스트

- 로그인부터 주문취소까지 일련의 프로세스를 각 시나리오대로 검증할 수 있도록 구현하였습니다.
- H2 데이터베이스를 사용하여 테스트가 빠르게 실행될 수 있도록 구현했습니다.
- Restassured 라이브러리를 이용하여 API레벨의 E2E 테스트를 구현했습니다
    - Request, Reponse 이외의 서버 내부의 정보는 가능한 가리는 블랙박스 형식의 테스트입니다.

### 3)상품 엔티티

- product 엔티티에서 가격,재고, 판매일을 정규화 하여 product_option 엔티티를 추가했습니다.
- 이는 상품이 날짜 별로 다른 가격,재고, 판매일을 가질 수 있도록 하기 위함입니다.
    - 만약 정규화를 하지 않을 경우, 즉 product 엔티티가 모든 필드를 가지고 있을 경우 특정 날짜에 특정 가격 혹은 재고를 적용하려면 수동으로 수정하거나 스케쥴러를 사용해야 하는 불편함이 생깁니다. 이는 휴먼 에러로 이어질 수 있습니다.

![product](/assets/img/posts/2024/2024-05-01-payment-system/erd-product.png)


### 4)주문 엔티티와 결제 엔티티 분리

- 정기배송의 경우 한 번의 주문 후 여러 번의 결제가 진행됩니다.
- 따라서 주문엔티티를 정규화하여 따로 결제 엔티티를 구현하였습니다.
- 이번 프로젝트에서는 정기배송 기능을 구현하지는 않았습니다. 하지만 정기배송은 대부분의 이커머스에서 지원하는 기능이므로 오버엔지니어링을 감수하고 정규화를 하였습니다.

![order](/assets/img/posts/2024/2024-05-01-payment-system/erd-order-payment.png)

### 5)결제 서비스

```jsx
cf)이번 토이프로젝트를 포함하여 일반적인 PG사를 통한 결제 서비스는 서버의 관점에서  아래 3단계를 거칩니다

1)결제초기화 
- 총 결제금액 및 할인액수 계산, 상품 재고차감, 쿠폰 사용처리 등 결제에 필요한 엔티티 초기화 및 업데이트
- 클라이언트에게 총 결제금액 전달
2)PG사에게 결제승인요청 (혹은 네이버페이 등 간편결제사 이용)
- 1번단계를 토대로 클라이언트가 직접 PG사에게 결제수단인증 요청을 하여 결제키를 획득하고 획득한 결제키를 서버에 전달합니다.
- 서버는 클라이언트로부터 전달받은 결제키를 이용하여 PG사에게 직접 결제승인요청을 합니다.
3)결제 트랜잭션 성공 혹은 실패 후 후처리
- 결제 관련 엔티티의 상태를 성공 혹은 실패로 변경합니다.
```

**a.재고 차감 및 롤백**

- 동시성 문제를 고려하여 결제초기화 시점에 상품의 재고를 차감하도록 구현했습니다
    - 결제승인이 완료된 후 상품의 재고를 차감할 경우  재고가 마이너스가 될 수 있기 때문입니다
- 경합이 자주 발생하지 않는다고 가정하고 재고차감은 Optimistic Lock으로 구현했습니다.
    - 그러나 Optimistic Lock은 경합이 많이 발생할 수록 리소스가 낭비됩니다. 대안으로 Pessimistic Lock, Distributed Lock([링크](https://www.notion.so/24-03-Distributed-Lock-Redis-ca449dbdde8a47188a7010076e872c57?pvs=21)) 혹은 PUB/SUB 구조로 비동기로 처리하는 방법이 있습니다.

**b.전략패턴으로 결제초기화객체에게 결제 초기화 책임 위임**

- 이커머스 서비스는 다양한 결제 할인이벤트를 제공할 수 있어야 합니다 (ex: 할인쿠폰, 특정결제수단 사용 시 할인 등)
- 수 많은 이벤트들을 하나의 service layer 클래스에서 if-else로 구현하는 것은 개방-폐쇄 원칙에 어긋납니다.
- 따라서 전략패턴을 이용하여 1)Service 객체는 전략객체를 주입받고 2)결제초기화객체에게 결제초기화 역할은 위임하여 구현했습니다.

```java
//PaymentService.java (service layer 클래스)
@Transactional
public PaymentInitResponse initPayment(PaymentRequest paymentRequest, UserPrincipal userPrincipal) {
   .
   .
   //생략
   .
   .
	  //1)Factory 클래스로부터 결제초기화객체를 주입받음
    PaymentInitStrategy paymentInitStrategy = paymentInitStrategyFactory.getInitStrategy(paymentRequest.getEventType());

    try { //2)결제초기화 객체가 결제초기화 역할 수행
        Payment payment = paymentInitStrategy.getPayment(paymentRequest, member, orders);
        return new PaymentInitResponse(payment.getId(), orders.getOrderProductSummary(), payment.getTotalPayAmount(), userPrincipal.getUsername());

    } catch (Exception exception) {
        deadLetterQueueService.enqueue(orders.getOrderProducts());
        throw exception;
    }
}
```

**c.책임 연쇄 패턴을 이용하여 총 결제 금액 계산**

- 아래 2가지 조건을 만족하며 개방-폐쇄 원칙을 지키기 위해  책임 연쇄 패턴을 사용했습니다.
    - 1)결제 금액은 정해진 정책에 따라 순차적으로 계산되어야 합니다.
        - ex)비율할인쿠폰 적용 case
            - (1)총 상품 액수 합산
            - (2)계산된 총 상품 액수에서 정해진 비율로 할인
            - (3)배달비 추가
    - 2)각 계산단계 로직을 구현한 코드는 재사용될 수 있어야 합니다.
        - ex) ‘총 상품 액수 합산’ 및 ‘배달비 추가 ‘ 로직은 모든 결제에서 공통적으로 사용됨
- 예시
    - (1)기본결제 ( `DefaultPaymentInitStrategy`)
        - 상세 : 할인쿠폰 적용가능 및 배달비 적용
        
        ```java
        @Override
        @Transactional
        protected Payment initPayment(PaymentRequest paymentRequest, Member member, Orders orders) {
            List<OrderProduct> orderProducts = orders.getOrderProducts();
            List<Coupon> couponList = couponRepository.findByIdInAndMemberId(paymentRequest.getCouponIdList(), member.getId());
            Address address = memberRepository.findAddressById(paymentRequest.getAddressId(), member.getId()).orElseThrow(EntityNotFoundException::new);
        
        		//책임 연쇄 패턴 적용 
            PricePolicy pricePolicy = new ProductPricePolicy(orderProducts) //(1)총 상품 액수 합산
                    .setNextPricePolicy(new CouponPricePolicy(new Coupons(couponList))) //(2)할인쿠폰 적용
                    .setNextPricePolicy(new DeliveryFeePolicy(address)); //(3)배달비 추가
        
            List<OrderPriceHistory> orderPriceHistoryList = pricePolicy.getOrderPriceList(0, orders);
        
            int totalPayAmount = orderPriceHistoryList.stream()
                    .mapToInt(OrderPriceHistory::getAmount).sum();
        
            int totalDiscountAmount = orderPriceHistoryList.stream()
                    .filter(history -> history.getAmount() < 0)
                    .mapToInt(OrderPriceHistory::getAmount)
                    .sum();
        
            return new Payment(totalPayAmount, totalDiscountAmount, paymentRequest.getPaymentMethod(), paymentRequest.getEventType(), orders, member);
        }
        ```
        
    - (2)네이버페이 할인 결제(`NaverPayPaymentInitStrategy`)
        - 상세: 결제수단이 네이버페이인 경우 2000원 할인, 할인쿠폰은 적용 불가능, 배달비 적용
        
        ```java
        @Override
        @Transactional
        protected Payment initPayment(PaymentRequest paymentRequest, Member member, Orders orders) {
        
            //결제수단이 네이버페이가 아닐 경우 예외 처리
            if(!paymentRequest.getPaymentMethod().equals(PaymentMethod.NAVER_PAY)){
                throw new IllegalArgumentException("Payment method is not NAVER_PAY");
            }
        
            List<OrderProduct> orderProducts = orders.getOrderProducts();
            Address address = memberRepository.findAddressById(paymentRequest.getAddressId(), member.getId()).orElseThrow(EntityNotFoundException::new);
        
        		//책임 연쇄 패턴 적용 
            PricePolicy pricePolicy = new ProductPricePolicy(orderProducts) //(1)총 상품 액수 합산
                    .setNextPricePolicy(new NaverPayPolicy()) //(2)네이버페이 할인 적용
                    .setNextPricePolicy(new DeliveryFeePolicy(address)); //(3)배달비 추가
        
            List<OrderPriceHistory> orderPriceHistoryList = pricePolicy.getOrderPriceList(0, orders);
        
            int totalPayAmount = orderPriceHistoryList.stream()
                    .mapToInt(OrderPriceHistory::getAmount).sum();
        
            return new Payment(totalPayAmount, 0, paymentRequest.getPaymentMethod(), paymentRequest.getEventType(), orders, member);
        }
        ```
        

**d.결제 단계별 상태 UPDATE**

- 장애는 언제든 어느 계층에서나 발생할 수 있습니다.
- 따라서 예외처리가 명확하게 진행하기 위해 각 단계가 완료될 때 마다 결제 엔티티의 상태를 변경하도록 구현했습니다.
- 순서도 
  <!-- -  [(click)주문상태변경 순서도](/assets/img/posts/2024/2024-05-01-payment-system/flow-chart.pdf) -->
    <embed src="/assets/img/posts/2024/2024-05-01-payment-system/flow-chart.pdf" type="application/pdf" width="100%" height="600px" />


**e.내부 테스트 환경**

- E2E 테스트 실행 시 외부 API를 실제로 호출할 경우 테스트가 외부 API에게 의존하는 문제가 발생합니다.
- 때문에 PG사(혹은 간편결제서버)의 외부API를 모킹하는 컨트롤러를 구현하고 테스트 실행 시에는 해당 컨트롤러를 호출하도록 구현하였습니다.
- 예시
    - 1)PG사(혹은 간편결제서버)의 외부API를 호출하는 클래스
        - profile 별로 다른 url이 주입되도록 구현

```java
@Service
public class TossPayService implements PayService {
		
    @Value("${tossPay.secretKey}")
    private static String secretKey;

    @Value("${tossPay.transactionUrl}") //테스트 시 http://localhost:8080/v1/payments/confirm 호출
    private String transactionUrl; 

    @Value("${tossPay.cancelTransactionUrl}") //테스트 시 http://localhost:8080/v1/payments/%s/cancel 호출
    private String cancelTransactionUrl;
    
   .
   .
   //생략
   .
   .
```

- 2)외부 API를 모킹하는 테스트 컨트롤러

```java
 

@RestController
@RequiredArgsConstructor
public class PayTestController {

    public final static String PAY_KEY = "payKeyExample";
    public final static String INVALID_PAY_KEY = "invalidPaymentKey";
    public final static String FAULURE_MSG = "잘못된 요청입니다.";

    private final OrderRepository orderRepository;

    @PostMapping("/v1/payments/confirm")
    public ResponseEntity<JSONObject> tossConfirm(@RequestBody JSONObject jsonObject) {
        String payKey = (String) jsonObject.get("paymentKey");

        JSONObject result = new JSONObject();

        if (payKey.equals(PAY_KEY)) {
            return ResponseEntity.ok(result);
        }

        JSONObject failure = new JSONObject();
        failure.put("message", FAULURE_MSG);
        result.put("failure", failure);
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(result);

    }

	 .
   .
   //생략
   .
   .
```

**f.Dead Letter Queue**

- 장애는 언제든 어느 계층에서나 발생할 수 있습니다.
    - 계층 중 특히 데이터베이스와 외부API는 장애가 즉시 해결되기 어렵거나 통제 밖에 있습니다.
- 이에 대응하여 특정 예외의 경우 Dead Letter Queue를 통해 비동기로 처리하도록 명시했습니다.
- 예시
    - 1)결제승인 성공 후 후처리가 실패할 경우  비동기 처리
    - 2)주문취소 성공 후 상품 재고 증감 처리가 실패할 경우 비동기 처리

### 6)쿠폰

**일급컬렉션에게 쿠폰 관련 역할 위임**

*개발자 향로 님의 블로그 글을 참고하여 작성했습니다 ([링크](https://jojoldu.tistory.com/412))

- 일급컬렉션을 이용하여 생성자에서 요청된 쿠폰 목록이 사용가능한지 검증하도록 구현했습니다.
- 아래 구현은 3가지 장점이 있습니다
    - 불변성을 보장한다
        - 객체 생성 시점에만 컬렉션을 추가할 수 있으므로 불변성이 보장됩니다.
    - 상태와 행위를 한 곳에서 관리
        - Coupons  객체는 할인쿠폰정책에만 종속적인 자료구조입니다.
        - 쿠폰 리스트라는 ‘**상태’**와 각종 검증로직이라는 ‘**행위**’를 한 객체에서 관리함으로써 높은 코드 응집도를 얻었습니다.
        - 덕분에 확장에 유리합니다. 예를 들어 특정 이벤트에 다른 쿠폰정책을 적용하고 싶을 경우 새로운 쿠폰 일급컬렉션을 구현 및 적용하면 되므로 기존 코드를 전혀 수정하지 않고도 확장이 가능합니다.

```java
public class Coupons {
    private final List<Coupon> couponList;

    //쿠폰은 최대 2개만 적용 가능
    private static final int MAX_COUPON_APPLICABLE_COUNT = 2;

    //중복적용 불가 쿠폰은 1개만 적용가능
    private static final int MAX_NOT_DUPLICATED_COUPON_COUNT = 1;

    //비율할인 쿠폰은 1개만 적용가능
    private static final int MAX_RATE_COUPON_COUNT = 1;

    public Coupons(List<Coupon> couponList) {
        validateAvailable(couponList); //유효기간이 남았는지 검증
        validateSize(couponList); //최대 사용가능 개수 이하인지 검증
        validateDuplicateAvailable(couponList); //최대 중복 적용 불가 쿠폰 개수 검증
        validateRateCouponCount(couponList); //최대 적용 가능 비율할인 쿠폰 개수 검증
        this.couponList = couponList;
    }
    
   .
   .
   //생략
   .
   .
```

---

<br>
## 6. 추후 보완할 점

- 테스트 전략에 대해 좀 더 공부하고 위 프로젝트를 회고해보니 1) unit 테스트의 비중을 늘리고 2)Service layer 의 테스트 커버리지를 높일 필요성을 느꼈습니다
    - unit 테스트의 경우 각각의 비즈니스 로직을 세밀하게 검증할 수 있는 장점이 있고  테스트의 실행속도가 빠르기 때문에 Integrated test와 e2e test보다 많은 커버리지를 담당해야 합니다
        - 또한 unit test TDD 의 필수요소이기도 합니다
    - Service layer가 핵심적인 비즈니스 로직을 관리하므로 Service layer의 테스트 커버리지를 늘리면 견고한 소프트웨어를 구현할 수 있습니다
        - 참고자료 : 테스트 피라미드 IMG
            
           ![pyramid](/assets/img/posts/2024/2024-05-01-payment-system/test-pyramid.png)

            
- 아직 DLQ 관련 구현 및 테스트는 완성하지 못했습니다. 근시일 내에 추가할 예정입니다.
- 기회가 닿는 데로 다른 개발자분들에게 피드백을 받고 개선할 예정입니다.

---

<br>
## 7. 참고한 자료

- 일급 컬렉션 (First Class Collection)의 소개와 써야할 이유 : [https://jojoldu.tistory.com/412](https://jojoldu.tistory.com/412)
- ‘가상 면접 사례로 배우는 대규모 시스템 설계 기초 2’  11장 결제 시스템
