## JUnit + MockMvc를 활용하여 컨트롤러 테스트하기

### 테스트 환경
 - SpringBoot: 2.4.5
 - Junit: 5

### 1. 디펜던시 추가
> ex) build.gradle
~~~gradle
dependencies {
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
~~~
해당 디펜던시는 Spring Assistant로 프로젝트를 생성할 경우 자동으로 추가가 되어있고, <br/>
spring-boot-starter-test에 테스트에 필요한 라이브러리가 포함돼서 추가 설정은 필요가 없습니다.

### 2. 컨트롤러 작성
> ex) SampleController.java
~~~java
@RestController
public class SampleController{
    Logger logger = LogManager.getLogger(SampleController.class);

    @GetMapping("/return")
    public String callReturn(String data){
        String returnData = data + "_return";
        
        return returnData;
    }
    
    @PostMapping("/file")
    public void callFile(MultipartFile file){
        logger.info("callWithOneFile fileName:: " + file.getOriginalFilename());
    }
}
~~~

### 3. 테스트 클래스 작성
> ex) SampleTests.java
~~~java
@ExtendWith(SpringExtension.class)
@WebMvcTest(controllers = SampleController.class)
public class SampleTests {

    @Autowired
    private MockMvc mockMvc;
        
    @DisplayName("404 Not found")
    @Test
    public void testNotFound() throws Exception {
        String url = "/404";
        
        ResultActions resultActions = mockMvc.perform(get(url)
            .accept(MediaType.TEXT_HTML));
        
        resultActions.andExpect(status().isNotFound());
        resultActions.andDo(print());
    }

    @DisplayName("return content 비교 성공")
    @Test
    public void testReturn() throws Exception {
        String param = "TestParam";
        String url = "/return";

        ResultActions resultActions = mockMvc.perform(get(url)
            .param("data", param)
            .accept(MediaType.TEXT_HTML));

        resultActions.andExpect(content().string(param + "_return"));
        resultActions.andDo(print());
    }

    @DisplayName("Multipart file 업로드 성공")
    @Test
    public void testMultipartFile() throws Exception{
        String url = "/file";
        MockMultipartFile mockMultipartFile = new MockMultipartFile("file", "test.txt", "text/plain", "Mock Test Text File".getBytes());

        ResultActions resultActions = mockMvc.perform(multipart(url)
            .file(mockMultipartFile));

        resultActions.andExpect(status().isOk());
        resultActions.andDo(print());
    }
}
~~~


 3.1. @ExtendWith, @WebMvcTest 어노테이션 추가


<br/>

### 4. MockMvc 메소드
  4.1. perform
   - 요청 전송 역할로 HTTP 메소드, 요청 파라미터 등을 지정


  4.2. andExpect
   - 응답을 검증하는 역할로 Http 상태코드나 리턴 값에 대한 검증


  4.3. andDo
   - 요청/응답에 대한 결과 확인

<br/>

### 5. MockMvcRequestBuilders 메소드
  5.1. get/post
   - HTTP 메소드 가능
 
 
  5.2. multipart
   - multipart file 전송
 
<br/>

### 6. MockMvcResultMatchers 메소드
  6.1. status
   - 응답에 대한 HTTP 상태 코드 검증

 
  6.2. content
   - 응답 본문 내용 검증


  6.3. model
   - 스프링 MVC 모델 검증

<br/>

### 7. MockMvcResultHandlers 메소드
  7.1. print
   - 요청에 대한 결과 출력

<br/>
주로 제가 사용한 메소드만 써놓은 것으로 이외에 다양한 메소드가 있습니다. <br/>
예제 소스: https://github.com/kokoo-dev/mock
