# Systems Manager

## 1. Systems Manager

![https://docs.aws.amazon.com/images/whitepapers/latest/aws-systems-manager-operational-capabilities/images/systems-manager-overview.png](./images/systems-manager/2026-01-21-21-20-17.png)

- AWS에서 인프라를 보고 제어하기 위해 사용할 수 있는 **AWS 서비스**이다.
- **Systems Manager 콘솔**을 사용하여 여러 AWS 서비스의 운영 데이터를 보고, AWS 리소스에서 **운영 태스크**를 자동화할 수 있다.
- **관리형 인스턴스**를 검사하고 탐지된 정책 위반을 보고하거나 시정 조치를 취해서 **보안** 및 **규정 준수**를 유지하는 데 도움이 된다.
- 주요 사용 사례
  - AWS 리소스를 쉽게 **관리**하고 **파악하고 싶을 때** 사용한다.
  - AWS 및 온프레미스 리소스에 쉽게 **접근하고 싶을 때** 사용한다.
  - AWS 위에서 구동되는 다양한 **앱**, **소프트웨어**, **OS** 등의 **상태를 확인하고 싶을 때** 사용한다.
  - 다양한 시스템 환경 구성 설정 및 **파라미터를 관리하고 싶을 때** 사용한다.
- 작동 방식
  - **Agent** 설치로 운영 중인 AWS/온프레미스 서버의 **상태**를 파악한다.
  - 다양한 **자동화 서비스** 및 **조회 서비스**로 제어 및 현황을 파악한다.
  - 기타 **관리 기능**을 제공한다.

### 1.1. 주요 기능

- **Run Command**: 등록된 여러 EC2 인스턴스 및 On-Premise 인스턴스에 **명령**을 실행한다.
  - 예: OS 업데이트, 전체 서버 리부팅 등
- **Patch Manager**: **패치 기준**을 정하여 인스턴스가 항상 정한 기준을 지켜 **최신 업데이트 상태**를 유지할 수 있도록 설정한다.
- **Inventory**: 등록된 인스턴스의 데이터를 종합해 **통계 데이터**를 생성하여 표시한다.
  - 예: 설치된 OS 종류별 통계, 설치된 애플리케이션, 실행 중인 서비스 목록 등
- **State Manager**: 인스턴스의 **상태 기준**을 정해 인스턴스들이 정한 기준의 상태를 **준수**하도록 설정한다.
  - 예: 특정 소프트웨어가 설치되어 있어야 함, 방화벽이 켜져 있어야 함 등
- **Parameter Store**: AWS 및 여러 서비스에서 사용하는 **값**을 저장하고 손쉽게 사용하기 위한 서비스이다.
- **Session Manager**: 인스턴스에 대해 **원클릭 액세스**를 제공하는 관리형 서비스이다.
  - 인스턴스에 **SSH 연결** 없이, **포트**를 열 필요 없이, **배스천 호스트(Bastion Host)** 를 유지할 필요 없이 인스턴스에 **로그인** 가능하다.
- **Automation**: AWS에서 미리 지정된 행동들을 **자동화**하여 실행시켜 주는 서비스이다.
  - 예: AMI 생성, EC2의 시작/정지 등
- **Incident Manager**: 갑작스러운 사고 및 장애 상황 등의 대응을 위한 **워크플로우**를 제공하는 서비스이다.
- **Change Calendar**: 리소스 **관리 일정**을 설정해 주는 서비스이다.
  - 예: 특정 날짜에 인스턴스 업데이트, 특정 주기별로 인스턴스 중지/시작 등
- **Quick Setup**: 다양한 Systems Manager의 기능을 엮어서 **빠르게** 사용할 수 있도록 준비한 서비스이다.

### 1.2. SSM Agent

- **Systems Manager Agent**: EC2 인스턴스, 온프레미스, VM에 설치해서 사용하는 **Systems Manager 전용 Agent**이다.
  - Systems Manager의 노드 **업데이트**, **관리**, **설정**을 지원한다.
- Systems Manager와 **통신**할 수 있는 환경이 필요하다.
  - `ssmmessages`, `ec2messages` 서비스와 통신한다.
  - **보안 그룹**, **NACL**, **방화벽** 등의 허용이 필요하다.
- **IAM 권한**이 필요하다.
  - 대표 정책: `AmazonSSMManagedInstanceCore`
- **Amazon Linux AMI** 및 AWS 지원 AMI 등에는 **기본 설치**되어 있다.
- 기타 AWS 서비스 등을 활용해서 SSM Agent의 **자동 업데이트**가 가능하다.
