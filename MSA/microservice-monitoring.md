# Microservice 모니터링

## 1. Micrometer

- **JVM 기반 애플리케이션의 Metrics를 수집하고 관리하기 위한 라이브러리**이다.
- Spring Framework 5와 Spring Boot 2부터 **Spring의 Metrics 처리를 Micrometer가 담당**한다.
- Prometheus, Datadog, New Relic 등 다양한 **모니터링 시스템과의 연동을 지원**한다.
- Spring Boot에서 기본적으로 Micrometer를 내장하고 있으며, `@Timed`와 같은 어노테이션을 통해 코드에서 손쉽게 사용 가능하다.

### 1.1 Timer

- **짧은 지연 시간 및 이벤트 사용 빈도**를 측정한다.
- 타이머를 통해 시계열로 이벤트의 **수행 시간, 호출 빈도, 총 호출 수** 등을 수집할 수 있다.
- `@Timed` 어노테이션을 메서드에 붙이면 해당 메서드의 실행 시간과 호출 횟수를 자동으로 기록한다.

## 2. Prometheus

- Metrics를 수집하고 **모니터링 및 알람**에 사용되는 오픈소스 애플리케이션이다.
- 2016년부터 CNCF에서 관리되는 두 번째 공식 프로젝트이다. (첫 번째는 쿠버네티스이다.)
- 내부적으로 LevelDB를 사용하다가 **시계열 데이터베이스(TSDB)** 기반으로 설계되었다.
- **Pull 방식**으로 Metric 데이터를 수집하며, 다양한 **Metric Exporter**를 제공한다.
- 수집된 데이터를 시계열 DB에 저장하며, **PromQL 쿼리 언어를 통해 데이터 조회**가 가능하다.

## 3. Grafana

- 데이터 **시각화, 모니터링, 분석**을 위한 오픈소스 애플리케이션이다.
- Prometheus, InfluxDB, Elasticsearch 등 다양한 데이터 소스로부터 데이터를 가져올 수 있다.
- 수집된 시계열 데이터를 보기 위한 **대시보드 구성 기능**을 제공한다.
