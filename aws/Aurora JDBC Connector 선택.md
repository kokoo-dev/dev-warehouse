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
~~아래와 같은 이유로 **AWS MySQL JDBC** 사용~~ <br>

- MariaDB JDBC 의 메이저 버전 업데이트에서 Aurora 지원 중단
- <a href="https://aws.amazon.com/ko/blogs/database/using-the-mariadb-jdbc-driver-with-amazon-aurora-with-mysql-compatibility/">AWS Database 공식 블로그</a>에서 AWS JDBC 사용 권장
  - <img src="/img/aws-database-blog.png" width="400px">
- 수동 설정을 통해 Reader Instance 의 Scale Out 에 따라 Reader 쪽 Connection Pool 만 늘리도록 변경 가능
  
> 2023.03.15 수정

수정날짜(2023.03.15) 기준 최신 버전 (1.1.4) 기준으로 확인했을 때 Read Cluster 에 Connection 만 맺어지고, <br>
실제 쿼리는 Write Cluster 로 보내지는 것을 확인. <br>
라이브러리 내에서 왜 connection url 이 변경 되는 것인지 확인 필요 <br>
  
<h3>Spring 에서 Read/Write 분리 수동 설정 예시</h3>

> ex) build.gradle

~~~gradle
dependencies {
    implementation 'software.aws.rds:aws-mysql-jdbc:1.1.4'
}
~~~
- <a href="https://github.com/awslabs/aws-mysql-jdbc#as-a-gradle-dependency">https://github.com/awslabs/aws-mysql-jdbc#as-a-gradle-dependency</a>

<br>

> ex) application.yml

~~~yml
mysql:
  driver-class-name: software.aws.rds.jdbc.mysql.Driver
  username: {id}
  password: {password}
  write:
    name: write
    # url: jdbc:mysql:aws://{Write Cluster Endpoint}
    url: jdbc:mysql://{Write Cluster Endpoint}
  read:
    name: read
    # url: jdbc:mysql:aws://{Read Cluster Endpoint}
    url: jdbc:mysql://{Read Cluster Endpoint}
~~~

- write/read 의 url 분리
- ~~url 은 jdbc:mysql:aws:// 로 설정 (aws 빼먹을 경우 일반 MySQL JDBC 로 사용됨)~~
  - 라이브러리 현재 버전 (1.1.2 ~ 1.1.4 모두 해당) 에서 Reader 에 Connection 만 가지고 실제 쿼리는 Writer 로 전송
  - aws-jdbc-mysql 은 mysql connector/j 를 기반으로 만든 것이라 build.gradle 의 의존성이나 driver-class-name 은 유지하고 jdbc-url 의 연결 방식만 변경
  - Failover vs Reader/Writer 분리의 문제였으나, Failover 는 비교적 흔치 않은 상황이고 Reader Cluster 로 부하를 분산시키는게 DB 안전성으로 볼 때 더 중요하기에 mysql 커넥터로 결정
    - https://github.com/awslabs/aws-mysql-jdbc 버전 업데이트 주기적으로 확인 필요
- 반드시 Instance Endpoint 가 아닌 Cluster Endpoint 사용

<br>

> ex) ReplicationDataSourceProperties.java

~~~java
@Getter
@Setter
@Configuration
@ConfigurationProperties(prefix = "mysql")
public class ReplicationDataSourceProperties {
    
    private String driverClassName;
    private String username;
    private String password;
    private Write write;
    private Read read;
    
    @Getter
    @Setter
    public static class Write {
        private String name;
        private String url;
    }
    
    @Getter
    @Setter
    public static class Read {
        private String name;
        private String url;
    }
}
~~~

- application.yml 의 mysql 설정을 바인딩

<br>

> ex) ReplicationRoutingDataSource.java

~~~java
@Builder
@AllArgsConstructor(access = AccessLevel.PRIVATE)
public class ReplicationRoutingDataSource extends AbstractRoutingDataSource {
    
    private String writeLookUpKey;
    private String readLookUpKey;

    @Override
    public void setTargetDataSources(Map<Object, Object> targetDataSources) {
        super.setTargetDataSources(targetDataSources);
    }
    
    @Override
    public Object determineCurrentLookupKey() {
        return TransactionSynchronizationManager.isCurrentTransactionReadOnly() ? readLookUpKey : writeLookUpKey;
    }
}
~~~

- AbstractRoutingDataSource 를 상속받아 Transaction 의 ReadOnly 여부에 따라 다른 DataSource 를 사용하도록 구현

<br>

> ex) ReplicationRoutingDataSource.java

~~~java
@Configuration
@RequiredArgsConstructor
public class ReplicationDataSourceConfig {
    
    private final ReplicationDataSourceProperties replicationDataSourceProperties;
    
    @Bean
    public DataSource routingDataSource() {
        Write write = replicationDataSourceProperties.getWrite();
        Read read = replicationDataSourceProperties.getRead();
        
        DataSource writeDataSource = createDataSource(write.getUrl());
        DataSource readDataSource = createDataSource(read.getUrl());
        
        Map<Object, Object> dataSourceMap = new HashMap<>();
        dataSourceMap.put(write.getName(), writeDataSource);
        dataSourceMap.put(read.getName(), readDataSource);
        
        ReplicationRoutingDataSource replicationRoutingDataSource = ReplicationRoutingDataSource.builder()
            .writeLookUpKey(write.getName())
            .readLookUpKey(read.getName())
            .build();
        replicationRoutingDataSource.setDefaultTargetDataSource(writeDataSource); // (1)
        replicationRoutingDataSource.setTargetDataSources(dataSourceMap);
        replicationRoutingDataSource.afterPropertiesSet();
        
        return new LazyConnectionDataSourceProxy(replicationRoutingDataSource);
    }
    
    private DataSource createDataSource(String url) {
        HikariDataSource hikariDataSource = new HikariDataSource();
        hikariDataSource.setDriverClassName(replicationDataSourceProperties.getDriverClassName());
        hikariDataSource.setJdbcUrl(url);
        hikariDataSource.setUsername(replicationDataSourceProperties.getUsername);
        hikariDataSource.setPassword(replicationDataSourceProperties.getPassword);
        // (2)
        return hikariDataSource;
    }
}
~~~

- (1): Primary 인 Write Cluster 가 Default DataSource 가 되도록 설정
- (2): Timeout 이나 Connection Pool Size 등 추가 설정

@Transactional(readOnly=true|false) 설정만 해주면 DataSource 분기 처리가 됩니다.