## JUnit + MockMvc를 활용하여 컨트롤러 테스트하기
주로 컨트롤러 테스트를 Postman 같은 외부 툴로 하다가 테스트 단계에서 할 수 없을까 하고 찾아보니 Mock이 있어서 사용하게 되었습니다. <br/>
설정이나 활용 방법은 간단하게 구현할 수 있었습니다.


MockMvc는 가짜 객체를 만들어 WAS에 배포없이 컨트롤러를 테스트 할 수 있는 클래스라고 알아두면 될 것 같습니다.<br/><br/>

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

    @GetMapping("/callCheckReturnStr")
    public String callCheckReturnStr(String data){
        String returnData = data.concat("_return");
        
        return returnData;
    }
    
    @PostMapping("/callWithOneFile")
    public void callWithOneFile(MultipartFile file){
        logger.info("callWithOneFile fileName:: " + file.getOriginalFilename());
    }
}
~~~

### 3. 테스트 클래스 작성
> ex) SampleTests.java
~~~java
@SpringBootTest
@AutoConfigureMockMvc
public class SampleTests {

    @Autowired
    SampleController sampleController;

    @Autowired
    private MockMvc mockMvc;
    
    @BeforeEach
    public void createSampleController(){
        mockMvc = MockMvcBuilders.standaloneSetup(sampleController)
            .build();
    }
    
    @Test
    public void testCallNotFound() throws Exception {
        String url = "/404";
        mockMvc.perform(MockMvcRequestBuilders.get(url)
            .accept(MediaType.TEXT_HTML))
            .andExpect(MockMvcResultMatchers.status().isNotFound())
            .andDo(MockMvcResultHandlers.print());
    }
    
    @Test
    public void testCallCheckReturnStr() throws Exception {
        String param = "TestParam";
        String url = "/callCheckReturnStr";

        mockMvc.perform(MockMvcRequestBuilders.get(url)
            .accept(MediaType.TEXT_HTML)
            .param("data", param))
            .andExpect(MockMvcResultMatchers.content().string(param.concat("_return")))
            .andDo(MockMvcResultHandlers.print());
    }
    
    @Test
    public void testCallWithOneFile() throws Exception{
        String url = "/callWithOneFile";
        MockMultipartFile mockMultipartFile = new MockMultipartFile("file", "test.txt", "text/plain", "Mock Test Text File".getBytes());

        mockMvc.perform(MockMvcRequestBuilders.multipart(url)
            .file(mockMultipartFile))
            .andExpect(MockMvcResultMatchers.status().isOk())
            .andDo(MockMvcResultHandlers.print());
    }
}
~~~


 3.1. @SpringBootTest, @AutoConfigureMockMvc 어노테이션 추가
  - 테스트, MockMvc를 사용하기 위한 어노테이션


 3.2. createSampleController() 메소드
  - SampleController를 테스트 하기 위함을 지정

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
