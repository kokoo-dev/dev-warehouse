logstash로 외부 mysql에 있는 데이터를 같은 서버의 ElasticSearch로 옮길 수 있게 설정하는 방법입니다. <br>

임시 테이블 구조
~~~sql
CREATE TABLE IF NOT EXISTS `shop` (
  `shop_no` int NOT NULL AUTO_INCREMENT,
  `shop_name` varchar(50) NOT NULL,
  `del_yn` varchar(1) NOT NULL,
  `reg_date` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `upd_date` datetime NOT NULL,
  PRIMARY KEY (`shop_no`)
)
~~~

1. mysql connector 설치
~~~sh
sudo yum install -y mysql-connector-java
~~~

<br>

2.logstash conf파일 생성
~~~sh
vi /home/ec2-user/mysql.conf
~~~

> mysql.conf 내용

~~~conf
input{
    jdbc{
        jdbc_validate_connection => true
        jdbc_driver_class => "com.mysql.jdbc.Driver"
        jdbc_driver_library => "/usr/share/java/mysql-connector-java.jar" # (1)
        jdbc_connection_string => "jdbc:mysql://{DB 호스트}:3306/{DB명}?{DB옵션}"
        jdbc_user => "{DB계정}"
        jdbc_password => "{DB패스워드}"
        lowercase_column_names => false # (2)
        statement => "select shop_no as shopNo, shop_name as shopName from shop"
    }
}
output{
    elasticsearch{
        hosts => "http://localhost:9200"
        index => "shop" # (3)
        document_id => "%{shopNo}" # (4)
    }
}
~~~

(1): yum을 통해 설치된 connector의 전체 경로 <br>
(2): 카멜표기법으로 컬럼을 조회할 때 대소문자 구분을 해줌 <br>
(3): 인덱스 이름, 없으면 생성됨 <br>
(4): document id로 사용될 값으로 %{shopNo}는 input에서 조회되는 shopNo값을 아이디로 사용하겠다는 의미 <br>

<br>

3. logstash실행

~~~sh
sudo /usr/share/logstash/bin/logstash -f /home/ec2-user/mysql.conf
~~~

서비스로 등록했다면 서비스를 실행해줍니다. <br>

이 후 curl -X GET 'localhost:9200/shop/_search' 로 확인해봅니다.

 <br> <br>
 
 ++ 주기적으로 중복되지 않는 데이터만 동기화하는 방법입니다.<br>
<a href="https://www.elastic.co/kr/blog/how-to-keep-elasticsearch-synchronized-with-a-relational-database-using-logstash">여기</a>에 설명이 잘되어있어 참고하였습니다.<br>

<br>

위에 작성했던 conf에 내용을 추가 및 변경해줍니다. <br>

~~~conf
input{
    jdbc{
        jdbc_validate_connection => true
        jdbc_driver_class => "com.mysql.jdbc.Driver"
        jdbc_driver_library => "/usr/share/java/mysql-connector-java.jar"
        jdbc_connection_string => "jdbc:mysql://{DB 호스트}:3306/{DB명}?{DB옵션}"
        jdbc_user => "{DB계정}"
        jdbc_password => "{DB패스워드}"
        lowercase_column_names => false
        jdbc_paging_enabled => true # (1)
        tracking_column => "unix_ts_in_secs" # (2)
        tracking_column_type => "numeric" # (3)
        use_column_value => true # (4)
        statement => "select shop_no as shopNo, shop_name as shopName
                            , unix_timestamp(upd_dt) AS unix_ts_in_secs
                      from shop
                      where (unix_timestamp(a.upd_dt) > :sql_last_value and a.upd_dt < now())
                      order by a.upd_dt asc" # (5)
        schedule => "30 * * * * *" # (6)
    }
}
filter {
  mutate {
    remove_field => ["id", "@version", "unix_ts_in_secs"] # (7)
  }
}
output{
    elasticsearch{
        hosts => "http://localhost:9200"
        index => "shop"
        document_id => "%{shopNo}"
    }
}
~~~
(1) jdbc_paging_enabled: 데이터 조회시 페이징 처리를 할 것인지 지정하며 jdbc_page_size설정으로 페이징 사이즈 설정 가능 <br>
(2) tracking_column: 마지막으로 mysql에서 읽은 문서를 추적할 때 쓰일 필드<br>
(3) tracking_column_type: tracking_column의 타입으로 numeric과 timestamp가 있음<br>
(4) use_column_value: tracking_column값을 :sql_last_value로 정의<br>
(5) sql_last_value: shop테이블의 업데이트 날짜와 tracking_column의 값으로 변경된 필드만 조회<br>
(6) schedule: 데이터를 조회할 주기를 크론 형식으로 설정<br>
(7) remove_filed: []안에 값들이 새 필드로 생성되지 않도록 지정해주는 값 <br>

이 후 서비스로 등로해서 실행하면 schedule로 지정된 주기마다 업데이트된 값만 조회해서 ElasticSearch에 갱신하는 것을 확인할 수 있습니다. <br>

<a href="https://www.elastic.co/guide/en/logstash/current/plugins-inputs-jdbc.html#plugins-inputs-jdbc-use_column_value">logstash - jdbc 설정값에 대한 설명</a> 참고!




