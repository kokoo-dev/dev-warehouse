## Redis Pub/Sub 의 사용 ##

Redis Pub/Sub 은 topic 을 구독하고 있는 모든 Subscriber 가 메세지를 수신할 수 있습니다. <br>
애플리케이션 서버가 Scale out 되어있는 경우 Subscriber 로 설정된 서버들은 메세지를 수신 후 독립적인 작업 (로컬 캐시 동기화 등) 을 할 수 있습니다.
<br><br>


## 예제 ##

> ex) build.gradle

~~~gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-redis'
}
~~~

redis 사용을 위한 spring boot data redis 의존성을 추가
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

> ex) PubSubMessage.java

~~~java
@Getter
@NoArgsConstructor
@AllArgsConstructor
public class PubSubMessage implements Serializable {

    private static final long serialVersionUID = 7886980891647899172L;

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

> ex) RedisSubscriber.java

~~~java
@Service
@RequiredArgsConstructor
public class RedisSubscriber implements MessageListener {
    
    private final ObjectMapper mapper;

    @Override
    public void onMessage(Message message, byte[] pattern) {
        try {
            PubSubMessage pubSubMessage = mapper.readValue(new String(message.getBody()), PubSubMessage.class);
            // TODO 로컬 캐시 동기화 처리
        } catch (JsonProcessingException e) {
            throw new RuntimeException(e);
        }
    }
}
~~~

MessageListener를 상속받아 onMessage를 구현하여 발행된 메세지에 대한 추가 작업 처리