# CloudFormation

## 1. 인프라 구성의 역사

| 구분                        | 소요 시간       | 주요 작업                                                                          |
| :-------------------------- | :-------------- | :--------------------------------------------------------------------------------- |
| Pre-Cloud                   | 몇 주 ~ 몇 개월 | - 데이터 센터/서버 확보<br>- 네트워크 공사 및 기기 설치<br>- OS/프로그램 수동 설치 |
| Cloud                       | 몇 시간         | - 리소스 선별<br>- 데이터 입력 및 클릭 (반복)                                      |
| IaC(Infrastructure as Code) | 몇 분           | - 설정파일 업로드<br>- 끝                                                          |

### 1.1. IaC(Infrastructure as Code)

- 클라우드에서 인프라를 **코드**로 관리하는 방법이다.
- 장점
  - 항상 **일관된 아키텍처**를 구현한다.
    - **휴먼 에러**를 배제할 수 있다.
  - **재사용성**이 뛰어나다.
    - 일정 단위의 **모듈**로 만들어 다양한 프로젝트에서 재사용 가능하다.
    - 인프라의 구성을 다른 사람과 **공유**하거나 **복제** 가능하다.
  - **관리**가 용이하다.
    - 리소스의 **변경** 및 **이력 추적**이 가능하다.
    - **일괄 삭제**가 가능하다.
    - **Git** 등을 활용한 **형상 관리**가 가능하다.

## 2. CloudFormation

- AWS 리소스를 모델링하고 설정하여 리소스 관리 시간을 줄이고, 애플리케이션 실행에 더 많은 시간을 사용할 수 있도록 돕는 **서비스**이다.
- AWS의 IaC 서비스 이다.
- 주요 사용 사례
  - 인프라를 **자동**으로 프로비저닝하고 싶을 때
  - 인프라를 **재사용**하거나 **공유**하고 싶을 때
  - 정형화된 아키텍처를 **코드화**하여 관리하고 싶을 때
  - 인프라의 **프로비저닝** 및 **업데이트** 과정을 체계적으로 관리하고 싶을 때
- 코드 기반으로 AWS 인프라를 여러 리전에 자동으로 **프로비저닝**하거나 **업데이트**한다.
  - 프로비저닝 및 업데이트 과정에서 오류 발생 시 자동으로 **롤백(Rollback)** 된다.
- **JSON**과 **YAML** 형식을 지원하며, 재사용성 및 편의를 위한 수많은 기능을 포함한다.
- **커스텀 리소스**를 사용하여 거의 모든 작업을 수행할 수 있다.
  - 예: 슬랙 알림, 온프레미스 데이터 센터의 리소스 구성 등

### 2.1. CloudFormation 주요 개념

- **템플릿(Template)**: JSON 또는 YAML 형식의 텍스트 파일로 AWS 인프라를 구성하는 리소스들의 **청사진**이다.
  - 프로비저닝할 리소스를 **정의**하고 리소스 간의 **관계**를 정의한다.
  - 다양한 **섹션**과 기능들로 구성된다.
  - **재사용** 가능하다.
- **논리적 리소스**: 템플릿을 기반으로 실제 프로비저닝되는 리소스를 대표하는 **논리적 개념**이다.
  - 템플릿 + 사용자의 **파라미터**/프로비저닝 시점의 값들로 구성된다.
- **스택(Stack)**: CloudFormation으로 리소스를 프로비저닝할 때 관련된 리소스를 묶은 **단위**이다.
  - **스택 ID**와 **이름**으로 구분한다(ID는 글로벌 고유 값, 이름은 리전 단위 고유 값).
  - 논리적 리소스의 구성을 실제 리소스로 **반영**한다.
    - 즉 논리적 리소스가 변경되면 실제 **리소스**도 변경된다.
  - 삭제 시 스택 단위로 삭제 가능하며, 스택 삭제 시 **모든 리소스**가 삭제된다.

