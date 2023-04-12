Spring Data Redis + Embedded Redis 를 이용한 Redis 단위 테스트 방법  <br>

> ex) build.gradle

~~~gradle
dependencies {
    // redis
    implementation 'org.springframework.boot:spring-boot-starter-data-redis'
    
    // test embedded redis
    testImplementation 'it.ozimov:embedded-redis:0.7.3'
}
~~~

단위 테스트용 embedded redis 의존성 추가 <br>

- github: https://github.com/ozimov/embedded-redis

<br>

> ex) application-test.yml

~~~yml
spring:
  config:
    activate:
      on-profile: test
  redis:
#    host: 127.0.0.1
    port: 6379
~~~

test 용 설정 파일, 사용할 포트만 기입

<br>

> ex) Book.java

~~~java
@Getter
@Builder
@RedisHash("book")
@EqualsAndHashCode
public class Book {

    @Id
    private String cacheId;
    private long id;
    private String name;
    private int price;
}
~~~

- @RedisHash : 캐시 키의 prefix 값
- @Id : @RedisHash 로 지정한 prefix 이후의 구분 값
  - 최종적으로 {@RedisHash}:{@Id} 형태의 키로 생성

<br>

> ex) BookRepository.java

~~~java
@Repository
public interface BookRepository extends CrudRepository<Book, String> {

}

~~~

CrudRepository 를 상속받는 Repository 인터페이스 생성

<br>

> ex) EmbeddedRedisConfig.java

~~~java
@TestConfiguration
public class EmbeddedRedisConfig {

    @Value("${spring.redis.port}")
    private int redisPort;

    private RedisServer redisServer;

    @PostConstruct
    public void redisServer() throws IOException {
        redisServer = new RedisServer(redisPort);
        redisServer.start();
    }

    @PreDestroy
    public void stopRedis() {
        if (redisServer != null) {
            redisServer.stop();
        }
    }
}
~~~

Embedded Redis 를 위한 Java 설정 <br>

<br>

> ex) RedisTest.java

~~~java
@DataRedisTest
@ActiveProfiles("test")
@Import(EmbeddedRedisConfig.class)
@DisplayName("Redis 단위 테스트 샘플")
public class RedisTest {

    private static final String BOOK_CACHE_ID = "book-1";

    @Autowired
    private BookRepository bookRepository;

    @BeforeEach
    void BeforeEach() {
        Book book = Book.builder()
            .cacheId(BOOK_CACHE_ID)
            .id(1)
            .name("Clean Code")
            .price(33000)
            .build();

        bookRepository.save(book);
    }

    @DisplayName("책 전체 목록 조회 성공")
    @Test
    void successFindAllBook() {
        Book expected = Book.builder()
            .cacheId(BOOK_CACHE_ID)
            .id(1)
            .name("Clean Code")
            .price(33000)
            .build();

        Book actual = bookRepository.findById(BOOK_CACHE_ID).get();

        assertEquals(expected, actual);
    }

    @DisplayName("책 전체 목록 조회 실패")
    @Test
    void noSuchElementFindAllBook() {
        Class<NoSuchElementException> expected = NoSuchElementException.class;

        assertThrows(expected, () -> bookRepository.findById("book-2").orElseThrow());
    }
}
~~~

Redis 단위 테스트를 위한 간단한 예제 <br>

- @DataRedisTest: redis 테스트를 위한 설정만 사용
- @ActiveProfiles("test"): test 용 application.yml 을 사용
- @Import(EmbeddedRedisConfig.class): @TestConfiguration 으로 등록한 Embedded Redis 설정 사용