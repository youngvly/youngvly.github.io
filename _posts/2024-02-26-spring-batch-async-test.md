---
title: "Spring Batch Async 실행"
date: 2024-02-26 20:00:00 +0400
categories: spring, batch
tags: spring, batch
---

# spring batch async 실행테스트

### Reader
```java
@Slf4j
@Component
public class AItemReader implements ItemReader<Integer> {
	AtomicInteger value = new AtomicInteger(0);

	@Override
	public Integer read() throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException {
		int it = value.addAndGet(1);
		log.error("reader : {}", it);
		return it < 1000 ? it : null;
	}
}
```

### Processor
```java
@Slf4j
@Component
public class AItemProcessor implements ItemProcessor<Integer, Integer> {
	@Override
	public Integer process(Integer item) throws Exception {
		log.error("processor : {} -> {}", item, item * 100);
		return item * 100;
	}
}
```

### Writer
```java
@Slf4j
@Component
public class AItemWriter implements ItemWriter<Integer> {
	@Override
	public void write(List<? extends Integer> items) throws Exception {
		log.error("aggregator  size : {}", items.size());
		items.forEach(
			item -> log.error("aggregator : items : {}", item);
		);
	}
}
```

### Config
```java
	@Bean
	public Job job(JobBuilderFactory jobBuilderFactory, Step step) throws Exception {
		return jobBuilderFactory
			.get("job")
			.incrementer(new RunIdIncrementer())
			.flow(step)
			.end()
			.build();
	}

	@Bean
	public Step addQnaMigrationStep(StepBuilderFactory stepBuilderFactory, AsyncItemProcessor<Integer, Integer> asyncProcessor,
		AsyncItemWriter<Integer> asyncItemWriter) throws Exception {
		return stepBuilderFactory
			.get("step")
			.<Integer, Future<Integer>>chunk(20)
			.reader(reader)
			.processor(asyncProcessor)
			.writer(asyncItemWriter)
			.build();
	}

	@Bean
	public AsyncItemProcessor<Integer, Integer> asyncProcessor(TaskExecutor taskExecutor) {
		AsyncItemProcessor<Integer, Integer> asyncItemProcessor = new AsyncItemProcessor<>();
		asyncItemProcessor.setDelegate(processor);
		asyncItemProcessor.setTaskExecutor(taskExecutor);

		return asyncItemProcessor;
	}

	@Bean
	public AsyncItemWriter<Integer> asyncItemWriter() {
		AsyncItemWriter<Integer> writer = new AsyncItemWriter<>();
		writer.setDelegate(aggregator);

		return writer;
	}

	@Bean
	public TaskExecutor taskExecutor() {
		ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
		executor.setCorePoolSize(25);        // 기본 스레드풀 사이즈
		executor.setMaxPoolSize(25);        // 최대
		executor.setQueueCapacity(25);        // ?
		executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
		executor.setThreadNamePrefix("Thread-");
		return executor;
	}
```

### 실행 결과
chunk 단위가 reader - processor - writer 을 실행하는 주기
- 다음 chunk단위는 순차적으로 호출된다. 
- ex) chunk가 20이면
  - reader 20회 호출 (request thread)
  - processor 20회 호출 (병렬 thread)
  - writer 1회 호출 (request thread)
  - (next) reader 20회 호출 (request thread)
  - processor 20회 호출 (병렬 thread)
- reader에서 null 반환시 멈춤.
```sh
[http-nio-0000-exec-1] reader : 981 (AItemReader.java:30)
[http-nio-0000-exec-1] reader : 982 (AItemReader.java:30)
[http-nio-0000-exec-1] reader : 983 (AItemReader.java:30)
...
[Thread-17] processor : 981 -> 98100 (AItemPrcessor.java:22)
[Thread-19] processor : 982 -> 98200 (AItemPrcessor.java:22)
[Thread-19] processor : 984 -> 98400 (AItemPrcessor.java:22)
...
[http-nio-0000-exec-1] aggregator  : 19 (AItemWriter.java:24)
[http-nio-0000-exec-1] aggregator : items : 98100 (AItemWriter.java:27)
[http-nio-0000-exec-1] aggregator : items : 98200 (AItemWriter.java:27)
```

### 이슈 mysql Deadlock
- 에러로그
```sh
Caused by: com.mysql.cj.jdbc.exceptions.MySQLTransactionRollbackException: Deadlock found when trying to get lock; try restarting transaction
```
- DB 상태 확인
```mysql
SHOW ENGINE INNODB STATUS;

------------------------
LATEST DETECTED DEADLOCK
------------------------
*** (1) TRANSACTION:
mysql tables in use 1, locked 1
LOCK WAIT 3 lock struct(s)
INSERT INTO database_name.table_name()...
*** (1) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 10000 page no 33 n bits 1000 index auto_increment_column_name of table_name lock_mode X insert intention waiting
*** (2) TRANSACTION:
mysql tables in use 1, locked 1
4 lock struct(s),
INSERT INTO database_name.table_name()...
*** (2) HOLDS THE LOCK(S):
RECORD LOCKS space id 10000 page no 33 n bits 1000 index auto_increment_column_name of table_name lock_mode X
*** (2) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 10000 page no 33 n bits 1000 index auto_increment_column_name of table_name lock_mode X insert intention waiting
*** WE ROLL BACK TRANSACTION (1)
```

동일 db.table에 insert 쿼리가 병렬스레드에서 동시에 동작하는 상황이고,
테이블에는 auto increment key가 있어 인덱스락을 실행한다.
mysql + innoDB 조합은 next-key locking 전략을 사용한다. (아래 참고) 마지막 키와 새로운 키를 Lock 한다.
동시에 insert 요청이 들어오면, deadlock으로 탐지한다.

### next-key locking
- Index Record Locks: When inserting a new row with an auto-increment key, InnoDB locks the index records it needs to access to ensure data consistency and prevent concurrent modifications. This typically involves locking the index record corresponding to the newly generated auto-increment value.

- Gap Locks: InnoDB may also use gap locks to prevent other transactions from inserting rows that would fall within the range of auto-increment values reserved by the current transaction. This ensures that each auto-increment value is unique.

- Next-Key Locks: InnoDB combines index record locks and gap locks into next-key locks. Next-key locks lock the index record itself and the gap before the record, preventing phantom reads and ensuring consistent query results.


