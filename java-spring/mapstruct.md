클래스 복사를 위해서는 ModelMapper 나 BeanUtils 의 copyProperties 를 사용할 수도 있지만 <br>
속도나 안정성의 측면에서 이점이 있는 mapstruct 를 사용하게 됩니다. <br>
리플렉션이 일어나는 ModelMapper 등의 방식 보다 속도가 빠르며 컴파일 시점에 구현체를 생성하기 때문에 런타임이 아닌 컴파일 시점에서 오류를 알 수 있다는 장점이 있습니다. <br>

<br>

<h4> 1. 의존성 추가 </h4>

> ex) build.gradle

~~~gradle
implementation 'org.projectlombok:lombok:1.18.22'
annotationProcessor 'org.projectlombok:lombok:1.18.22'
annotationProcessor 'org.projectlombok:lombok-mapstruct-binding:0.2.0'

implementation 'org.mapstruct:mapstruct:1.4.2.Final'
annotationProcessor 'org.mapstruct:mapstruct-processor:1.4.2.Final'
~~~

Lombok 과 함께 사용 할 경우 충돌이 발생할 수 있기에 lombok-mapstruct-binding 도 추가가 필요합니다. <br>
이 방법은 mapstruct 1.2.0 버전과 Lombok 1.18.16 이상의 버전에서 사용 가능합니다.

<br><br>

<h4> 2. 샘플 클래스 생성 </h4>

> ex) Source.java

~~~java
@Getter
public class Source {
    private int sourceField1;
    private String sourceField2;
    private Date sourceField3;
}
~~~

> ex) Target.java

~~~java
//@Setter
@Builder
public class Target {
    private int targetField1;
    private String targetField2;
    private Date targetField3;
}
~~~


원본 클래스에는 조회하기 위한 Getter 가, 대상 클래스에는 입력을 위한 Setter 나 Builder 가 필수적입니다.

<br><br>

<h4> 3. Mapper 생성 </h4>

> ex) TestMapper.java

~~~java
@Mapper(componentModel = "spring") //(1)
public interface TestMapper {
    TestMapper INSTANCE = Mappers.getMapper(TestMapper.class); //(2)

    @Mappings({
            @Mapping(source = "sourceField1", target = "targetField1"), //(3)
            @Mapping(source = "sourceField2", target = "targetField2"),
            @Mapping(target = "targetField3", ignore = true) //(4)
    })
    Target copySourceToTarget(Source source);
}
~~~

(1) : componentModel = "spring" 으로 지정해주면 빈으로 등록하여 사용 가능 <br>
(2) : Mapper 인스턴스를 받아와 사용 가능 <br>
(3) : source 클래스와 target 클래스의 필드명 지정 <br>
(4) : 특정 필드는 복사하지 않으려는 경우, target 만 지정 후 ignore = true 설정 <br>

위 Mapper 를 설정 후 컴파일 하게 되면 아래와 같은 구현체가 생성됩니다. <br><br>

~~~java
@Generated(
    value = "org.mapstruct.ap.MappingProcessor",
    date = "2022-06-23T21:25:45+0900",
    comments = "version: 1.4.2.Final, compiler: IncrementalProcessingEnvironment from gradle-language-java-7.4.1.jar, environment: Java 15.0.1 (Oracle Corporation)"
)
@Component
public class TestMapperImpl implements TestMapper {

    @Override
    public Target copySourceToTarget(Source source) {
        if ( source == null ) {
            return null;
        }

        TargetBuilder target = Target.builder();

        target.targetField1( source.getSourceField1() );
        target.targetField2( source.getSourceField2() );

        return target.build();
    }
}
~~~

<br><br>

<h4> 4. 사용 예시 </h4>

> ex) SampleService.java

~~~java
@Service
@RequiredArgsConstructor
public class SampleService {
    private final TestMapper testMapper; //(1)
    
    public void copyBean(){
        Source source = new Source();
        // source set
        
        Target target = testMapper.copySourceToTarget(source);
    }

    public void copyInstance(){
        Source source = new Source();
        
        Target target = TestMapper.INSTANCE.copySourceToTarget(source); //(2)
    }
}
~~~

(1) : 위에서 @Mapper(componentModel = "spring") 설정을 통해 빈으로 등록해서 의존성 주입을 통해 사용 가능 <br>
(2) : 위에서 Mappers.getMapper() 를 통해 인스턴스를 가져왔기에 싱클톤 방식으로 사용 가능