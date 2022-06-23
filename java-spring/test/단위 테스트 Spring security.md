단위 테스트 코드를 작성하였을 때 스프링 시큐리티가 적용된 환경이라면 인증 정보가 없기 때문에 원하는 결과를 얻지 못할 때가 있습니다. <br>

> ex) SecurityTest.java

~~~java
@ExtendWith(SpringExtension.class)
@WebMvcTest(controllers = TestController.class)
class SecurityTest {
    @Autowired
    private MockMvc mockMvc;

    @DisplayName("테스트 화면 조회 성공")
    @Test
    void testSecurityUser(){
        String uri = "/test";
        
        ResultActions resultActions = mockMvc.perform(get(uri)
                .accept(MediaType.TEXT_HTML));
        
        resultActions.andExpect(status().isOk());
    }
}
~~~

/test 가 로그인한 사용자만 접근이 가능한 경로일 경우, <br>
기대 값인 isOk (= 200) 과는 다르게 isFound (=302) 라는 결과가 나옵니다. <br>
이는 스프링 시큐리티의 설정으로 인해 인증되지 않은 접근을 리다이렉트 시키기 때문입니다. <br>
이는 인증된 가짜 유저를 설정하여 처리가 가능합니다. <br><br>

> ex) build.gradle

~~~gradle
dependencies {
    implementation 'org.springframework.security:spring-security-test'
}
~~~

spring boot start test 말고도 security 테스트를 위한 의존성을 추가해줍니다. <br>
<br>

~~~java
@ExtendWith(SpringExtension.class)
@WebMvcTest(controllers = TestController.class)
class SecurityTest {
    @Autowired
    private MockMvc mockMvc;

    @DisplayName("테스트 화면 조회 성공")
    @Test
    @WithMockUser(username = "testUser", password = "P@ssw0rd", roles = {"USER", "ADMIN"})
    void testSecurityUser(){
        String uri = "/test";

        ResultActions resultActions = mockMvc.perform(get(uri)
                .accept(MediaType.TEXT_HTML));

        resultActions.andExpect(status().isOk());
    }
}
~~~
위 SecurityTest.java 와 같은 테스트 메소드에서 @WithMockUser 만 추가되었습니다. <br>
username (아이디), password (비밀번호), roles (권한)을 지정할 수 있습니다.  <br>
로그인 유저가 아닌 익명의 유저를 테스트 하고 싶은 경우 @WithAnonymousUser 로 대체하면 됩니다.