## 3. 템플릿(Template)

- JSON 또는 YAML 형식의 텍스트 파일로 AWS 인프라를 구성하는 리소스들의 **청사진**이다.
- **텍스트 파일**로서의 다음과 같은 특징을 가진다.
  - 에디터로 수정 가능하다.
  - 소스 컨트롤(Git 등)이 가능하다.
  - LLM으로 생성이 가능하다.
- 다양한 **섹션**과 **기능**으로 구성된다.
  - **Resources 섹션**은 필수이며 나머지는 선택이다.
  - **Resources**, **Parameters**, **Intrinsic Functions** 3가지 요소가 가장 기초 단위이다.
- 특수한 상황을 제외하고 스택을 생성하는 **IAM 사용자**의 권한을 활용하여 프로비저닝한다.

### 3.1. 템플릿 생성 시 목표

- 최대한 외부의 도움 없이 **스스로 동작**할 수 있어야 한다(**Portability**).
  - 예: EC2 AMI 기본 값으로 프로비저닝하는 리전의 Amazon Linux 2023 ID를 가져온다.
- **파라미터**를 통해 커스터마이징 가능하도록 설정하여 **범용성**을 확보해야 한다.
  - 예: 하나의 템플릿에 Prod/Dev 옵션에 따라 다르게 프로비저닝한다.
  - 예: EC2 인스턴스 타입 등을 입력받아 다양한 상황에 일괄적으로 사용할 수 있어야 한다.
- 가능한 **독립적**으로 만들어야 한다.
  - 추후 다른 스택에서 템플릿을 참조할 수 있도록 **모듈화**가 가능해야 한다.

### 3.2. 템플릿 섹션(Section)

- **Format Version (Optional)**: 템플릿의 버전을 명시한다.
- **Description (Optional)**: 템플릿에 대한 설명을 기술한다.
- **Metadata (Optional)**: 템플릿에 대한 추가 데이터를 명시한다.
- **Parameters (Optional)**: 템플릿을 **프로비저닝**하는 시점에 논리적 리소스에 전달하는 값을 정의한다.
- **Rules (Optional)**: 파라미터를 **검증**하는 정책을 명시한다.
- **Mappings (Optional)**: 추가적으로 지정한 변수 **Map**으로, 키-값 쌍을 정의하여 파라미터처럼 사용한다.
- **Conditions (Optional)**: 템플릿 프로비저닝 시 **조건**을 사용하여 리소스 생성 여부 등을 제어할 때 활용한다.
- **Transform (Optional)**: 템플릿에서 사용하는 **매크로(Macro)** 등을 정의한다(예: Serverless Application Model).
- **Resources (Required)**: 템플릿에서 프로비저닝할 리소스를 정의하는 **필수** 섹션이다.
- **Outputs (Optional)**: 템플릿 프로비저닝 이후 추가적으로 **명시할** 내용을 정의하며, 추후 다른 스택에서 **참조** 가능하다.

#### Format Version / Description 섹션

- **CloudFormation**의 버전과 설명을 명시하는 섹션이다.
- **Description** 섹션은 반드시 **Format Version** 섹션 바로 뒤에 위치해야 한다.

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: A sample template
```

#### Resources 섹션

- 스택에서 실제로 프로비저닝할 리소스를 정의하는 **유일한 필수 섹션**이다.
- **리소스의 구성 요소**는 다음과 같다.
  - **Logical Name (논리적 이름)**: 스택 안에서 리소스를 구분하기 위한 **ID**이다.
    - 실제 리소스 이름/ID와는 다른 개념으로 **스택 안**에서만 통용된다.
  - **Type (유형)**: 리소스의 유형(S3, EC2 등)을 지정한다.
    - 형식: `AWS::ServiceName::ResourceType`
    - 예: `AWS::EC2::Instance`
  - **리소스 속성 (Resource Attribute)**: 리소스 자체의 **공통적인 속성**을 정의한다.
    - **삭제**, **업데이트**, 리소스의 **전후 관계** 등을 정의한다.
  - **속성 (Properties)**: 리소스의 **유형별**로 자세한 설정을 정의한다.
    - 예: EC2의 속성 -> **EC2 인스턴스 유형**, **AMI**, **보안 그룹**, **EBS 설정**, **태그** 등
    - 다른 **리소스** 혹은 **파라미터**를 참조하여 설정 가능하다.

```yaml
Resources:
  MyBucket:
    Type: "AWS::S3::Bucket"
    Properties:
      BucketName: amzn-s3-demo-bucket
