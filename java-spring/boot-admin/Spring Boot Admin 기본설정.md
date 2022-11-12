actuator 를 활용한 관리 & 모니터링을 위해 사용하는 Spring Boot Admin 기본 설정입니다.
<br>

<h3>1. Spring Boot Admin Server</h3>

<h4>1.1. 의존성 추가</h4>
> ex) build.gradle

~~~gradle
plugins {
    id 'org.springframework.boot' version '2.7.4'
    id 'io.spring.dependency-management' version '1.0.14.RELEASE'
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter'
    implementation 'de.codecentric:spring-boot-admin-starter-server:2.7.5' // (1)
}
~~~
(1): spring boot server 의존성을 추가 <br>
Spring Boot 버전은 2.7.4 입니다.

<br>

<h4>1.2. 설정</h4>
> ex) application.yml

~~~yml
server:
  port: 8085
  
spring:
  boot:
    admin:
      ui:
        public-url: # 
~~~

<br>

> ex) Application.java

~~~java
@EnableAdminServer //(1)
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
~~~
(1): @EnableAdminServer 어노테이션 추가

<h3>2. Spring Boot Admin Client</h3>

<h4>2.1. 의존성 추가</h4>

~~~gradle
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter'
	implementation 'org.springframework.boot:spring-boot-starter-web'

	implementation 'de.codecentric:spring-boot-admin-starter-client:2.7.5'
	implementation 'org.springframework.boot:spring-boot-starter-actuator'
}
~~~

actuator 와 spring boot admin client 의존성 추가
<br>

<h4>2.2. 설정</h4>

~~~yml
spring:
  application:
    name: admin-client
  boot:
    admin:
      client:
        url: http://localhost:8085 #(1)

management:
  endpoints:
    web:
      exposure:
        include: '*' #(2)
  endpoint:
    health:
      show-details: always #(3)
~~~

(1): Spring Boot Admin Server url <br>
(2): 사용할 actuator 지정 <br>
(3): 상세내역을 항상 보기위한 always 설정 (default: never) <br>

이 후 두 서버를 실행만 시키면 http://localhost:8085 에서 관리 화면을 볼 수 있습니다.
