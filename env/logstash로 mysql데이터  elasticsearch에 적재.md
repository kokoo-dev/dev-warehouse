logstash로 외부 mysql에 있는 데이터를 같은 서버의 ElasticSearch로 옮길 수 있게 설정하는 방법입니다. <br>

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
