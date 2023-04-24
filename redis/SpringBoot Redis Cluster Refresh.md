AWS ElastiCache 와 같은 Cluster 형태의 Redis 운영 시 Maintenance 나 노드 유형을 변경할 경우, <br>
각 애플리케이션에서의 client 에서는 변경되는 노드들의 정보를 갱신해주어야 합니다. <br>
Spring Boot 에서 무중단으로 운영이 가능한 설정 방법을 설명합니다. <br>

<h2>version</h2>

- Spring Boot: 2.7.x

> ex) build.gradle

~~~gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-redis'
}
~~~

<br>

> ex) application.yml

~~~yml
spring:
  redis:
    host: 127.0.0.1
    port: 6379
~~~

<br>

> ex) RedisConfig.java

~~~java
@Configuration
@RequiredArgsConstructor
public class RedisConfig {

    private final RedisProperties redisProperties;

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        RedisClusterConfiguration clusterConfiguration = new RedisClusterConfiguration();
        clusterConfiguration.clusterNode(redisProperties.getHost(), redisProperties.getPort());
        LettuceClientConfiguration clientConfiguration = LettuceClientConfiguration.builder()
            .clientOptions(createClusterClientOptions())
            .build();

        return new LettuceConnectionFactory(clusterConfiguration, clientConfiguration);
    }

    @Bean
    public RedisTemplate<String, Object> redisTemplate() {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory());
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());

        return redisTemplate;
    }

    private ClusterClientOptions createClusterClientOptions() {
        return ClusterClientOptions.builder()
            .topologyRefreshOptions(ClusterTopologyRefreshOptions.builder()
                .enablePeriodicRefresh() // (1)
                .enableAllAdaptiveRefreshTriggers() // (2)
                .build())
            .build();
    }
}
~~~

- (1): 일정 시간마다 변경을 감지하고 갱신
  - default: 60초
- (2): 특정 이벤트 발생 시 갱신
  - default: 모든 이벤트 
  - RefreshTrigger 값으로 특정 이벤트만 설정 가능
    - MOVED_REDIRECT
    - ASK_REDIRECT
    - PERSISTENT_RECONNECTS
    - UNCOVERED_SLOT
    - UNKNOWN_NODE