## Redis Pub/Sub 의 사용 ##

Redis Pub/Sub 은 topic 을 구독하고 있는 모든 Subscriber 가 메세지를 수신할 수 있습니다. <br>
애플리케이션 서버가 Scale out 되어있는 경우 Subscriber 로 설정된 서버들은 메세지를 수신 후 독립적인 작업 (로컬 캐시 동기화 등) 을 할 수 있습니다.
<br><br>


## 예제 ##

> ex) build.gradle

~~~gradle
dependencies {
    // redis
    implementation 'org.springframework.boot:spring-boot-starter-data-redis'
    
    // cache
    implementation 'org.springframework.boot:spring-boot-starter-cache'
	implementation 'org.ehcache:ehcache'
}
~~~

spring boot data redis, ehcache 의존성을 추가
<br><br>

> ex) application.yml

~~~yml
spring:
  redis:
    host: localhost
    port: 6379
    topic:
      name: pubsub-test
~~~

redis 기본 연결 설정과 topic 이름으로 사용할 값 추가
<br><br>

> ex) RedisConfig.java

~~~java
@Configuration
@RequiredArgsConstructor
public class RedisConfig {

    @Value("${spring.redis.topic.name}")
    private String topicName;

    private final RedisSubscriber redisSubscriber;

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {

        return new LettuceConnectionFactory();
    }

    @Bean
    public RedisTemplate<String, Object> redisTemplate() {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory());

        StringRedisSerializer keySerializer = new StringRedisSerializer();
        GenericJackson2JsonRedisSerializer valueSerializer = new GenericJackson2JsonRedisSerializer();

        redisTemplate.setKeySerializer(keySerializer);
        redisTemplate.setValueSerializer(valueSerializer);
        redisTemplate.setHashKeySerializer(keySerializer);
        redisTemplate.setHashValueSerializer(valueSerializer);

        return redisTemplate;
    }

    @Bean
    MessageListenerAdapter messageListenerAdapter() {
        return new MessageListenerAdapter(redisSubscriber);
    }

    @Bean
    RedisMessageListenerContainer redisContainer() {
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        container.setConnectionFactory(redisConnectionFactory());
        container.addMessageListener(messageListenerAdapter(), topic());

        return container;
    }

    @Bean
    ChannelTopic topic() {
        return new ChannelTopic(topicName);
    }
}
~~~

Redis 기본 연결 설정과 Subscriber 를 위한 Listener 설정
<br><br>

> ex) CacheConfig.java

~~~java
@Configuration
@EnableCaching
public class CacheConfig {

    @Bean
    public CacheManager cacheManager() {
        return new ConcurrentMapCacheManager();
    }
}
~~~

<br>

> ex) ehcache.xml

~~~xml
<ehcache>
  <defaultCache
    maxElementsInMemory="100"
    eternal="true"
    overflowToDisk="true"
    maxElementsOnDisk="10000000"
    memoryStoreEvictionPolicy="LRU"/>
</ehcache>
~~~

로컬 캐시를 위한 기본 설정
<br><br>

> ex) PubSubMessage.java

~~~java
@Getter
@NoArgsConstructor
@AllArgsConstructor
public class PubSubMessage {

    private String cacheName;
    private String cacheKey;
    private String content;
}
~~~

메세지로 사용할 클래스
<br><br>

> ex) RedisPublisher.java

~~~java
@Service
@RequiredArgsConstructor
public class RedisPublisher {

    @Value("${spring.redis.topic.name}")
    private String topicName;

    private final RedisTemplate<String, Object> redisTemplate;

    public void sendMessage(PubSubMessage pubSubMessage) {
        redisTemplate.convertAndSend(topicName, pubSubMessage);
    }
}
~~~

RedisTemplate.convertAndSend()를 통해 지정된 topic으로 메세지 발행
<br><br>

> ex) RedisSubscriber.java

~~~java
@Service
@RequiredArgsConstructor
public class RedisSubscriber implements MessageListener {
    
    private final ObjectMapper mapper;
    private final CacheManager cacheManager;

    @Override
    public void onMessage(Message message, byte[] pattern) {
        try {
            PubSubMessage pubSubMessage = mapper.readValue(new String(message.getBody()), PubSubMessage.class);
            cacheManager.getCache(pubSubMessage.getCacheName()).put(pubSubMessage.getCacheKey(), pubSubMessage.getContent());
        } catch (Exception e) {
            
        }
    }
}
~~~

MessageListener를 상속받아 onMessage를 구현하여 발행된 메세지로 로컬 캐시 동기화
<br><br>

> ex) CacheService.java

~~~java
@Service
public class CacheService {

    @Cacheable(value = "local", key = "#key")
    public String get(String key) {

        return "not cache";
    }
}
~~~

캐시가 갱신됐는지 확인