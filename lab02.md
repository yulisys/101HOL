

# 실습 2: DB 서버 구축 및 앱을 사용하여 DB와 연동

본 실습은 AWS 관리형 데이터베이스 인스턴스를 활용하여 관계형 데이터베이스 요구 사항을 해결하는 개념을 강화하도록 설계되었습니다.

***Amazon Relational Database Service***\(Amazon RDS\)를 사용하면 클라우드에서 관계형 데이터베이스를 더욱 간편하게 설정, 관리 및 확장할 수 있습니다. 시간 소모적인 데이터베이스 관리 작업을 관리하는 한편, 효율적인 비용으로 크기 조정이 가능한 용량을 제공하므로 사용자는 애플리케이션과 비즈니스에 좀 더 집중할 수 있습니다. Amazon RDS를 사용하면 6개의 익숙한 데이터베이스 엔진, 즉 Amazon Aurora, Oracle, Microsoft SQL Server, PostgreSQL, MySQL 및 MariaDB 중에서 원하는 것을 선택할 수 있습니다.

**목표**

본 실습을 완료하면 다음을 할 수 있습니다.

- 고가용성을 갖춘 Amazon RDS DB 인스턴스를 시작
- 웹 서버로부터의 연결을 허용하도록 DB 인스턴스를 구성
- 웹 애플리케이션을 열고 데이터베이스와 상호 작용

**소요 시간**

본 실습에는 약 **45분** 정도가 소요됩니다.

**시나리오**

