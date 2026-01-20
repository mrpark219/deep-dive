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

## 5. CloudFormation Wait Condition

- 리소스의 **프로비저닝** 중 특정 주체가 로직을 수행 후 **성공 신호**를 알릴 때까지 리소스 프로비저닝을 **멈추는(Wait)** 구성이다.
  - 예: EC2 인스턴스를 프로비저닝할 때 다양한 설정 작업(웹 서버 설치 등)이 끝난 후 **완료 처리**를 한다.
- **N개**의 성공 신호를 **X초**까지 대기 가능하다(최대 **12시간**).
  - **실패 신호**를 받으면 실패 처리된다.
  - 시간 내에 N개의 성공 신호를 받지 못하면 **실패(Timeout)** 처리된다.
- **두 가지 방법**이 있다.
  - 별도의 `AWS::CloudFormation::WaitCondition` 리소스를 생성한다.
    - **다수**의 리소스를 컨트롤할 경우 사용한다.
  - 리소스의 **CreationPolicy**를 설정한다.
    - **EC2**, **Auto Scaling Group**의 경우 추천한다.

### 5.1. CloudFormation Helper Scripts

- CloudFormation과의 **소통**을 돕기 위한 **Python** 기반 스크립트이다.
- **Amazon Linux**에는 기본적으로 설치되어 있다.
  - 다른 OS에서는 **별도 설치**가 필요하다.

#### cfn-signal

- CloudFormation의 **WaitCondition** 리소스 혹은 **CreationPolicy**가 설정된 리소스에 **처리 완료 신호**를 보내는 스크립트이다.

```shell
cfn-signal --success|-s signal.to.send \
        --access-key access.key \
        --credential-file|-f credential.file \
        --exit-code|-e exit.code \
        --http-proxy HTTP.proxy \
        --https-proxy HTTPS.proxy \
        --id|-i unique.id \
        --region AWS.region \
        --resource resource.logical.ID \
        --role IAM.role.name \
        --secret-key secret.key \
        --stack stack.name.or.stack.ID \
        --url CloudFormation.endpoint
```

#### cfn-init

- 리소스 **메타데이터**(`AWS::CloudFormation::Init`)를 기반으로 **패키지 설치**, **파일 생성** 등을 담당하는 스크립트이다.

```shell
cfn-init --stack|-s stack.name.or.id \
         --resource|-r logical.resource.id \
         --region region \
         --access-key access.key \
         --secret-key secret.key \
         --role rolename \
         --credential-file|-f credential.file \
         --configsets|-c config.sets \
         --url|-u service.url \
         --http-proxy HTTP.proxy \
         --https-proxy HTTPS.proxy \
         --verbose|-v
```

#### cfn-get-metadata

- 특정 경로의 **메타데이터**를 확보(조회)하는 스크립트이다.

```shell
cfn-get-metadata --access-key access.key \
                 --secret-key secret.key \
                 --credential-file|f credential.file \
                 --key|k key \
                 --stack|-s stack.name.or.id \
                 --resource|-r logical.resource.id \
                 --role IAM.role.name \
                 --url|-u service.url \
                 --region region
```

#### cfn-hup

- **업데이트**를 체크하여 변경이 일어나면 **커스텀 로직**을 수행하는 데몬 스크립트이다.

```shell
cfn-hup --config|-c config.dir \
        --no-daemon \
        --verbose|-v
```

## 6. CloudFormation Custom Resource

- AWS 리소스 이외의 **프로비저닝 로직**을 지원하는 리소스이다.
  - CloudFormation에서 지원하지 않는 리소스 타입을 프로비저닝하거나, AWS와 무관한 **독립적인 로직** 수행이 가능하다.
