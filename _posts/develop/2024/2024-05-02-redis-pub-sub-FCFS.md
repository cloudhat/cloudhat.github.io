---
title: (POC)Redis의 Pub/Sub과 Sorted Set을 이용한 비동기 선착순 시스템
date: 2024-05-02 12:00:00 +0800
categories: [Backend, Architecture]
tags: [Backend, Architecture, Redis, Pub/Sub, Sorted Set, Lua Script, Async]    
---


---

<br>
## 1.개요

- Redis의 Pub/Sub과 Sorted Set을 이용하여 응모와 발급을 비동기로 분리한다
- Sorted Set 자료구조의 특성을 이용하여 발급순서를 보장한다
    - lua script로 동시성 문제를 해결한다

---

<br>
## 2.배경

- 이전에 소개한 Distributed Lock과 Redis를 이용한 선착순 쿠폰 시스템도 문제 없이 작동했습니다 ([🔗](https://cloudhat.github.io/posts/distributed-lock-redis-FCFS/))
- 그러나 위 구현은 일정 이상의 트래픽이 몰릴 경우 응모 Request가 Lock 획득에 실패하여 실패처리 되는 문제가 발생할 수 있습니다.
    - 응모 실패 시 사용자는 다시 응모시도를 해야 하므로 이는 나쁜 고객경험으로 이어집니다.
- 위 문제 발생을 예방하기 위해 선제적으로 응모와 발급이 비동기로 분리된 선착순 시스템을 POC 형태로 제시했습니다
    
  
---

<br>
## 3.구현 상세

### 1)Pub/Sub 활용

- Pub/Sub을 이용하여 응모 요청 발생 시 Worker가 발급을 진행하도록 구현하였습니다.
- Worker는 Sorted Set에서 최신순으로 응모정보를 획득하여 쿠폰 발급을 진행합니다

### 2)Sorted set 및 lua script 활용

- Sorted set을 queue로 사용하였습니다
    - **비교적 빠른 시간복잡도로 (*logN) 특정 회원의 ‘대기순번’을 얻을 수 있기 때문에 Sorted set을 선택했습니다**
    - Sorted set의 ‘score’에 Timestamp를 넣어 응모순으로 발급이 진행되도록 구현하였습니다
        - ex) `ZADD 이벤트ID TimeStamp 회원ID`
- queue의 POP에 해당하는 메소드는 동시성 문제를 해결하기 위해 lua script를 활용했습니다
    - **ZPOPMIN** 이라는 명령어도 있지만 기존 레거시 시스템의 RedisTemplate 버전이 해당 명령어를 지원하지 않아 대안으로 lua script를 사용했습니다.
    - (1)Sorted Set에 Value가 있는지 확인하고 (2)Value가 있다면 그 중 가장 Score가 낮은 Value를 ZREM으로 삭제하고 (3)해당 Value를 리턴하는 과정을 Atomic하게 처리해야 하기 때문입니다.
        
         *Redis는 싱글쓰레드로 작동하므로 lua script로 atomic한 연산이 보장됩니다
        
---
- 💡 lua script를 이용한 POP 구현 예시
    
      
    ```java
    @Repository
    @RequiredArgsConstructor
    public class RedisQueue {
      private final RedisTemplate redisTemplate;
    
      private final String ZSET_PREFIX = "ZSET_PREFIX:";
    
      private DefaultRedisScript<List> popScript;
    
      public RedisQueueElement POP(String eventTypeCd) {
    
          String key = getKey(eventTypeCd);
    
          List<Object> result = (List<Object>) redisTemplate.execute(popScript, Collections.singletonList(key));
    
          if (result.get(0) == null) {
              return null;
          }
    
          String value = (String) result.get(0);
          Long score = (Long) result.get(1);
          return new RedisQueueElement(value, score);
    
      }
    
      @PostConstruct
      public void init() {
    
          String script =
        "local key = KEYS[1]\n" +
        "local result = redis.call('ZRANGE', key, 0, 0, 'WITHSCORES')\n" + //ZRANGE 명령어로 value,score 조회
        "if #result > 0 then\n" + //만약 조회 결과가 존재한다면
          "redis.call('ZREM', key, result[1])\n" + //ZREM 명령어로 Sorted set에서 value 삭제
          "return {result[1], result[2]}\n" + //value와 score 리턴
        "else\n" + //만약 조회 결과가 없다면
        "    return nil\n" + //null 리턴
        "end";
    
          // RedisScript 객체 설정
          popScript = new DefaultRedisScript<>();
          popScript.setScriptText(script);
          popScript.setResultType(List.class);
      }
    }
    ```
        

    

### 3)전체 프로세스 요약

(1) 사용자가 응모 요청

(2) 요청을 queue(Sorted set)에 저장

(3) Pub/Sub으로 worker에게 message send

(4) message를 받은 worker는 발급 진행

(5) 사용자는 주기적으로 발급결과 확인 요청

- 발급 완료 시 요청 종료

---
💡 전체 프로세스 상세
    
<embed src="/assets/img/posts/2024/2024-05-02/sequence diagram.pdf" type="application/pdf" width="100%" height="600px" />


---

<br>
## 4.왜 Redis의 Pub/Sub과 Sorted set을 선택했는가?

- 내부 사정 상 사용 가능한 미들웨어가 Redis 밖에 없기 때문에 선택했습니다.

---

<br>
## 5.장점

- 처리 가능한 만큼만 비동기로 발급을 진행하므로 장애 없이 대량의 request 처리 가능
- 응모 내역이 별개의 queue에 저장되므로 유실 없이 발급 처리 가능

---

<br>
## 6.단점

- Redis Pub/Sub은 별도의 수신확인 기능이 없고 메시지를 저장하지 않습니다
    - 이 때문에 서버 상황에 따라 Worker가 작동하지 않을 수도 있습니다
    - 이 문제는 주기적으로 Worker에게 작업시작을 요청하는 것으로 보완했습니다
- Redis Sorted set의 명령어의 시간복잡도는 대부분 log(N)입니다. 이는 발급대기열이 지나치게 많이 쌓일 경우 작업시간이 길어지고 Redis에 부하로 작용할 수 있음을 의미합니다.
    - Sorted set으로 감당 불가능한 트래픽이 발생할 것이 예상될 경우 대안으로 Kafka 등의 다른 미들웨어를 사용할 수 있습니다

---

<br>
## 7.마치며

- 서비스가 성장함에 따라 자연스럽게 새로운 아키텍쳐를 도입하면서 아래의 경험을 쌓을 수 있어 즐거웠습니다.
    - 비동기를 이용하여 느슨한 결합의 아키텍쳐 구현
    - 복잡한 이벤트 로직을 테스트 코드를 통해 견고하게 구현
    - 동시성 문제에 대한 이해 및 해결
    - 인덱스, Lock 등 MySQL의 구체적인 작동 원리
    - 트래픽으로 인한 장애 대응
- 시니어 개발자의 도움을 받을 수 없는 상황에서 혼자 구현한 것이라 피드백을 받을 수 없어 아쉬웠습니다. 하지만 오히려 성장에 대한 욕구가 더 불타오르는 계기가 되었습니다
