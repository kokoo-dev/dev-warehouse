> SpringBoot version: 2.5.x

<br>

1. HandlerInterceptor 구현
> ex) TestHandlerInterceptor.java

~~~java
public class TestHandlerInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        return HandlerInterceptor.super.preHandle(request, response, handler);
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        HandlerInterceptor.super.postHandle(request, response, handler, modelAndView);
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        HandlerInterceptor.super.afterCompletion(request, response, handler, ex);
    }
}
~~~

HandlerInterceptor를 상속받아 preHandle, postHandle, afterCompletion 메소드를 재정의하여 사용할 수 있습니다. <br>
- preHandle: 요청 처리 전
- postHandle: 요청 처리 후
- afterCompletion: view 렌더링 후

<br>

2. WebMvcConfigurer구현
> ex) WebMvcConfig.java

~~~java
@Configuration
public class WebMvcConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new TestHandlerInterceptor())
                .addPathPatterns("/*");
    }
}
~~~

인터셉터를 추가 및 url패턴을 설정해줍니다.
