AWS Aurora 의 경우 1개의 Writer 인스턴스와 0~15개의 Reader 인스턴스로 구성하여 부하를 분산 시킬 수 있고, 뛰어난 Failover 성능을 보여주는 장점이 있습니다. <br>
Spring 에서 JDBC 를 선택할 때 위 두가지 장점을 살릴 수 있는 것은 무엇인지 **MySQL vs MariaDB vs AWS** JDBC Driver 이 세가지를 비교해보겠습니다. <br>

<h3>특이사항</h3>

- MariaDB JDBC
  - Major 버전업에 따른 AWS Aurora 미지원 (2.x -> 3.x)
    - 따라서 비교도 2.x 와 3.x 로 나눠서 진행
    - 2.x 에서는 jdbc url 을 'jdbc:mariadb:aurora://{writer cluster endpoint},{reader cluster endpoint}' 만으로 설정 완료
- AWS MySQL JDBC
  - Read/Write 분리 미지원
    - <a href="https://github.com/awslabs/aws-mysql-jdbc#read-write-splitting">aws-mysql-jdbc github</a> 작성일 기준 미지원
      - <img src="/img/aws-jdbc-splitting.png" width="400px">
    - github 에 read-write splitting 관련 issue 와 branch 가 있는 것으로 볼 때 추후 버전에 지원 할 것으로 기대

<h3>Test Version</h3>

- Spring Boot: 2.7.x
- MySQL JDBC: 8.0.31
- MariaDB JDBC v2.x: 2.7.5
- MariaDB JDBC v3.x: 3.1.0
- AWS JDBC: 1.1.2

<h3>비교</h3>

|               | MySQL JDBC | MariaDB JDBC v2.x | MariaDB JDBC v3.x | AWS MySQL JDBC |
|:-------------:|:----------:|:-----------------:|:-----------------:|:--------------:|
| Read/Write 분리 | 수동 설정으로 가능 | jdbc url 설정만으로 가능 |    수동 설정으로 가능     |   수동 설정으로 가능   |
| Failover 시 HA |    미지원     |        지원         |        미지원        |       지원       |

- Read/Write 분리 수동 설정
  - LazyConnectionDataSourceProxy 를 이용해 DataSource 를 분기 처리
  
- Failover 시 HA
  - MySQL 이나 MariaDB v3.x 의 경우 Failover 시 write(insert/update/delete) 작업을 할 경우 SQL Error 1836-HY00-
    ER_READ_ONLY_MODE 발생, 즉 read-only DB 에 write 작업을 한 것
    - Replica(read-only) 가 Primary(read/write) 로 승격되면서 Cluster Endpoint DNS 의 CNAME 이 변경되는데 WAS 의 Connection
      Pool 은 강등(read/write → read-only)된 기존 Primary 의 인스턴스 정보를 가지고 있기 때문
    - 서버를 재시작해야 올바른 인스턴스로 연결되기에 애플리케이션에서 HA 구성이 불가
  - MariaDB v2.x 나 AWS 의 경우 내부 장애조치 덕분에 올바른 인스턴스로 연결해 곧바로 정상적인 read/write 가능

<h3>결과</h3>

앞서 목표로 삼은 AWS Aurora 의 Read/Write 의 접속 분리, Failover 시 HA 구성 조건을 만족하는 Driver 는 MariaDB JDBC v2.x, AWS MySQL JDBC 2개 <br>
아래와 같은 이유로 **AWS MySQL JDBC** 사용
- MariaDB JDBC 의 메이저 버전 업데이트에서 Aurora 지원 중단
- <a href="https://aws.amazon.com/ko/blogs/database/using-the-mariadb-jdbc-driver-with-amazon-aurora-with-mysql-compatibility/">AWS Database 공식 블로그</a>에서 AWS JDBC 사용 권장
  - <img src="/img/aws-database-blog.png" width="400px">
    