로컬에서 테스트할 때는 문제가 없다가 테스트 서버에서 해보니 redirect하는 구간에서 에러가 발생하고 있었습니다. <br>
테스트 서버의 프로토콜은 https인데 자꾸 http로 리다이렉트 시키고 있던 것이었습니다. <br>
Spring Boot의 내장 톰캣을 사용하는 경우 https -> https, http -> http로 리다이렉트 하기 위한 방법은 설정파일 변경으로 간단히 처리됩니다. <br>

~~~yml
server:
  tomcat:
    use-relative-redirects: true
~~~
