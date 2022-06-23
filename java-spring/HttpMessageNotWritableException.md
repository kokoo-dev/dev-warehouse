RestController에서 Gson으로 응답 시 해당 Exception이 발생합니다. <br/>
스프링에서는 기본적으로 Jackson 라이브러리를 이용하기에 Gson을 이용하면 충돌이 발생한다고 합니다. <br/>

이는 아래와 같은 설정값으로 해결할 수 있습니다.<br/>
> ex) application.yml
~~~yml
spring:
  mvc:
    converters:
      preferred-json-mapper: gson
~~~
