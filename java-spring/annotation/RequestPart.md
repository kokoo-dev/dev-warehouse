Controller에서 Multipart를 바인딩 하기 위해 사용합니다. <br/>

> ex) TestController.java
~~~java
@Controller
public class TestController{
    @PostMapping("/upload")
    @ResponseBody
    public String upload(@RequestPart("uploadFile") List<MultipartFile> files ){
        return "";
    }
}
~~~

위와 같이 구성하면 되나 View단에서 멀티파트 설정임에도 파일을 전송하지 않을 경우, <br/>
Required request part '' is not present 오류가 발생할 수 있습니다. <br/>
이 경우 required = false 를 지정해주면 해결됩니다.

> ex) TestController.java
~~~java
@Controller
public class TestController{
    @PostMapping("/upload")
    @ResponseBody
    public String upload(@RequestPart(value = "uploadFile", required = false) List<MultipartFile> files ){
        return "";
    }
}
~~~
