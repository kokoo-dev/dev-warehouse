컨트롤러에서 발생한 Exception을 일괄적으로 처리할 수 있는 방법입니다. <br/>

서버에서 에러처리이므로 당연히 400번대 에러는 처리할 수 없습니다. 400번대 에러는 <a href="https://github.com/kokoo-dev/dev-warehouse/blob/main/spring/Springboot%20%EC%BB%A4%EC%8A%A4%ED%85%80%20%EC%97%90%EB%A6%AC%ED%8E%98%EC%9D%B4%EC%A7%80%20%EA%B5%AC%EC%84%B1.md">커스텀 에러처리</a>
를 참고하면 됩니다.

> CustomExceptionHandler.java
~~~java
@Slf4j
// @ControllerAdvice(annotations = Controller.class) // (1)
@RestControllerAdvice(annotations = RestController.class) // (2)
public class CustomExceptionHandler {

    @ExceptionHandler(Exception.class) // (3)
    protected String exceptionHandler(Exception e, HttpServletResponse response) {
        log.error("Exception : ", e);
        
        response.setStatus(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);

        return "Exception!!";
    }

    @ExceptionHandler(CustomException.class) // (4)
    protected String customExceptionHandler(Exception e, HttpServletResponse response) {
        log.error("CustomException : ", e);
        
        response.setStatus(HttpServletResponse.SC_INTERNAL_SERVER_ERROR);

        return "CustomException!!";
    }
}
~~~

@ControllerAdvice나 @RestControllerAdvice를 추가해주면 됩니다. (@ResponseBody차이)<br/>
저같은 경우는 한 프로젝트에 Controller와 RestController가 섞여있었는데 두 어노테이션만으로는 구분돼서 처리가 안됐습니다. <br/>
Controller에서는 view를 RestController에서는 JSON을 리턴해줘야 하는 상황이었던것입니다.<br/>
여려가지 테스트 결과 Controller에 @ResponseBody로 정의해놨던 메소드들을 RestController로 다 옮긴 후 <br/>
(1), (2)와 같이 annotations = Controller.class, annotations = RestController.class를 지정해주어 구분하였습니다. (물론 java파일도 두개로 분리)<br/>

<br/><br/>
추가로 (3), (4)와 같이 Exception별로 지정해서 처리를 할 수 있습니다.