```

#### Parameters

- 템플릿을 **커스터마이징**하기 위해 설정 가능한 값이다.
- 두 가지 종류가 있다.
  - **일반 파라미터**: 템플릿 프로비저닝 시 사용자에게 받는 값으로 **Parameters Section**에서 정의한다.
    - 예: **EC2 Instance Type**, **RDS DB** 기본 **패스워드** 등
    - 활용 예시: 하나의 정형화된 아키텍처를 배포할 때 **DNS**만 교체하거나 **인스턴스 타입**만 교체하는 경우에 사용한다.
  - **Pseudo 파라미터(의사 매개변수)**: CloudFormation에서 템플릿 프로비저닝 시 **자동으로 지정**해주는 값으로, 프로비저닝 당시의 상황을 반영하여 스택으로 전달된다.
    - 예: `AWS::Region`, `AWS::AccountId`, `AWS::StackName`
    - 활용 예시
      - 리소스의 **ARN**을 구성할 때 현재 **리전**과 **계정 ID**를 동적으로 입력하거나, **태그**에 스택 이름을 자동으로 부여할 때 사용한다.
      - 리소스의 이름 등에 계정명 혹은 리전명 등을 포함하여 중복된 이름이 나오지 않도록 설정할 수 있다.
    - `!Ref` 혹은 `!Sub`로 활용한다.
      - **!Sub**: 특정 값들(파라미터, 다른 함수 등)로 **스트링**을 만드는 함수이다.

#### Parameters 섹션

- 템플릿에서 **일반 파라미터**를 정의하는 섹션이다.
- 형식
  - Logical ID: 스택 내에서 통용되는 **파라미터 이름**이다.
  - Type: **String**, **Number**, **List<Number>** 등 기본 타입과 **AWS 지정 타입**(AWS-Specific Types)으로 AWS 리소스(**Subnet**, **EC2 Key Pair** 등) 지정이 가능하다.
  - Description: 파라미터에 대한 **설명**을 작성한다.
  - Default: **기본값**을 설정한다.
  - AllowedValues: **허용 가능한 값**의 목록을 정의한다.
  - 기타: 파라미터 **조건**(길이, 정규식 등)이나 **NoEcho**(암호 마스킹 등에 활용) 설정을 포함한다.
- `!Ref` **Intrinsic Function(내장 함수)** 으로 참조한다.

```yaml
Parameters:
  VPC:
    Type: String
    Default: vpc-123456
  VpcAzs:
    Type: CommaDelimitedList
    Default: us-west-2a, us-west-2b
  DbSubnetIpBlocks:
    Type: CommaDelimitedList
    Default: 172.16.0.0/26, 172.16.0.64/26
Resources:
  DbSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      AvailabilityZone: !Select
        - 0
        - !Ref VpcAzs
      VpcId: !Ref VPC
      CidrBlock: !Select
        - 0
        - !Ref DbSubnetIpBlocks
  DbSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      AvailabilityZone: !Sub
        - ${AWS::Region}${AZ}
        - AZ: !Select
            - 1
            - !Ref VpcAzs
      VpcId: !Ref VPC
      CidrBlock: !Select
        - 1
        - !Ref DbSubnetIpBlocks
