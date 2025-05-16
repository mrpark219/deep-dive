# 스케일링과 병렬 처리

## 1. 스케일링과 병렬 처리

- 대부분의 배치 처리는 **단일 스레드/프로세스 기반**으로도 충분히 수행 가능하다.
- 그러나 **대용량 데이터를 빠르게 처리해야 하는 상황**이나 **API 응답 지연으로 인한 처리 속도 저하** 등의 이슈로 인해 병렬 처리를 도입해야 할 때가 있다.
- Spring Batch는 **단일 프로세스 내 멀티 스레드 처리**와 **다중 프로세스 처리 방식**을 모두 지원한다.

### 병렬 처리 방법

1. **Multi-thread Step** (단일 프로세스)
2. **Parallel Steps** (단일 프로세스)
3. **Remote Chunking** (다중 프로세스)
4. **Step Partitioning** (단일 또는 다중 프로세스)

## 2. Multi-thread Step

- 하나의 Step 내부에서 여러 스레드가 동시에 Chunk 단위를 처리하는 방식이다.
- `TaskExecutor`를 설정하여 Step의 Reader, Processor, Writer가 **병렬 스레드에서 동작**하도록 만든다.
- 가장 간단하게 병렬 처리를 적용할 수 있는 방법이다.

### 2.1 장점

- 설정이 간단하다.
- 별도의 인프라 구성이나 배포 없이 기존 Job에 병렬 처리를 도입할 수 있다.

### 2.2 단점

- **동시성 문제가 발생할 수 있다.**
  - 예: 상태를 가지는 `ItemReader`를 여러 스레드가 공유할 경우 충돌 발생 가능성 있음.
  - 해결 방법: `SynchronizedItemStreamReader`와 같은 **Thread-safe한 Reader** 사용 필요.
- 처리 순서가 보장되지 않는다.
- **데이터베이스 커넥션 풀**은 스레드 수 이상으로 설정해야 한다.

### 2.3 예제 코드

```java
@Bean
public TaskExecutor taskExecutor() {
    return new SimpleAsyncTaskExecutor("spring_batch");
}

@Bean
public Step sampleStep(TaskExecutor taskExecutor, JobRepository jobRepository, PlatformTransactionManager transactionManager) {
	return new StepBuilder("sampleStep", jobRepository)
				.<String, String>chunk(10, transactionManager)
				.reader(itemReader()) // Thread-safe wrapper 적용 필요
				.writer(itemWriter())
				.taskExecutor(taskExecutor)
				.build();
}
```
