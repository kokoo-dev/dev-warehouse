1. Gradle에 의존성 추가
~~~gradle
implementation "org.springframework.boot:spring-boot-starter-data-redis"
implementation "redis.clients:jedis:3.6.3"
implementation "org.springframework.session:spring-session-data-redis"
~~~

2. yml설정 파일에 redis host, port 추가
~~~yml
redis:
  host: {REDIS_HOST}
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
