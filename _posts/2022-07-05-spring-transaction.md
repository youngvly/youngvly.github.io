---
layout: post
title: "Spring transaction"
date: 2022-07-05 16:50
categories: SpringBoot summary
tags: spring
---
# Transaction
- transaction : 최소 행동 단위
- exception 발생시 commit하지않고 rollback을 하기 위함.

## Spring Transaction
1. 트랜잭션 동기화
  - 트랜잭션을 위한 connection을 스레드마다 할당받아 멀티스레드에서 사용할 수 있다.
2. db에 종속되지않게 트랜잭션이 추상화되어있다.
   - PlatformTransactionManager
   - AOP 사용으로 비즈니스로직과 분리되어있다.

## @Transactional
1. 트랜잭션 전파 Propagation
   - A 트랜잭션 내부에서 다른 B 트랜잭션이 생성될때 어떻게 처리할것인지
   - PROPAGATION_REQUIRED (default)
     - 트랜잭션 참여, A와 B 가 같은 트랜잭션을 사용.
     - B <-> A 결과 전파가 가능하다.
   - PROPAGATION_SUPPORTS
     - 기존 트랜잭션(A)과 커넥션,세션 등을 공유하고, 기존 트랜잭션(A)이 없으면 트랜잭션이 없는것과 같다.
     - sync 문제 해결용으로 사용가능할듯.
   - PROPAGATION_MANDATORY
     - 기존 트랜잭션이 있으면 참여하고, 기존 트랜잭션(A)가 없으면 exception을 발생시킨다
     - 단독으로 실행되면 안되는 로직이 있을때 사용가능.
   - PROPAGATION_REQUIRES_NEW
     - 트랜잭션 분리, B트랜잭션이 새로 생성되고, A의 트랜잭션과 무관하다.
   - PROPAGATION_NOT_SUPPORTED
     - 기존 트랜잭션(A)이 있으면 중단시킨다.
     - 어떤 상황에서필요할까
   - PROPAGATION_NEVER
     - 기존 트랜잭션(A)가 있으면 Exception을 발생시킨다.
   - PROPAGATION_NESTED
     - required와 같지만, B의 성공실패가 A에 영향을 주지않는다. A->B로의 롤백전파는 가능.
2. 격리수준
    - DEFAULT
      - datasource의 기본 설정에 따른다.
      - mysql은 REPEATABLE_READ
    - READ_UNCOMMITTED (dirty read)
      - 다른 트랜잭션에서 커밋되지않은 변경사항까지 읽어온다. 만약 다른 트랜잭션에서 롤백되었다면, invalid한 데이터를 읽게된다.
      - 가장 빠르기 때문에, 일관성대신 속도를 택할경우 의도적으로 사용될 수 있다.
    - READ_COMMITTED
      - 실제 커밋된 변경사항만 읽어온다.
      - 다른 트랜잭션에서 수정하는것을 허용하기때문에, 다시 읽을때 변경된 값을 읽어올 수 있다.
    - REPEATABLE_READ
      - 실제 커밋된 변경사항만 읽어오고, 다시읽어오는게 가능하다.
      - A 트랜잭션에서 읽고, B에서 수정후 A에서 다시 읽을때 변경된 값을 읽어올 수 없다. ()
      - A 트랜잭션에서 읽고, B에서 insert 후 다시 읽을때 insert된 row가(phantom row) 읽힌다.
      - 수정은 막아주지만, insert는 허용된다.
    - SERIALIZABLE
      - REPEATABLE_READ와 동일한데, phantom read가 일어나지 않는다.
      - 읽고-> insert 후 -> 다시읽을때, insert된 row가(phantom) 읽히지 않는다.
      - 여러 트랜잭션이 같은 테이블에 접근할수없어 성능이 떨어질수있고, 가장 강력한 격리수준이다.
3. 제한시간
    - 설정하지 않으면, 기본 타임아웃이 설정된다. 
    - org.springframework.transaction.TransactionDefinition#TIMEOUT_DEFAULT = -1 (제한 없음?)
5. 읽기전용
   - 성능 최적화, 의도되지않은 부작용(상태변경) 방지

## JPA에서의 Transaction
```java
no entitymanager with actual transaction available for current thread
```

jpa는 transaction을 기반으로 동작하므로, transaction 이 있어야 영속성 컨텍스트를 사용할 수 있다.
- save 의 동작 [LINK](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.entity-persistence.saving-entites)

