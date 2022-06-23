세션은 기본적으로 WAS에 저장되기에 서버가 분산환경일 경우 공유 세션 스토리지가 필요하게 됩니다. <br>
WAS끼리 세션을 클러스터링 할 수도 있고 RDB를 스토리지로 쓸 수 있지만, 이 방법들에 비해 성능상 뛰어난 메모리 DB인 Redis로 스프링에서 설정하는 방법을 설명합니다. <br>

레디스 의존성 추가 및 client 연결 설정은 <a href="https://github.com/kokoo-dev/dev-warehouse/blob/main/redis/SpringBoot%EC%97%90%EC%84%9C%20%EC%82%AC%EC%9A%A9.md">여기</a> 를
참고하면 됩니다. <br>

방법은 두가지 입니다.<br>
1. 어플리케이션 일괄 설정 <br>
2. 환경 별 설정 <br>

<h4>1. 어플리케이션 일괄 설정</h4>

~~~java
@SpringBootApplication
@EnableRedisHttpSession //(1)
public class TestApplication {

    public static void main(String[] args) {
        SpringApplication.run(TestApplication.class, args);
    }
}
~~~

(1) EnableRedisHttpSession 어노테이션 하나로 설정은 끝나게 됩니다.<br>
로컬에서는 분산 환경일 경우도 없고 테스트용도가 아니라면 세션저장소를 사용하지 않아도 됩니다. <br>
실무에서 위 처럼 일괄 설정을 하고 스펙이 낮은 테스트용 Redis를 사용하다가 웹 응답속도는 느려지고 이유를 찾기까지 오래 헤맨적이 있습니다.. <br>

<h4>2. 환경 별 설정</h4>

> ex) application.yml
~~~yml
spring:
  session:
    #store-type: none 사용안함
    store-type: redis #redis사용
~~~

yml 설정파일에 spring.session.store-type을 지정해줍니다. 이때 EnableRedisHttpSession 어노테이션은 사용하지 않아야 합니다. <br>
속성 값으로는 none, redis, hazelcast, jdbc, mongodb 가 있기에 원하는 방식을 환경에 따라 적용하시면 됩니다.



