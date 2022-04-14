현재 사내 서비스의 공통 환경설정 처리는 AWS의 Secrets Manager를 이용하고 있습니다. <br>
해당 서비스는 서버 재기동없이는 refresh가 안된다는점과 로컬 환경에서 각 pc마다 aws client 설정을 해줘야 한다는 점, 사용사례 및 예제가 많지 않다는 점에서 Spring Cloud로 전환을 고려하며 간단히 해본 설정을 정리해보려 합니다.

<br>

version
> SpringBoot 2.5.5

<br>

구성은 repository, config, application 3개의 서버로 구분합니다. <br>

<h3>1.repository</h3>
<h4> 1.1. profiles 별 설정 </h4>

repository 는 github 에 저장하여 사용할 것입니다. <br>

> ex) config-local.yml, config-dev.yml, config-prod.yml
~~~yml
test:
  message: success
~~~

프로젝트의 루트 경로에 yml 설정 파일들을 생성해줍니다. <br>
서버 환경에 맞게 적용하기 위해 {applicationName}-{profiles}.yml 형태의 이름을 작성해줍니다. <br>

<h3>2.config</h3>
<h4> 2.1. gradle 설정 </h4>

> ex) build.gradle

~~~gradle
..생략..
ext {
	set('springCloudVersion', "2020.0.3")
}

dependencies {
	implementation 'org.springframework.cloud:spring-cloud-config-server'
}

dependencyManagement {
	imports {
		mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
	}
}
..생략..
~~~

Spring Cloud 버전은 Spring Boot 버전마다 상이하므로 <a href="https://spring.io/projects/spring-cloud">여기</a> 에서 자신의 Boot 버전에 맞는 것을 찾아야합니다. <br><br>


<h4> 2.2. git 정보 및 refresh 설정 </h4>

> ex) application.yml

~~~yml
spring:
  cloud:
    config:
      server:
        git:
          default-label: master
          uri: https://github.com/{저장소 경로} #(1)

server:
  port: 8888 #(2)

#(3)
management:
  endpoints:
    web:
      exposure:
        include: refresh
  endpoint:
    shutdown:
      enabled: true
~~~
(1): repository 의 git 주소 <br>
(2): config 서버의 port <br>
(3): application 의 서버 재시작 없이 설정 값을 refresh 하기 위한 설정<br>

서버 시작 후 http://localhost:8888/config/local 을 접속해보면 정상 조회된 결과를 볼 수 있습니다. <br>
http://localhost:8888/{applicationName}/{profiles} 형태 <br>

<h3>3.application</h3>
<h4> 3.1. gradle 설정 </h4>

> ex) build.gradle

~~~gradle
..생략..
ext {
	set('springCloudVersion', "2020.0.3")
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
	implementation 'org.springframework.boot:spring-boot-starter-web'
	implementation 'org.springframework.boot:spring-boot-starter-actuator'

	implementation 'org.springframework.cloud:spring-cloud-starter-bootstrap'
	implementation 'org.springframework.cloud:spring-cloud-starter-config'
	
	implementation 'org.projectlombok:lombok'
}

dependencyManagement {
	imports {
		mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
	}
}
..생략..
~~~

cloud config 및 테스트를 위한 web과 refresh를 위한 actuator도 추가해줍니다. <br><br>

<h4> 3.2. config 관련 설정파일 </h4>

> ex) bootstrap-local.yml, bootstrap-dev.yml, bootstrap-prod.yml

~~~yml
spring:
  application:
    name: config #(1)
  cloud:
    config:
      uri: http://localhost:8888 #(2)
management:
  endpoints:
    web:
      exposure:
        include: refresh #(3)
~~~

bootstrap-{profiles}.yml 의 파일을 root 경로에 추가해줍니다. <br>

(1): repository 에서 읽어올 설정파일 이름과 동일 <br>
(2): config 서버의 uri 정보 <br>
(3): 서버 재시작 없이 refresh 하기 위한 설정 <br><br>


<h4> 3.3. 테스트 서비스 & 컨트롤러 추가</h4>

> ex) ConfigService.java
 
~~~java
@Service
@RefreshScope //(1)
@Slf4j
public class ConfigService {
    @Value("${test.message}") // (2)
    private String message;

    public void printMessage(){
        log.info("message :: {}", message);
    }
}
~~~
(1): 서버 재시작 없이 refresh 하기 위한 어노테이션 <br>
(2): application.yml 의 값을 읽을 때와 같이 @Value 사용
<br><br>

> ex) ConfigController.java

~~~java
@RestController
@RequiredArgsConstructor
public class ConfigController {

    private final ConfigService configService;

    @GetMapping("")
    public void index(){
        configService.printMessage();
    }
}
~~~

http://localhost:8080 에 접속해보면 repository 에서 profiles 별로 설정했던 test.message 값이 출력되는 것을 확인할 수 있습니다.

<h4> 3.4. refresh</h4>
spring actuator 를 이용해 application 의 재시작 없이 변경된 설정 값을 적용해줍니다. <br>
http://localhost:8080/actuator/refresh 를 post로 호출해준 후
http://localhost:8080 에 다시 접속해보면 변경된 설정값을 확인할 수 있습니다.