- 기본적인 리소스처럼 **생성(Create)**, **업데이트(Update)**, **삭제(Delete)** 를 지원한다.
- 주요 용어
  - **Custom Resource Provider**: 커스텀 리소스를 프로비저닝하거나 로직을 수행하는 **주체**이다.
    - AWS 서비스(**Lambda**, **SNS**) 혹은 AWS 바깥의 주체도 가능하다(사람, 온프레미스 서버 등).
  - **Template Developer**: 커스텀 리소스가 포함된 템플릿을 작성하는 **개발자**이다.

### 6.1. Custom Resource 활용

- **Custom Resource Provider**가 주어진 요청에 따라 수행할 **로직** 및 **리소스**를 정의한다.
- Custom Resource Provider가 해당 로직 수행 주체에게 요청을 보낼 수 있는 **SNS 토픽** 혹은 **Lambda 함수**를 생성한다.
- **Template Developer**가 템플릿 안에 **커스텀 리소스**를 정의한다.
  - 이때 전달할 **Input 데이터**와 **Service Token**(SNS ARN 혹은 Lambda ARN)을 같이 명시한다.
- 이후 해당 템플릿을 **프로비저닝(생성, 업데이트, 삭제)** 할 때마다 해당 커스텀 리소스에 SNS/Lambda로 요청을 보내고 **응답**을 대기한다.
- 해당 리소스가 **로직 처리**를 완료하면 CloudFormation에 **응답**을 보낸다.
- **성공**이라면 이후 단계를 진행하며, **실패**(타임아웃, 실패 응답)한다면 **실패 처리**된다.

### 6.2. 주의할 점

- 기본 커스텀 리소스의 **Timeout**은 **1시간**이다.
  - 즉 잘못 프로비저닝하면 1시간 동안 스택 삭제가 **불가능**하다.
  - **ServiceTimeout** 속성을 사용하여 적절한 타임아웃 설정을 권장한다(예: 60초).
- **Lambda**로 커스텀 리소스를 구현할 경우 **삭제 시**에도 로직이 수행된다.
  - 즉 첫 프로비저닝 로직(생성)에 오류가 있어 수정하려 할 때, **삭제 로직** 또한 오류가 발생한다면 스택 삭제가 **불가능**해질 수 있다(예외 처리 필수).

## 7. CloudFormation Stack 생명주기

- **스택(Stack)**: CloudFormation으로 리소스를 프로비저닝할 때 관련된 리소스를 묶은 **단위**이다.
- **스택의 생명 주기**: **생성** -> **업데이트** -> **삭제** 순으로 진행된다.
- **Nested Stack(중첩 스택)**: 하나의 스택 안에 **다른 스택**을 포함하여 생성하는 방식이다.
- **Stack Set**: 한 번의 동작으로 **여러 계정**과 **여러 리전**에 스택을 프로비저닝할 수 있는 기능이다.

### 7.1. 스택의 생명 주기 - 생성

- **CloudFormation 템플릿**을 기반으로 생성한다.
  - S3 경로를 지정하거나 직접 템플릿을 업로드(S3에 업로드됨)하거나 **Git**에서 동기화한다.
- 가능한 설정
  - **IAM 역할**: 별도로 지정한 IAM 역할을 사용해서 프로비저닝 **가능**하다.
  - **실패 동작**
    - **모든 리소스 롤백**: 모든 리소스 프로비저닝 성공 혹은 **전체 롤백**한다(하나라도 실패하면 전부 롤백).
    - **성공한 리소스 보존**: 성공한 리소스는 그대로 두고 프로비저닝에 실패한 리소스만 **롤백**한다.
  - **롤백 정책**
    - **삭제 정책(DeletionPolicy) 활용**: `DeletionPolicy` 설정대로 삭제 여부를 판단한다.
    - **모든 리소스 삭제**: 롤백 중 생성된 리소스는 무조건 **삭제**한다.
  - **기타**: **SNS 알림**, **생성 제한 시간(Timeout)** 등이 있다.

### 7.2. 스택의 생명 주기 - 업데이트

- 기본 템플릿 기반으로 **파라미터**만 변경하거나 **새로운 템플릿** 기반으로 업데이트한다.
  - S3 경로를 지정하거나 직접 템플릿을 업로드하거나 **Git**에서 동기화한다.