다음 인프라에서 시작합니다.
![architecture-lab1.png](https://ap-northeast-1-tcprod.s3.amazonaws.com/courses/ILT-TF-100-TECESS/v4.7.16/lab-2-configure-website-datastore/instructions/ko_kr/images/architecture-lab1.png)

실습을 완료하면, 다음과 같은 인프라가 구성됩니다.
![architecture-lab2.png](https://ap-northeast-1-tcprod.s3.amazonaws.com/courses/ILT-TF-100-TECESS/v4.7.16/lab-2-configure-website-datastore/instructions/ko_kr/images/architecture-lab2.png)

## 실습 시작
---------------------------------------

## 과제 1: RDS DB 인스턴스에 대한 VPC 보안 그룹 생성

본 과제에서는 웹 서버가 RDS DB 인스턴스에 액세스하도록 허용하는 VPC 보안 그룹을 생성합니다.

1. **AWS Management Console**의 <span style="background-color:#232f3e; font-weight:bold; font-size:90%; color:white; position:relative; top:-1px; padding-top:3px; padding-bottom:3px; padding-left:10px; padding-right:10px;">Services <i class="fas fa-angle-down"></i></span> 메뉴에서 **VPC** 를 클릭합니다.

2. 콘솔 언어를 영어로 변경합니다. 화면 맨 왼쪽 아래 **Feedback(의견)**  바로 오른쪽에 언어를 확인합니다. **한국어** 로 되어 있다면 반드시 **English(US)** 로 변경하십시오. 이 실습 가이드는 **English** 를 기준으로 작성 되었습니다.

3. 왼쪽 탐색 창에서  **Security Groups** 를 클릭합니다.

4. <span style="background-color:#ec7211; font-weight:bold; font-size:90%; color:white; position:relative; top:-1px; padding-top:3px; padding-bottom:3px; padding-left:10px; padding-right:10px;white-space: nowrap">Create security group</span>를 클릭한 후 다음 설정을 구성합니다.

- **Security group name:** `DB Security Group`
- **Description:** `Permit access from Web Security Group`
- **VPC:** _Lab VPC_

이제 인바운드 데이터베이스 요청을 허용하는 규칙을 Security Group에 추가하십시오. 보안 그룹에는 현재 규칙이 없습니다. _Web Security Group_ 에서 액세스를 허용하는 규칙을 추가하십시오.

5. **Inbound rules** 세션에서 <span style="background-color:white; font-weight:bold; font-size:90%; color:#545b64; position:relative; top:-1px; border-color:#545b64; border-radius:2px; border-width:1px; border-style:solid; padding-top:3px; padding-bottom:3px; padding-left:10px; padding-right:10px;">Add rule</span>을 클릭 하고 다음을 구성합니다:

- **Type:** _MySQL/Aurora (3306)_
- **Source** 의 검색창에 Type `sg` 라고 입력한 후에  _Web Security Group_ 을 찾아서 선택합니다.

6. 화면 맨아래 <span style="background-color:#ec7211; font-weight:bold; font-size:90%; color:white; position:relative; top:-1px; padding-top:3px; padding-bottom:3px; padding-left:10px; padding-right:10px;white-space: nowrap">Create security group</span> 클릭합니다.

이제 Amazon RDS 데이터베이스를 실행할 때 이 Security Group을 사용할 수 있습니다.

## 과제 2: DB 서브넷 그룹 생성하기

본 과제에서는 데이터베이스에 사용할 수 있는 서브넷을 RDS에 알리는 데 사용되는 DB 서브넷 그룹을 만듭니다. 각 DB 서브넷 그룹은 지정된 리전에서 두 개 이상의 가용 영역에 서브넷이 있어야 합니다.

7. <span style="background-color:#232f3e; font-weight:bold; font-size:90%; color:white; position:relative; top:-1px; padding-top:3px; padding-bottom:3px; padding-left:10px; padding-right:10px;">Services <i class="fas fa-angle-down"></i> </span> 메뉴에서 **RDS** 를 클릭합니다.

8. 왼쪽 탐색 창에서 **Subnet groups**를 클릭합니다.

<i class="fas fa-exclamation-triangle"></i> 탐색 창이 보이지 않으면 왼쪽 상단에서 <i class="fas fa-bars"></i> 메뉴 아이콘을 클릭합니다.

9. <span style="background-color:#ec7211; font-weight:bold; font-size:90%; color:white; position:relative; top:-1px; padding-top:3px; padding-bottom:3px; padding-left:10px; padding-right:10px;white-space: nowrap">Create DB Subnet Group</span> 을 클릭 후 다음 설정을 구성합니다.

- **Name:** `DB Subnet Group`
- **Description:** `DB Subnet Group`
- **VPC ID:** _Lab VPC_

10. **Add subnets** 세션에서 *Availability zones*   <i class="fas fa-caret-down"></i> 을 클릭 한 후에 다음을 선택합니다.

- <i class="far fa-check-square"></i> 첫번째 가용영역을 선택합니다.
- <i class="far fa-check-square"></i> 두번째 가용영역을 선택합니다.

11. **Subnets** <i class="fas fa-caret-down"></i> 을 클릭후 다음을 구성합니다.

- 첫번째 가용영역에 있는 <i class="far fa-check-square"></i> _10.0.1.0/24_ 를 선택
- 두번째 가용영역에 있는 <i class="far fa-check-square"></i> _10.0.3.0/24_ 를 선택

12. <span style="background-color:#ec7211; font-weight:bold; font-size:90%; color:white; position:relative; top:-1px; padding-top:3px; padding-bottom:3px; padding-left:10px; padding-right:10px;white-space: nowrap">Create</span>를 클릭합니다.

여기에는 Private Subnet 1(10.0.1.0/24)과 Private Subnet 2(10.0.3.0/24)가 추가됩니다. 다음 작업에서 데이터베이스를 작성할 때 이 DB 서브넷 그룹을 사용하십시오.

## 과제 3: RDS DB 인스턴스 생성

본 과제에서는 MySQL 지원 Amazon RDS DB 인스턴스를 구성 및 시작합니다.

Amazon RDS **Multi-AZ** 배포는 데이터베이스(DB) 인스턴스에 향상된 가용성과 내구성을 제공하므로 프로덕션 데이터베이스 워크로드에 적합합니다. Multi-AZ DB 인스턴스로 프로비저닝하면 Amazon RDS는 자동으로 기본 DB 인스턴스를 생성하고 데이터를 다른 AZ(Availability Zone)의 대기 인스턴스에 동기적으로 복제한다.

13. 왼쪽 탐색 창에서 **Databases** 를 클릭합니다.

14. <span style="background-color:#ec7211; font-weight:bold; font-size:90%; color:white; position:relative; top:-1px; padding-top:3px; padding-bottom:3px; padding-left:10px; padding-right:10px;white-space: nowrap">Create database</span> 를 클릭합니다.

<i class="fas fa-exclamation-triangle"></i> 만일 상단 메뉴에 **Switch to the new database creation flow** 가 보이면 클릭 하십시오.

15. <i class="far fa-dot-circle"></i> **MySQL** 을 선택합니다.

16. **Settings** 설정 아래에서 다음을 구성합니다.

- **DB instance identifier:** `lab-db`
- **Master username:** `lab_admin`
- **Master password:** `lab-password`
- **Confirm password:** `lab-password`

17. **DB instance class** 설정 아래에서 다음을 구성합니다.

 - <i class="far fa-dot-circle"></i> **Burstable classes (includes t classes)** 선택
 - _db.t3.micro_  (반드시 t3로 선택하세요!)

18. **Connectivity** 설정 아래에서 다음을 구성합니다.

- **Virtual Private Cloud (VPC):** _Lab VPC_

19. **VPC security group** 아래에서 다음을 구성합니다.

- <i class="far fa-dot-circle"></i> **Choose existing** 을 선택합니다.
- **DB Security Group** 라고 파란색으로 표시된 이름을 선택하세요.
- 반드시 _default X_ 를 제거 하십시오 ( X 모양 클릭)

20. <i class="fas fa-caret-right"></i> **Additional configuration** 를 클릭 하여 확장하고 다음을 구성합니다.

- **Initial database name:** `lab`
- <i class="far fa-square"></i> **Enable automatic backups** 을 선택 해제합니다.
- <i class="far fa-square"></i> **Enable Enhanced monitoring** 을 선택 해제합니다.

<i class="fas fa-comment"></i> 이렇게 하면 백업이 해제되어 일반적인 운영환경에서 권장되지 않지만, 실습을 위한 데이터베이스의 구축 속도가 빨라집니다.

21. 화면 맨아래에 <span style="background-color:#ec7211; font-weight:bold; font-size:90%; color:white; position:relative; top:-1px; padding-top:3px; padding-bottom:3px; padding-left:10px; padding-right:10px;white-space: nowrap">Create database</span> 를 클릭합니다.

이제 Dababase 생성이 시작됩니다.

<i class="fas fa-comment"></i> "not authorized to perform: iam:CreateRole" 오류 메시지가 표시되면 이전 단계에서 Enable Enhanced monitoring 을 선택 취소했는지 확인합니다.

22. **lab-db** 링크를 클릭합니다.

이제 **약 4분** 정도 기다려야 데이터베이스를 사용할 수 있습니다. 이 프로세스는 실제로 데이터베이스를 2개의 가용 영역에 배포합니다.

<i class="fas fa-info-circle"></i> 기다리는 동안 [Amazon RDS FAQ](https://aws.amazon.com/rds/faqs/)를 보는 것도 좋을 것입니다.

23. **DB instance Status**가 *Available* 또는 *Modifying*이 될 때까지 기다립니다.

<i class="fas fa-comment"></i> 2분마다 웹 페이지를 새로 고쳐 상태를 업데이트할 수 있습니다.

24. 화면아래 **Connectivity** 섹션으로 스크롤하여 **Endpoint** 필드를 복사합니다.

이 값은 _lab-db.cggq8lhnxvnv.us-west-2.rds.amazonaws.com_ 과 비슷한 형태입니다.

25. 텍스트 편집기에 엔드포인트 값을 붙여넣습니다. 나중에 실습에서 이 값을 사용하게 됩니다.

## ~~과제 4: 데이터베이스와 상호 작용~~ (실습 불가)

~~이 작업에서는 웹 서버에서 실행 중인 웹 응용프로그램에 접근해 데이터베이스를 사용하도록 구성합니다.~~

~~26. 현재 읽고 있는 가이드 페이지 왼쪽에 표시된 **WebServer** IP 주소를 복사하십시오.~~

~~27. 새로운 웹 브라우저 탭을 열고, IP 주소를 붙여넣은 다음 Enter를 누릅니다.~~

~~웹 애플리케이션이 표시되어 EC2 인스턴스에 대한 정보를 보여줍니다.~~

~~28. 페이지 상단의 **RDS** 링크를 클릭합니다.~~

~~이제 애플리케이션을 구성하여 데이터베이스에 연결합니다.~~

~~29. 다음 설정을 구성합니다.~~

- ~~**Endpoint**: 앞서 텍스트 편집기에 복사해 놓은 Database의 엔드포인트를 붙여 넣습니다.~~
- ~~**Database**: `lab`~~
- ~~**Username**: `lab_admin`~~
- ~~**Password**: `lab-password`~~
- ~~**Submit** 을 클릭합니다.~~

~~애플리케이션이 데이터베이스에 정보를 복사하는 명령을 실행 중이라고 설명하는 메시지가 표시되고, 몇 초 후 애플리케이션에 **Address Book**이 표시됩니다.~~

~~Address Book 애플리케이션은 RDS 데이터베이스를 사용하여 정보를 저장합니다.~~

30. ~~연락처를 추가, 편집. 제거하면서 웹 애플리케이션을 테스트해 봅니다.~~

~~데이터가 데이터베이스에 저장 관리 되고 자동으로 두 번째 가용 영역으로 복제됩니다.~~

## 실습 완료

<i class="icon-flag-checkered"></i>축하합니다! 웹 사이트를 위한 관계형 데이터 스토어를 성공적으로 구성했습니다.
