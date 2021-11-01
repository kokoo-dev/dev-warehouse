해당 어노테이션이 달린 빈의 범위를 지정해줍니다. <br/>
스프링에서 빈은 기본적으로 싱글톤으로 생성되며 아래와 같이 값에 따라 그 범위를 변경할 수 있습니다.<br/>

- singleton: 한개의 bean 생성 (default)
- prototype: bean 요청마다 생성
- request: Http request 마다 생성
- session: Http session 마다 생성
- global-session: 포틀릿의 Global Session 마다 생성, 이 부분은 더 알아봐야할 것 같습니다.

> ex) @Scope("singleton")