- 가능한 설정은 생성 시와 거의 동일하다.
- **Stack Policy**: 스택 리소스의 **불필요한 업데이트**를 방지하기 위한 일종의 **보호 장치**이다.
  - 설정 시 **명시적**으로 허용한 리소스를 제외하고는 업데이트가 **불가능**하다.
  - 한 번 설정하면 **삭제**가 불가능하다.
  - **주요 사용 사례**: 특정 리소스만 업데이트를 허용하거나 특정 리소스의 **삭제**를 방지한다(스택 삭제는 가능).
- **변경 세트(Change Set)** 로 변경 내용에 대해 미리 **확인** 가능하다.
  - 단, 변경 **성공 여부**를 보장하지는 않는다.

```json
# Stack Policy 예제
{
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "Update:*",
      "Principal": "*",
      "Resource": "*"
    },
    {
      "Effect": "Deny",
      "Action": "Update:*",
      "Principal": "*",
      "Resource": "LogicalResourceId/ProductionDatabase"
    }
  ]
}
```

### 7.3. 스택의 생명 주기 - 삭제

- 스택 삭제 시 **전체 리소스**의 삭제를 시도한다.
  - `DeletionPolicy`에 따라 **삭제되지 않는** 리소스를 지정 가능하다.
- **삭제가 불가능한 경우**(예: S3 버킷에 파일이 남아있는 경우)
  - 다른 모든 리소스를 삭제 후 **실패 상태(DELETE_FAILED)** 로 남긴다.
  - 이후 **강제 삭제**를 시도(다시 시도)하거나, 해당 리소스를 **제외(Retain)** 하고 스택 삭제가 가능하다.

## 8. CloudFormation Nested Stack / Stack Set

### 8.1. Nested Stack

- 하나의 스택 안에 **다른 스택**을 포함하여 생성하는 기능이다.
  - 스택의 **모듈화**가 가능하다.
- **계층 구조**로 스택 프로비저닝이 가능하다.

### 8.2. Stack Set

- 하나의 템플릿으로 **다양한 리전** 및 **계정**에 동시에 프로비저닝 및 관리가 가능한 기능이다.
  - 일괄적인 **업데이트** 및 **삭제**가 가능하다.
- **별도의 역할**을 활용하여 프로비저닝한다(IAM 사용자 역할과 별도이다).
  - 프로비저닝을 관리하는 계정의 역할과 실제 배포되는 대상 계정의 역할이 **분리**되어야 한다.
- **다양한 설정**을 지원한다.
  - **리전** 및 **계정** 선택
  - **동시** 프로비저닝 스택 숫자 설정(Concurrency)
  - **배포 방식** 설정(순차/병렬, Failure Tolerance 등)

## 9. CloudFormation 추가 파라미터 타입

- 일반적인 파라미터 타입(Integer/String) 이외에 CloudFormation에서 지원하는 **추가 파라미터 타입**이다.
- **AWS Parameter**: AWS 리소스를 명시하는 파라미터 타입이다.
  - 예:
    - `AWS::EC2::InstanceId`
    - `AWS::EC2::Image::Id`
    - `List<AWS::EC2::AvailabilityZone::Name>`
- **Systems Manager Parameter**: AWS의 Systems Manager Parameter Store에서 값을 가져오는 타입이다.
  - 예: `AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>`

### 9.1. AWS Systems Manager

- AWS에서 인프라를 보고 제어하기 위해 사용할 수 있는 **AWS 서비스**이다.
- **Systems Manager 콘솔**을 사용하여 여러 AWS 서비스의 운영 데이터를 보고 AWS 리소스에서 **운영 태스크**를 자동화할 수 있다.

#### Parameter Store

- AWS에서 주요 설정과 값들을 **저장/관리/활용**하기 위한 서비스이다.
  - 예: API 주소, DB 호스트명, AMI ID, API Token, 사용자 ID/패스워드, 환경 변수 등
