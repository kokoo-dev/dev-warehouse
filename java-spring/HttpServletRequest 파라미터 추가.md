## HttpServletRequest 파라미터 추가
SpringBoot 필터 예제를 작성하다 파라미터를 변경해야 하나 setParameter() 함수가 별도로 없어서 방법을 찾던 중 <br/>
HttpServletRequestWrapper 클래스를 상속받아 HttpServletRequest를 복사하는 방법을 찾았다.
<br/><br/><br/>


1.HttpServletRequestWrapper 상속
> ex) ModifyRequest.java
~~~java
public class ModifyRequest extends HttpServletRequestWrapper { // HttpServletRequestWrapper 상속

      public ModifyRequest(HttpServletRequest request) {
            super(request); // 부모 클래스 생성자 호출
      }
      
      //TODO 메소드 추가
}
~~~

<br/><br/><br/>

2.ModifyRequest 사용
> ex) MyFilter.java
~~~java
public class MyFilter implements Filter {
      @Override
      public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
            if(request instanceof HttpServletRequest){
                  ModifyRequest modifyRequest = new ModifyRequest((HttpServletRequest) request);
                  modifyRequest.setParameter("name","value");
                  
                  chain.doFilter(modifyRequest, response);
            } else {
                  chain.doFilter(request, response);
            }
      }
}
~~~
<br/><br/><br/>

참고 사이트 : https://sunghs.tistory.com/64


나의 소스 : https://bit.ly/3uSGt7K
