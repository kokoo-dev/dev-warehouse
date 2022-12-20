Spring Cloud Config 서버를 리눅스 기반 서버에 올려놓고 사용하다가 RepositoryNotFoundException: repository not found 메세지와 함께 갑자기 동작하지 않는 경우가 발생하곤 합니다. <br>
기본 설정의 경우 /tmp/config-repo-<randomId> 위치에 클론을 받게 되는데 일부 운영체제에서는 해당 위치에 폴더들을 주기적으로 삭제한다고 합니다. <br>
이는 디렉토리를 강제로 지정해줌으로써 대응이 가능합니다. <br>
  
> ex) application.yml
 
~~~yml
spring:
  cloud:
    config:
      server:
        git:
          basedir: ./config-repo
~~~

위와 같이 spring.cloud.config.server.git.basedir 에 상대경로 또는 절대경로를 입력해줍니다. <br>
  
해당 내용은 <a href="https://github.com/spring-cloud/spring-cloud-config/blob/main/docs/src/main/asciidoc/spring-cloud-config.adoc#version-control-backend-filesystem-use">spring cloud config github </a> 에서도 확인이 가능합니다. 
