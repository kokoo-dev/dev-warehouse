Spring Cloud Config 에서 일괄적으로 Refresh 를 사용하기 위한 Spring Cloud Bus 적용 방법입니다. <br>
Spring Cloud Config 설정이 다 되어 있다면 Config 서버 수정, Application 서버 수정, 메세지 브로커 설치만 하면 됩니다. <br>

서버 설정 관리의 특성상 빌드할 때나 리프레쉬 때만 사용하기에 메세지 브로커도 경량인 RabbitMQ 를 선택해줍니다. <br>


### 1.Config, Application 공통
1.1. 의존성 추가

> ex) build.gradle

~~~gradle
implementation 'org.springframework.boot:spring-boot-starter-actuator'
implementation 'org.springframework.cloud:spring-cloud-starter-bus-amqp'
~~~

1.2. RabbitMQ 설정

> ex) application.yml

~~~yml
spring:
  rabbitmq:
      host: 127.0.0.1
      port: 5672
      username: guest
      password: guest
management:
  endpoints:
    web:
      exposure:
        include: busrefresh #(1)
~~~

rabbitmq 접속 정보는 두 서버 공통으로 application.yml 에 작성해주며, <br>
(1) 은 refresh -> busrefresh 로 변경된 값으로 Config 서버에서는 application.yml 에 작성해주고, <br>
Applcation 서버에는 bootstrap.yml 에 작성해줍니다. <br>
이 후 rabbitMQ -> Config -> Application 순서대로 서버를 시작해주면 되고, <br>
기존에 Application 서버별 /refresh 로 요청 하던 방식을 Config 서버에 /actuator/busrefresh 로 요청하면 일괄 적용 됩니다. <br>

ex) http://127.0.0.1:8888/actuator/busrefresh