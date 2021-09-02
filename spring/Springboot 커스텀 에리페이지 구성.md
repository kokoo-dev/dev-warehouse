WAS에서 제공되는 에러페이지 대신 커스텀 에러페이지를 구성하는 방법입니다. <br/>
Spring boot에서는 /error 맵핑을 통해 커스텀 에러화면을 구성합니다.

> ex) CustomErrorController.java

~~~java
@Controller
public class CustomErrorController implements ErrorController {
    @RequestMapping(value = "/error")
    public String handleError(Model model, HttpServletRequest request){
        int status = (int) request.getAttribute(RequestDispatcher.ERROR_STATUS_CODE);

        HttpStatus httpStatus = HttpStatus.resolve(status);
        String reason = httpStatus.getReasonPhrase();

        model.addAttribute("status", status);
        model.addAttribute("reason", reason);

        return "/error"; //error.html
    }
}
~~~