- **Public Parameter**: AWS에서 공식적으로 배포하는 파라미터이다.
  - 각 OS 별로 최신 AMI의 ID
  - AWS의 모든 리전 목록
  - AWS의 모든 서비스 목록
  - 특정 서비스를 사용 가능한 리전 목록

#### Parameter Store 값 참조

- CloudFormation 템플릿 안에서 **SSM Parameter Store**의 값을 참조 가능하다.
  - `{{resolve:ssm:파라미터 이름}}` 형식을 사용한다.
- 주요 사용 사례
  - **퍼블릭 파라미터** 참조(AMI Image ID, Endpoint 등)
  - **환경 설정 값**의 공유
    - DB 패스워드, 호스트 등
  - 기타 주요 값들 설정

```yaml
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: "{{resolve:ssm:/aws/service/ami-amazon-linux-latest/al2023-ami-kernel-6.1-x86_64}}"
      InstanceType: t2.micro
```

### 9.2. 파라미터 순서 및 규칙 정의

- CloudFormation의 파라미터를 입력받을 때 **그룹**, **라벨링**, **순서** 조정이 가능하다.
- **Metadata Section**을 활용한다.

```yaml
Metadata:
  AWS::CloudFormation::Interface:
    ParameterGroups:
      - Label:
          default: "Network Configuration"
        Parameters:
          - VPCID
          - SubnetId
          - SecurityGroupID
      - Label:
          default: "Amazon EC2 Configuration"
        Parameters:
          - InstanceType
          - KeyName
    ParameterLabels:
      VPCID:
        default: "Which VPC should this be deployed to?"
```

### 9.3. 다른 스택의 Output 참조

- 다른 스택에서 **Outputs 섹션**으로 내보낸(Export) 값을 참조 가능하다.
- `!ImportValue` Intrinsic Function(내장 함수)으로 참조한다.
- 예: `!ImportValue MyExportedName`

## 10. CloudFormation 기타

### 10.1. 콘솔에서는 숨겨진 리소스들

- CloudFormation에서 프로비저닝 시 콘솔에서는 보이지 않는 **숨겨진 리소스**가 존재한다.
  - 주로 하나의 리소스와 다른 리소스를 **연결**해주는 리소스이다.
  - 예:
    - **Instance Profile**: IAM 역할과 EC2를 연결한다(`AWS::IAM::InstanceProfile`).
    - **Volume Attachment**: EBS 볼륨과 EC2를 연결한다(`AWS::EC2::VolumeAttachment`).

### 10.2. CloudFormation 가져오기(Import)

- 기존에 존재하는 리소스를 생성된 CloudFormation **논리적 리소스(스택)**으로 가져오는 기능이다.
  - 스택 간의 리소스 **이동**도 가능하다.
- 두 가지 방법
  - **IaC Generator**: 기존 리소스를 기반으로 자동으로 템플릿을 생성해 주는 서비스이다(종종 권장되지 않음).
  - **수동 가져오기**: 수동으로 기존 리소스에 해당하는 리소스를 템플릿에 정의하여 가져오는 방법이다.

### 10.3. Drift Detection

- **Drift**: CloudFormation의 **논리적 리소스**(템플릿)와 실제 **물리적 리소스**의 상태가 **다른 경우**를 말한다.
  - 주로 물리적 리소스를 CloudFormation 이외의 수단(SDK, 콘솔, CLI)으로 **직접 변경**한 경우에 발생한다.
- CloudFormation에서 스택의 **Drift Detection**을 지원한다.
  - 즉, 현재 상태를 기반으로 실제 **리소스**를 모니터링하여 변경점을 확인한다.
  - 해결 방법으로는 템플릿을 수정하여 실제 상태와 맞추거나, 해당 리소스를 **Retain 삭제**(보존 삭제) 후 다시 **가져오기(Import)** 등을 수행해야 한다.
