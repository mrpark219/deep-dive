# Spring Batch

## 1. 배치 애플리케이션

- **배치(Batch)**는 **일괄 처리 작업**을 의미한다.
- 일반적으로 **정해진 시간에 대량의 데이터를 처리**하는 데 사용된다.
- 예시:
  - 하루 동안 발생한 거래 정보를 기반으로 **통계 데이터를 생성**한다.
  - 특정 조건을 만족하는 사용자에게 **일괄적으로 쿠폰을 지급**한다.
  - 외부 API로부터 **대량의 데이터를 주기적으로 동기화**한다.
- 이러한 정형화된 작업을 자동화하고, 효율적으로 실행할 수 있도록 도와주는 것이 배치 애플리케이션이다.

## 2. Spring Batch

- **Spring 진영에서 배치 애플리케이션을 효과적으로 구성하고 실행하기 위해 제공하는 프레임워크**이다.
- 스프링의 핵심 개념(IoC, AOP, DI 등)과 자연스럽게 통합된다.
- 대량 데이터 처리, 로그 관리, 트랜잭션 관리, 작업 재시작, 병렬 처리 등 **배치 처리에 필요한 기능을 기본 제공**한다.
- **Job → Step → Reader, Processor, Writer** 또는 **Tasklet** 구조로 실행 흐름을 정의한다.

## 2.1 Job

- Job은 **배치 애플리케이션에서 실행의 시작점이 되는 단위**이다.
- 하나의 Job은 여러 개의 Step을 포함할 수 있으며, 각 Step은 순차적으로 또는 조건에 따라 실행된다.
- Job마다 고유한 이름이 부여되며, 실행 이력(JobInstance, JobExecution)을 관리할 수 있다.

## 2.2 Step

- Step은 **실제 작업을 수행하는 단위**이다.
- 하나의 Step은 **Reader → Processor → Writer** 구조 또는 **Tasklet** 기반의 단일 작업으로 구성된다.
- 각 Step은 독립적으로 트랜잭션이 관리되며, 실패 시 재시도 및 재시작 등의 정책을 적용할 수 있다.

## 2.3 Reader, Processor, Writer

- **Reader**: 입력 데이터를 읽어오는 역할을 한다 (예: DB, 파일, API 등).
- **Processor**: 읽어온 데이터를 가공하거나 필터링하는 단계이다 (선택적).
- **Writer**: 최종적으로 처리된 데이터를 저장하는 단계이다 (예: DB 저장, 파일 쓰기 등).
- 이 구조는 대량 데이터를 처리할 때 **chunk 기반 처리 방식**과 함께 사용된다.

## 2.4 Tasklet

- Tasklet은 **단일 작업을 정의하는 방식**이다.
- 반복 없이 한번에 완료되는 로직에 적합하다 (예: 테이블 초기화, 로그 정리 등).
- **execute 메서드** 안에 로직을 작성하며, 직접 제어가 필요한 작업에 유용하다.

## 3. 메타 테이블

[메타 테이블](./meta-tables.md)

## 4. JobParameter

[JobParameter](./job-parameter.md)

## 5. Chunk

[Chunk](./chunk.md)

## 6. ItemReader

[ItemReader](./itemReader.md)

## 7. ItemWriter

[ItemWriter](./itemWriter.md)

## 8. ItemProcessor

[ItemProcessor](./itemProcessor.md)

## 9. 병렬 처리

[병렬 처리](./scaling-parallel-processing.md)

## 10. 반복

[반복](./repeat.md)