```

#### Intrinsic Function (내장 함수)

- CloudFormation에서 지원하는 **기본 함수**로, 스택 관리를 위한 다양한 기능을 제공한다.
  - **Resources 속성**, **Outputs**, **Metadata**, **UpdatePolicy** 속성에서만 사용 가능하다.
- 두 가지 호출 방식이 있다.
  - **Full Function Name**: `Fn::FunctionName`
  - **Short Form**: `!FunctionName`
- 예시 (Ref): 파라미터 혹은 리소스를 참조하기 위한 함수이다.
  - `Fn::Ref: LogicalID`
  - `!Ref LogicalID`
- 주요 사용 사례
  - 다른 **리소스 참조**
  - AWS 계정의 **AZ 목록 확보**
  - 리소스의 **속성** 불러오기

```yaml
Resources:
  DbSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      AvailabilityZone: !Select
        - 0
        - !Ref VpcAzs
      VpcId: !Ref VPC
      CidrBlock: !Select
        - 0
        - !Ref DbSubnetIpBlocks
  DbSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      AvailabilityZone: !Sub
        - ${AWS::Region}${AZ}
        - AZ: !Select
            - 1
            - !Ref VpcAzs
      VpcId: !Ref VPC
      CidrBlock: !Select
        - 1
        - !Ref DbSubnetIpBlocks
```

#### Mappings 섹션

- CloudFormation에서 미리 **Map**으로 데이터를 정의할 수 있는 섹션이다.
  - **프로비저닝** 상황에 따라 알맞은 값을 선택할 수 있도록 미리 데이터를 저장한다.
- `Fn::FindInMap` **Intrinsic Function(내장 함수)** 으로 활용한다.
  - `Fn::FindInMap: [MapName, TopLevelKey, SecondLevelKey]`
  - `!FindInMap [MapName, TopLevelKey, SecondLevelKey]`
- 주요 사용 사례
  - 리전별 AMI 선택
  - Route53 Hosted Zone 선택 등

```yaml
Mappings:
  MappingLogicalName:
    Key1:
      Name: Value1
    Key2:
      Name: Value2
    Key3:
      Name: Value3
```

#### Outputs 섹션

- 리소스가 **프로비저닝**된 후에 필요한 정보를 스택 바깥으로 **내보내는(Export)** 섹션이다.
  - 내보낸 정보는 **다른 스택**에서 참조해서 사용하거나 **콘솔**에서 확인 가능하다.
  - `Fn::ImportValue` 함수를 통해 다른 스택에서 값을 가져올 수 있다(**Cross-Stack Reference**).
- **구성 요소**는 다음과 같다.
  - **Logical ID**: 출력값의 고유 **식별자**이다.
  - **Description**: 출력값에 대한 **설명**이다.
  - **Value**: 실제 반환할 **값**이다(주로 `!Ref` 혹은 `!GetAtt`로 리소스의 속성을 반환).
  - **Export**: 다른 스택에서 참조할 때 사용할 **고유 이름**을 정의한다.

```yaml
Outputs:
  OutputLogicalID:
    Description: Information about the value
    Value: Value to return
    Export:
      Name: Name of resource to export
```

#### Conditions 섹션

- **조건**에 따라 리소스 생성 여부나 설정을 **결정**할 수 있는 섹션이다.
  - 예: Dev 환경이라면 더 작은 타입의 EC2 인스턴스를 생성한다.
  - 예: Prod 환경이라면 Multi-AZ로 DB를 구성한다.
- `!If`, `!Equals`, `!Not` 등의 **Intrinsic Function(내장 함수)** 을 활용한다.

```yaml
Properties:
  InstanceType: !If
    - IsProduction
    - c5.xlarge
    - t3.small
