Controller 테스트 시 @WebMvcTest 로 인해 읽은 Bean 들을 제외하고 다른 Bean 들로 인해 오류가 발생할 수 있습니다. (시큐리티 설정 등)<br>
이 때 특정 Bean 만 필터링하여 제외시키는 방법입니다. <br><br>

~~~java
@ExtendWith(SpringExtension.class)
@WebMvcTest(controllers = SampleController.class,
    excludeFilters = {
        @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE, classes = SecurityConfig.class)
    })
class SampleControllerTest {
    // 내용..
}
~~~

위에서 중요하게 볼곳은 excludeFilters 부분입니다. <br>
위 예제에서는 type 을 ASSIGNABLE_TYPE 으로 지정하여 특정 클래스를 제외할 수 있도록 하였으나 이 외에 다른 type 으로도 설정할 수 있습니다. <br>
- FilterType.ANNOTATION
- FilterType.ASPECTJ
- FilterType.REGEX
- FilterType.CUSTOM

어노테이션이나, aspect, 정규식 등을 기준으로 필터링 할 수 있습니다. <br>

본 내용은 제외에 대한 예제지만 excludeFilters 를 includeFilters 로 바꾸면 반대로 포함시키는 역할을 하게 됩니다.
