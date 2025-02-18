---
title: Distributed Lock 과 Redis를 이용한 선착순 할인쿠폰 시스템
date: 2024-03-01 12:00:00 +0800
categories: [Backend, Architecture]
tags: [Backend, Architecture, Redis, MySQL, Distributed Lock, Redisson]     # TAG names should always be lowercase
---


> **선착순 시스템 개발 시리즈**
> 
> - 1부 : Distributed Lock 과 Redis를 이용한 선착순 할인쿠폰 시스템
> - [2부 : Redis의 Pub/Sub과 Sorted Set을 이용한 비동기 선착순 시스템](https://cloudhat.github.io/posts/redis-pub-sub-FCFS/)


## 1.개요

- Redisson을 이용한 Distributed Lock의 도입 근거
- **Redis에 재고를 선반영**하는 선착순 쿠폰 지급 로직 소개
- **Redis에 재고를 선반영**하는 로직의 한계 및 대안 제시

<br>
## 2.배경

- 회사에서 약 8개월에 걸쳐 여러 선착순 쿠폰 이벤트를 개발해왔습니다.
- 기존에는 선착순 쿠폰 발급 로직의 동시성 문제를 Redis의 List 자료구조를 Queue로 사용하여 해결해왔습니다 
- 그러나 Redis Queue 방식은 아래의 2가지 문제점을 가지고 있습니다
    - 이벤트 시작 전에 각 날짜의 재고에 해당하는 Queue를 미리 생성해야 함
      - 이벤트 기간이 길거나 재고가 많을 경우 Queue를 생성하는데 시간이 소요됨
      - 트랜잭션을 보장하기 힘들어 재고의 정합성이 깨지거나 
      - 휴먼에러가 발생할 가능성이 있음
- 위 문제를 해결하고자 Redisson을 이용한 Distributed Lock을 도입했습니다

<br>
## 3.Redisson을 이용한 Distributed Lock

### 1-1)선착순 이벤트 구현에 있어 Critical Section의 필요성

- 선착순 이벤트는 크게 아래 3가지 단계로 진행됩니다
    - 1. 발급 가능한 재고가 있는지 확인
    - 2. 발급 가능할 경우 재고 Record UPDATE  (’SET 재고 = 재고 - 발급수량’)
    - 3. RDB에 유저 별 쿠폰 Record INSERT
- 위 단계를 실행할 때 재고 데이터는 반드시 Critical Section으로 격리해야 합니다.
    - Critical Section으로 보호되지 않을 경우 동시성 문제가 발생하여 정해진 재고보다 추가 발급 되는 문제가 발생할 수 있습니다.
- Critical Section으로 재고 데이터를 격리하려면 process 혹은 thread가 동시에 접근할 수 없도록 **Lock을 구현해야 합니다.**


### 1-2)**MySQL을 이용한 Pessimistic lock 과 Optimistic lock의 단점**

- Lock 은  MySQL등의 RDB에서도 구현할 수 있습니다. 그러나 동시에 여러 process 혹은 thread에서 경합이 발생할 경우 RDB 레벨에서 구현은 몇 가지 단점을 가지고 있습니다.
- MySQL을 기준으로 설명 드리겠습니다.

**(1)MySQL 레벨 Pessimistic lock 의 단점**

- 다른 트랜잭션의 Lock을 고려하지 않고 구현할 경우 deadlock이 발생할 수 있습니다.
- commit이 완료되어 lock이 풀리기 전까지 다른 Transction이 대기해야 합니다.
    - 특히 Mysql에서 ‘SELECT … FOR UPDATE’로 구현한 경우 pessimistic lock에 필요하지 않은 다른 ROW도 LOCK이 걸릴 수 있습니다 ([참고](https://stackoverflow.com/questions/6066205/when-using-mysqls-for-update-locking-what-is-exactly-locked))
        - MySQL의 구조 상 인덱스가 없을 경우 테이블 전체에 lock을 걸고 인덱스가 있을 경우 인덱스에 lock을 걸기 때문입니다.
    - 또한 MySQL의 기본 격리수준인 REPEATABLE READ은 MVCC로 인해 트랜잭션이 길어질 경우 언두 영역에 데이터가 쌓여 데이터베이스에 부담이 될 수 있습니다

**(2)MySQL 레벨 Optimistic lock 의 단점**

- version이 다를 때마다 rollback이 일어나므로 선착순 이벤트처럼 경합이 많이 발생하는 로직에서는 부적절합니다.

### 2)대안 : Redisson을 이용한 Distributed Lock

- 위 문제점 때문에 MySQL 레벨의 Lock 대신 **Redisson**을 이용하여 Distributed Lock을 구현하는 방식을 선택했습니다.

**Redisson을 이용하여 Distributed Lock을 구현한 이유**

- Redisson은 In-memory DB인 redis를 사용하므로 RDB을 이용한 구현에 비해 데이터에 빠른 접근 가능
- Redisson의 Lock은 부분적으로 pub/sub을 사용하므로 redis I/O에 대한 부하를 줄일 수 있음
- RDB을 이용한 구현과 달리 Lock을 persistent layer와 분리할 수 있음




<br>
> **✅ 참고자료** 
> - Distributed Lock 이란?
>   - 다수의 서버에서 Critical Section 접근할 때 상호 배체를 보장하는 Lock을 의미합니다.
>     - 단일 DB를 사용할 경우 앞서 설명한 RDB레벨의 Lock도 Distributed Lock에 해당합니다.
> - Redisson 이란?
>   - 자바 및 lua script로 구현된 redis 분산락 클라이언트입니다
>   - Distributed Lock 을 구현하기 위해 redis 공식문서에서 제안하는 오픈소스 중 하나입니다.

<br>


{% raw %}
<details>
  <summary style="font-weight:600;font-size:1.5em;line-height:1.3;margin:0">
    3)RedissonLock의 로직 상세 (<a href="https://github.com/redisson/redisson/blob/master/redisson/src/main/java/org/redisson/RedissonLock.java">RedissonLock.java</a>)
  </summary>
  <div class="indented">
    <p>
      <mark class="highlight-red">*아래 설명은 Incheol 님의 글을 요약한 내용입니다 (</mark>
      <mark class="highlight-red">
        <a href="https://incheol-jung.gitbook.io/docs/q-and-a/spring/redisson-trylock#undefined-5">링크</a>
      </mark>
      <mark class="highlight-red">)</mark>
    </p>
    <b>1. Lock 획득 시도</b>
    <pre><code class="language-java">
          &lt;T&gt; RFuture&lt;T&gt; tryLockInnerAsync(long waitTime, long leaseTime, TimeUnit unit, long threadId, RedisStrictCommand&lt;T&gt; command) {
              return evalWriteSyncedAsync(getRawName(), LongCodec.INSTANCE, command,
                      "if ((redis.call('exists', KEYS[1]) == 0) " + 
                          "or (redis.call('hexists', KEYS[1], ARGV[2]) == 1)) then " + 
                          "redis.call('hincrby', KEYS[1], ARGV[2], 1); " + 
                          "redis.call('pexpire', KEYS[1], ARGV[1]); " + 
                          "return nil; " + 
                      "end; " +
                      "return redis.call('pttl', KEYS[1]);",
                      Collections.singletonList(getRawName()), unit.toMillis(leaseTime), getLockName(threadId));
        }
    </code></pre>
    <b>2. waitTime이 초과되었는지 확인</b>
    <pre><code class="language-java">
      time -= System.currentTimeMillis() - current;
      if (time <= 0) {
          acquireFailed(waitTime, unit, threadId);
          return false;
      }
    </code></pre>
    <b>3. 고유 Thread Id를 채널로 구독하여 lock이 available할 때까지 대기</b>
    <pre><code class="language-java">
        current = System.currentTimeMillis();
        CompletableFuture&lt;RedissonLockEntry&gt; subscribeFuture = subscribe(threadId);
        try {
            subscribeFuture.get(time, TimeUnit.MILLISECONDS);
        } catch (TimeoutException e) {
            acquireFailed(waitTime, unit, threadId);
            return false;
        }
    </code></pre>
    <b>4. waitTime 이전까지 무한 루프를 수행하면서 lock 점유 시간을 한번 더 확인</b>
    <pre><code class="language-java">
        while (true) {
          long currentTime = System.currentTimeMillis();
          ttl = tryAcquire(waitTime, leaseTime, unit, threadId);
          if (ttl == null) {
              return true;
          }
        }
    </code></pre>
  </div>
</details>
{% endraw %}

<br>

### 4)Spring AOP를 이용하여 Redisson 실제 적용

- 비즈니스 로직과 Lock 로직을 분리하기 위해 AOP를 사용했습니다

**(1) 어노테이션을 위한 클래스 선언**

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface DistributeLock {
    String key();

    TimeUnit timeUnit() default TimeUnit.SECONDS;

    long waitTime() default 5L;

    long leaseTime() default 3L;
}

```

**(2)AOP 구현**

```java
@Aspect
@Component
@RequiredArgsConstructor
@Slf4j
public class DistributeLockAop {
    private static final String REDISSON_KEY_PREFIX = "RLOCK_";

    private final RedissonClient redissonClient;

    @Around("@annotation(DistributeLock.java의_패키지경로.DistributeLock)")
    public Object lock(final ProceedingJoinPoint joinPoint) throws Throwable {
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();
        DistributeLock distributeLock = method.getAnnotation(DistributeLock.class);

        String key = KEY_PREFIX + Arrays.toString(signature.getParameterNames()) + Arrays.toString(joinPoint.getArgs()) + distributeLock.key();

        RLock rLock = redissonClient.getLock(key);

        try {
            boolean available = rLock.tryLock(distributeLock.waitTime(), distributeLock.leaseTime(), distributeLock.timeUnit());
            if (!available) {
                throw new RuntimeException();
            }
            return joinPoint.proceed();
        } catch (Exception e) {
            throw new RuntimeException();
        } finally {
            rLock.unlock();
        }
    }
}
```

**(3)AOP 적용 예시**

```java
@DistributeLock(key = "#test")
public void issueCoupon(Long couponId, Long memberId) throws OutOfStockException {

    if (isCouponIssueAble()) { //1.발급 가능한 재고가 있는지 확인
        couponRepository.decreaseStock(couponId);//2.2. 발급 가능할 경우 재고 Record UPDATE
        couponRepository.IssueCoupon(couponId, memberId) //3. RDB에 유저 별 쿠폰 Record INSERT
    }
    else{
        throw new OutOfStockException();
    }
}
```

## 4.Redis 재고 변화 선반영 선착순 쿠폰 시스템 구현

### 1)개요

- 위 AOP 적용 예시처럼 재고 확인, 재고 UPDATE, RDB에 쿠폰발급 3가지 단계를 atomic operation으로 묶을 수도 있습니다.
- 그러나 RDB의 INSERT 혹은 UPDATE는 상대적으로 느리므로 3가지 단계를 모두 atomic operation으로 처리하면 모든 응모 Request는 앞선 응모 Request가 순차적으로 처리 완료 될 때까지 기다려야 합니다.
- 위 문제는 재고 관련 처리만 atomic operation으로 묶어 redis에서 처리하고 RDB 관련 처리를 진행하여 해결할 수 있습니다
    - 1)Distributed Lock을 이용하여 Redis에서 재고조회 및 ’현재 발급된 쿠폰 수’의 증가를 atomic operation으로 묶어 처리하고 **(Redis 재고 변화 선반영)**
    - 2)1번 단계가 성공했을 때만 RDB에 INSERT만 하는 방식입니다.

### 2)상세로직

**a.재고가 있는 경우**

1)레디스관련 atomic operation(Distributed lock 적용)

- (1)레디스에서 ’현재 발급된 쿠폰 수’ 조회
- (2)발급가능 여부 판단
- (3)레디스의 ’현재 발급된 쿠폰 수’ 증가 (INCR)

2)RDB에 쿠폰 Record INSERT

3)응모 성공 Reponse 반환



![exhausted](/assets/img/posts/2024/2024-03-01-distributed-lock-redis-FCFS/flow-chart-stock-exists.png)

---

**b.재고가 소진된 경우**

1)레디스관련 atomic operation(Distributed lock 적용)

- (1)레디스에서 ’현재 발급된 쿠폰 수’ 조회
- (2)발급가능 여부 판단

2)재고가 없으므로 재고 소진 Response 반환



![exhausted](/assets/img/posts/2024/2024-03-01-distributed-lock-redis-FCFS/flow-chart-stock-exhausted.png)

---

### 3)코드 예시

```java
@Service
@RequiredArgsConstructor
public class FirstComeFirstServedService {
    
    private final DiscountCouponService discountCouponService;

    public boolean couponApply(Long couponId , Long memberId) {

        try{   //재고 관련 처리
            discountCouponService.redisAtomicOperation(couponId);
        }
        catch (OutOfStockException e){
            return false;
        }

        try {  //RDB에 쿠폰 Record INSERT
            discountCouponService.issueCoupon(couponId,memberId); 
        }
        catch (Exception e){ //INSERT 실패 시 DECR
            discountCouponService.couponQuantityDECR(couponId);
        }
    }
}
```

```java
@Service
@RequiredArgsConstructor
public class DiscountCouponService {
    private final CouponRepository couponRepository;
    private final RedisRepository redisRepository;

    @DistributeLock(key = "#FCFS")
    public void redisAtomicOperation(Long couponId) throws OutOfStockException {
        
        int allowedCouponQuantity = couponRepository.findById(couponId).getCouponQuantity();
        //레디스에서 ’현재 발급된 쿠폰 수’ 조회
        int issuedCouponQuantity = redisRepository.getIssuedCouponQuantity(couponId);

				//발급가능 여부 판단
        if (issuedCouponQuantity >= allowedCouponQuantity) {
            throw new OutOfStockException();
        }
        else{  //레디스의 ’현재 발급된 쿠폰 수’ 증가 (INCR)
            redisRepository.couponQuantityINCR(couponId);
        }
    }
		//RDB에 쿠폰 Record INSERT
    public void issueCoupon(Long couponId , Long memberId){
        couponRepository.issueCoupon(couponId, memberId);
    }

    public void couponQuantityDECR(Long couponId){
        redisRepository.couponQuantityDECR(couponId);
    }
}
```

### 4)장점

- 재고관련 처리를 RDB를 거치지 않고 Redis만 사용하므로 빠른 처리가 가능합니다.
    - 특히 재고가 소진된 시점에 발생한 Request는 아예 RDB를 거치지 않으므로 유저가 빠른 응답을 받을 수 있습니다.
- 로직의 순서 상 절대 초과발급이 발생하지 않습니다.

### 5)문제점

- WAS 혹은 RDB에서 장애가 발생할 경우 Redis의  ’현재 발급된 쿠폰 수’보다 실제 발급된 쿠폰의 수가 적을 수 있습니다.
    - RDB에서만 장애가 발생했을 경우 아래 예시처럼 Redis ’현재 발급된 쿠폰 수’를 차감하여 해결할 수 있습니다.
    - 그러나 서버에서도 장애가 발생한 경우 차감시도가 실패할 수 있습니다
- 이는 레디스와 RDB의 작업을 단일 트랜잭션으로 묶을 수 없기 때문에 원자성을 지킬 수 없는 것이 근본적인 원인입니다.



![failure](/assets/img/posts/2024/2024-03-01-distributed-lock-redis-FCFS/flow-chart-rdb-failure.png)

---

### 6)대안

- 발급을 비동기로 처리하여 응모와 발급을 분리하면 굳이 Redis에 재고 변화 선반영하지 않아도 안정적으로 트래픽을 처리할 수 있습니다.
    - 우아한 형제들의 예시 ([링크](https://www.youtube.com/watch?v=MTSn93rNPPE))
- 혹은 Kafka를 활용하여 RDB INSERT을 안정적으로 진행하는 방법도 있습니다.
    - 여기어때 예시 ([링크](https://techblog.gccompany.co.kr/redis-kafka%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%9C-%EC%84%A0%EC%B0%A9%EC%88%9C-%EC%BF%A0%ED%8F%B0-%EC%9D%B4%EB%B2%A4%ED%8A%B8-%EA%B0%9C%EB%B0%9C%EA%B8%B0-feat-%EB%84%A4%EA%B3%A0%EC%99%95-ec6682e39731))

## 5.마치며

- 불과 1년 전만 해도 저는 ‘동시성 문제’라는 키워드 조차 모르는 부족한 개발자였습니다.
- 그러나 선착순 쿠폰 이벤트 API를 지속적으로 리팩토링하면서 아래와 같은 많은 경험을 얻었습니다.
    - 복잡한 이벤트 로직을 테스트 코드를 통해 견고하게 구현
    - 동시성 문제에 대한 이해 및 해결
    - 인덱스, Lock 등 MySQL의 구체적인 작동 원리
    - 트래픽으로 인한 장애 대응
- 계속 바뀌는 요구사항, 널뛰는 트래픽에 대응하기 위해 어떻게 견고한 소프트웨어를 구현할지 매일 고민하고 있습니다. 그러나 이런 고민을 해결하는 과정이 전혀 괴롭지 않고 오히려 즐겁습니다.  즐거움을 원동력 삼아 앞으로도 열심히 정진하고자 합니다.

## 6.참고한 자료

- [Redisson Github](https://github.com/redisson/redisson/wiki/8.-Distributed-locks-and-synchronizers) 

- 마켓컬리 : [풀필먼트 입고 서비스팀에서 분산락을 사용하는 방법 - Spring Redisson](https://helloworld.kurly.com/blog/distributed-redisson-lock/)

- [redisson trylock 내부로직](https://incheol-jung.gitbook.io/docs/q-and-a/spring/redisson-trylock#undefined-5)