### delete 와 Transaction (write-behind)
- delete는 영속성 컨텍스트의 쓰기지연 저장소에 저장되었다가, 트랜잭션의 커밋시점에 실행된다.
- `SimpleJPARepository` 의 delete 에는 @Transactional(propagation = REQUIRED) 가 걸려있고, 상위 트랜잭션과 공유된다.
- 따라서 delete 는 호출부가 아닌 transaction이 제일 먼저 걸린 (application layer 일 수 있음) 에서 호출되어 exception 발생위치도 delete 메서드 호출시점에서는 발생되지 않는다.
- 테스트할때도 마찬가지이며, 테스트코드에 transactional이 걸리면 메서드 종료시점에서 수행될 수 있다.
- jpa에서 delete는 select -> (update) -> remove 의 순서로 진행된다. (save는 왜 없어도 되는걸까?)
- transaction이 없으면 entitymanager를 얻을수 없으므로, exception이 발생한다.
- [참고](https://velog.io/@giantim/5)

### write-behind
- 영속성컨텍스트가 변경사항을 모아두었다가, 커밋시점 (flush)시에 한번에 실행하는데,
- insert -> 변경없음 -> insert -> flush = 1번 insert 발생
- insert -> 변경 -> insert -> flush = 1번 insert, 1번 update
- [참고](https://soongjamm.tistory.com/150)

## Lock
```java
@Lock(LockMode.**)
```
- 낙관적 lock / 비관적 lock
  
|/| 낙관적 | 비관적 |
|---|---|---|
|방식| 버전 방식, 업데이트대상 row 버전이 낮을경우 업데이트하지 않음.| 트랜잭션으로 shared lock을 잡음 |
|롤백| 트랜잭션으로 커밋,롤백 하지않으므로 수동 롤백 필요.| 트랜잭션 롤백 가능|
|컨트롤|application|db|

- LockMode
  - READ == OPTIMISTIC
    - 되도록이면 명시적으로 Optimistic..을 사용하자.
  - WRITE == OPTIMISTIC_FORCE_INCREMENT
  - OPTIMISTIC (낙관적)
  - OPTIMISTIC_FORCE_INCREMENT
    - 버전정보를 활용하는 낙관적 락
    - JPA `@Version` 컬럼 필요
  - PESSIMISTIC_READ (비관적)
    - 트랜잭션으로 읽기까지 금지하는 락
  - PESSIMISTIC_WRITE
    - 트랜잭션으로 쓰기만 금지하는 락
  - PESSIMISTIC_FORCE_INCREMENT
    - 트랜잭션으로 쓰기를 금지시키고, 버전정보를 사용.
  - NONE : no lock


### 문제상황 : 1개의 column(PK) 와 1개의 row를 가지는 데이터의 lock
1. delete -> save -> select
   - PK가 곧 유일한 데이터이기때문에, delete 를 할경우 다른 transaction에서 lock을 잡을 row가 없는것으로 판단된다. > 의도대로 lock이 걸리지 않음
2. select -> update
    - select가 먼저 일어날 경우 read라 그런지(?) 먼저 lock을 잡지 않는다. -> 의도대로 lock이 걸리지 않음.
3. update -> select
    - 의도대로 동작.
    - 참고 ) mysql에서 CUD가 일어날경우 lock을 잡는다.
```java
    @Override
	@Retryable(maxAttempts = 2, backoff = @Backoff(delay = 1000L))
	@Lock(LockModeType.PESSIMISTIC_READ)
	@Transactional(isolation = Isolation.READ_COMMITTED, propagation = Propagation.REQUIRED)
	public int getSequence() {
		jpaRepository.updateSequence();     // custom query
		return jpaRepository.findFirst()
			.orElseGet(() -> jpaRepository.save(SomeEntity.init()))
			.getSequence();
	}
```

동작 확인 

```java
	@Test
	@DisplayName("여러 스레드가 동시에 업데이트 요청시 중복이 없어야한다.")
	void multiThread_lock() throws Exception {
		Set<Integer> sequences = new HashSet<>();
		int numberOfThreads = 20;
		ExecutorService service = Executors.newFixedThreadPool(10);
		CountDownLatch latch = new CountDownLatch(numberOfThreads);

		for (int i = 0; i < numberOfThreads; i++) {
			service.execute(() -> {
				sequences.add(service.getSequence());
				latch.countDown();
			});
		}
		latch.await();

		assertThat(sequences).as("중복이 발생하지 않아야한다.").hasSize(numberOfThreads);
	}
```

### 문제상황 : Deadlock found when trying to get lock; try restarting transaction
어플리케이션 로그에 db lock 관련 deadlock 로그가 찍혔고,
```sh
$ mysql> show engine innodb status;
------------------------
LATEST DETECTED DEADLOCK
------------------------
*** (1) TRANSACTION:...
*** (1) HOLDS THE LOCK(S): ... 
*** (1) WAITING FOR THIS LOCK TO BE GRANTED: ...
*** (2) TRANSACTION: ...
*** (2) HOLDS THE LOCK(S): ...
*** (2) WAITING FOR THIS LOCK TO BE GRANTED: ...
*** WE ROLL BACK TRANSACTION (1)
```

db로그를 확인하니 1과 2의 쿼리가 동일했다.
최근 변경사항중 클래스단에 `@Transactional` 어노테이션을 추가했는데 여기서 중복으로 트랜잭션이 발생한것같다.
```java
@Transactional
public class SomeCommandService{
    @Transactional
    public void updateSomething(){...}
}
```
둘중하나 어노테이션 떼어주니 정상동작.

----
참고
- https://mangkyu.tistory.com/154
- https://mangkyu.tistory.com/169
- https://velog.io/@giantim/5
- https://sas-study.tistory.com/348
- [비관적/낙관적 락](https://sabarada.tistory.com/175)