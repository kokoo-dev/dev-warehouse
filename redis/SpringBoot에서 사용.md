1. Gradle에 의존성 추가
~~~gradle
implementation "org.springframework.boot:spring-boot-starter-data-redis"
implementation "redis.clients:jedis:3.6.3"
implementation "org.springframework.session:spring-session-data-redis"
~~~

2. yml설정 파일에 redis host, port 추가
~~~yml
redis:
  host: 127.0.0.1
  port: 6379
~~~


3. Redis Configuration 추가
~~~java
@Configuration
public class RedisConfig {
    @Value("${redis.host}")
    private String host;

    @Value("${redis.port}")
    private int port;

    @Bean
    public JedisConnectionFactory jedisConnectionFactory(){
        JedisConnectionFactory jedisConnectionFactory = null;
        try {
            RedisStandaloneConfiguration conn = new RedisStandaloneConfiguration();
            conn.setHostName(host);
            conn.setPort(port);
            jedisConnectionFactory = new JedisConnectionFactory(conn);
            jedisConnectionFactory.setPoolConfig(jedisPoolConfig());
        }catch (Exception e){
          
        }
        
        return jedisConnectionFactory;
    }

    @Bean
    public RedisTemplate redisTemplate(JedisConnectionFactory jedisConnectionFactory){
        final RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(jedisConnectionFactory);
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        redisTemplate.setHashKeySerializer(new StringRedisSerializer());
        redisTemplate.setHashValueSerializer(new GenericJackson2JsonRedisSerializer());

        return redisTemplate;
    }

    private JedisPoolConfig jedisPoolConfig(){
        JedisPoolConfig config = new JedisPoolConfig();
        config.setMaxTotal(36);
        config.setMaxIdle(36);
        config.setMinIdle(0);
        config.setTestOnBorrow(true);
        config.setTestOnReturn(true);
        config.setTestWhileIdle(true);
        config.setBlockWhenExhausted(false);
        config.setMaxWaitMillis(-1);
        config.setTimeBetweenEvictionRunsMillis(Duration.ofSeconds(60).toMillis());
        config.setMinEvictableIdleTimeMillis(Duration.ofSeconds(30).toMillis());
        config.setNumTestsPerEvictionRun(3);
        
        return config;
    }

    @Bean
    public ConfigureRedisAction configureRedisAction() {
        return ConfigureRedisAction.NO_OP;
    }
}
~~~
위 방식의 jedis는 lettuce에 비해 성능이 좋지 않기에 사용을 잘 안하는 추세라고 합니다. <br>
jedis lettuce로 검색하면 비교 글이 많이 있습니다. <br>
아래는 lettuce로 설정하는 방식이며 connection pool 등 설정도 일부분 사라져 코드가 더 진 것을 볼 수 있습니다.
~~~java
@Configuration
public class RedisConfig {
    @Value("${redis.host}")
    private String host;

    @Value("${redis.port}")
    private int port;

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        RedisStandaloneConfiguration redisStandaloneConfiguration = new RedisStandaloneConfiguration();
        redisStandaloneConfiguration.setHostName(host);
        redisStandaloneConfiguration.setPort(port);

        LettuceConnectionFactory lettuceConnectionFactory = new LettuceConnectionFactory(redisStandaloneConfiguration);
        return lettuceConnectionFactory;
    }

    @Bean
    public RedisTemplate redisTemplate(){
        final RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory());
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        redisTemplate.setHashKeySerializer(new StringRedisSerializer());
        redisTemplate.setHashValueSerializer(new GenericJackson2JsonRedisSerializer());

        return redisTemplate;
    }

    @Bean
    public ConfigureRedisAction configureRedisAction() {
        return ConfigureRedisAction.NO_OP;
    }
}
~~~

4. 데이터 get/set
~~~java
public class RedisTest{
    private final RedisTemplate<String, Object> redisTemplate;
    
    
    public Object getData(String key){
        ValueOperations<String, Object> valueOperations = redisTemplate.opsForValue();
        return valueOperations.get(key);
    }

    public void setData(String key, Object value){
        ValueOperations<String, Object> valueOperations = redisTemplate.opsForValue();
        valueOperations.set(key, value);
    }
}
~~~
