Controller에서 Date 포맷을 지정할 수 있습니다. <br>
파라미터별 설정과 모든 파라미터에 일괄적용하는 방법 두가지를 설명합니다. <br>


1. 일괄적용, @InitBinder
~~~java
@RestController
public class TestController{
    @InitBinder
    protected void initBinder(WebDataBinder binder){
        DateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        binder.registerCustomEditor(Date.class, new CustomDateEditor(dateFormat, true));
    }
}
~~~

@InitBinder는 컨트롤러에서 바인딩이나 검증을 할 때 사용하는 어노테이션입니다. <br><br>


2. 파라미터별 적용
~~~java
@RestController
public class TestController{
    @GetMapping("")
    public String index(TestDTO testDTO){
        return "";
    }
}
~~~
~~~java
@Getter
@Setter
public class TestDTO{
    @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private Date date;
    @DateTimeFormat(pattern = "yyyy-MM-dd")
    private Date formatDate;
}
~~~
@DateTimeFormat로 각자 다른 포맷을 지정해줄 수 있습니다.
