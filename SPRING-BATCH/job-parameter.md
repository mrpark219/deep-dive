# JobParameter

## 1. JobParameter

- Spring Batch에서는 외부나 내부에서 전달받은 파라미터를 `JobParameter`로 정의하고, 이를 다양한 배치 컴포넌트에서 사용할 수 있도록 지원한다.
- `JobParameter`를 사용하려면 Spring Batch 전용 Scope인 `@StepScope` 또는 `@JobScope`를 선언해야 한다.
- 이 어노테이션을 사용하면 해당 컴포넌트는 Spring 컨테이너에 의해 **지정된 실행 시점에 Bean으로 생성**되며, 생성 시점을 늦출 수 있다.

## 2. Bean 생성을 지연시킴으로써 얻는 이점

- **JobParameter의 지연 바인딩(Late Binding)**이 가능하다.
  - 애플리케이션 시작 시점이 아닌 실제 Job 또는 Step 실행 시점에 파라미터를 주입받을 수 있다.
  - Controller나 Service 등에서 동적으로 생성된 파라미터를 Job에 유연하게 전달할 수 있다.
- **동일한 컴포넌트의 병렬 실행 시에도 안정성이 보장된다.**
  - `@StepScope` 없이 동일 Tasklet을 병렬로 실행하면 서로 다른 Step이 같은 Bean 인스턴스를 공유해 상태 충돌이 발생할 수 있다.
  - `@StepScope`를 선언하면 Step마다 **독립적인 Bean 인스턴스가 생성**되므로 상태 간섭이 없다.

## 3. @StepScope & @JobScope

- `@StepScope`는 `Tasklet`, `ItemReader`, `ItemWriter`, `ItemProcessor` 등에 사용하며, Step 실행 시점에 Bean을 생성한다.
- `@JobScope`는 Step 선언부에 사용하며, Job 실행 시점에 Bean을 생성한다.

| 어노테이션   | 적용 대상                          | 생성 시점    |
| ------------ | ---------------------------------- | ------------ |
| `@JobScope`  | Step 구성 메서드                   | Job 실행 시  |
| `@StepScope` | Reader, Processor, Writer, Tasklet | Step 실행 시 |

## 4. JobParameter와 시스템 변수

- 시스템 변수(예: `-Dspring.batch.job.parameters...`)를 사용하는 경우 Spring Batch의 `JobParameter` 관련 기능을 사용할 수 없다.
  - Spring Batch는 **동일한 JobParameter로는 같은 Job을 두 번 실행하지 않는다.**
- 시스템 변수 기반 실행은 **커맨드라인 실행 외에는 유연성이 떨어진다.**
  - 테스트 코드나 동시 다중 실행이 어려우며, 전역 설정을 동적으로 바꾸어야 하는 등 설계 복잡도가 높아진다.