```

#### 주요 Intrinsic Function 목록

- **Ref**: 파라미터 혹은 리소스를 **참조**하기 위한 함수이다.
  - `Fn::Ref: LogicalID`
  - `!Ref LogicalID`
- **Fn::GetAtt**: 리소스의 특정 **속성(Attribute)** 을 가져오는 함수이다.
  - `Fn::GetAtt: [LogicalID, AttributeName]`
  - `!GetAtt LogicalID.AttributeName`
- **Fn::Sub**: 특정 값들(파라미터, 다른 함수 등)로 **문자열(String)** 을 조합하거나 치환하는 함수이다.
- **기타 함수**
  - **Fn::GetAZs**: 가용 영역(AZ) 목록을 반환한다.
  - **Fn::Join**: 구분자를 사용하여 값의 목록을 하나의 문자열로 결합한다.
  - **Fn::Base64**, **Fn::Cidr**, **Fn::Split** 등이 있다.

## 4. CloudFormation 리소스 속성

- 리소스 자체의 **공통적인 속성**을 정의한다.
  - **CreationPolicy**: 리소스 생성 시 **특정 조건**을 만족해야 완료 처리될 수 있도록 설정한다.
    - 예: EC2에서 웹 서버가 설치되어야 EC2 생성을 완료로 간주한다.
  - **DeletionPolicy**: 리소스 **삭제 정책**을 정의한다.
  - **DependsOn**: 특정 리소스가 생성된 **이후**에 해당 리소스 생성을 시작하도록 설정한다.
    - 예: 반드시 RDS가 생성된 후에 EC2 생성을 시작해서 원활하게 서버가 설정되도록 구성한다.
  - **Metadata**: 리소스의 **추가적인 정보**를 제공한다.
  - **UpdatePolicy**: 리소스를 업데이트할 때의 **절차(프로세스)**를 정의한다.
    - 예: **Auto Scaling Group**의 경우 업데이트 시 인스턴스 업데이트 방식(Rolling 등)을 정의한다.
  - **UpdateReplacePolicy**: 리소스가 **교체(삭제 후 재생성)** 될 때, **기존 리소스**를 어떻게 할지 정의한다.
    - 예: EC2 인스턴스 업데이트 시 기존 EBS 볼륨의 **스냅샷**을 만들 것인지 여부 등을 정의한다.

### 4.1. DeletionPolicy

- **Delete**: 스택 삭제 시 같이 **삭제**된다(몇몇 리소스를 제외하고는 **기본 옵션**이다).
- **Retain**: 스택 삭제 시 해당 리소스는 삭제하지 않고 **보존**한다.
  - **주의**: 리소스 생성에 실패하여 **롤백**할 때도 보존된다.
  - **RetainExceptOnCreate**: 리소스를 **처음 생성**할 때를 제외하고 보존한다. 즉, 처음 생성 시 실패한다면 삭제된다.
- **Snapshot(지원 시)**: 스냅샷을 지원하는 리소스(EC2, RDS 등)의 경우 삭제 시 **스냅샷**을 생성하고 삭제한다.
  - 지원 리소스: **RDS**, **EBS**, **Redshift**, **ElastiCache** 등

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    DeletionPolicy: Retain
```

### 4.2. DependsOn

- `!Ref`, `!GetAtt`, `!Sub`로 묶인 경우에는 **암시적**으로 참조하는 리소스를 먼저 생성한 후 해당 리소스를 생성한다.
- 즉, 해당 참조가 없는 상태에서 리소스의 **생성 순서**를 명시적으로 제어하기 위해 사용한다.

```yaml
Resources:
  Ec2Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: "{{resolve:ssm:/aws/service/ami-amazon-linux-latest/al2023-ami-kernel-6.1-x86_64}}"
      InstanceType: t2.micro
    DependsOn: myDB
  myDB:
    Type: AWS::RDS::DBInstance
    Properties:
      AllocatedStorage: "5"
      DBInstanceClass: db.t2.small
      Engine: MySQL
      EngineVersion: "5.5"
      MasterUsername: "{{resolve:secretsmanager:MySecret:SecretString:username}}"
      MasterUserPassword: "{{resolve:secretsmanager:MySecret:SecretString:password}}"
```
