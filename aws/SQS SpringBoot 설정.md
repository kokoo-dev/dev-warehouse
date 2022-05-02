사내 서비스에서 사용할 메시지큐 종류를 찾던 중, 비용적 측면이나 개발시간 단축과 현재 서비스의 구성 상
AWS 의 메시지큐 서비스인 SQS 를 선택하였고 정말 간단하게 설정한 방법을 정리하고자 합니다. <br>

version
> SpringBoot 2.5.X

<br><br>

<h3>1. AWS 설정</h3>
<h4>1.1. SQS 대기열 생성</h4>
<img src="/img/20220427_1.png" width="400px"> <br>

선입선출 구조의 순서가 보장되는 처리가 필요하기에 FIFO 유형을 선택해줍니다. <br>
FIFO 유형의 경우 대기열 이름의 끝은 .fifo 로 끝나야 합니다.
<br><br>

<h4>1.2. 권한 추가</h4>
<img src="/img/20220427_2.png" width="400px">

<br>

AmazonSQSFullAccess 의 권한을 프로그래밍 방식으로 엑세스 할 수 있게 IAM 권한을 설정해줍니다. <br>

<h3>2. Spring Boot 설정</h3>
<h4>2.1. dependency 추가</h4>

> ex) build.gradle

~~~gradle
...
dependencies {
    implementation 'org.springframework.cloud:spring-cloud-aws-messaging:2.2.6.RELEASE'
    implementation 'org.springframework.cloud:spring-cloud-aws:2.2.6.RELEASE'
}
...
~~~
<br>

<h4>2.2. SQS 정보 설정 추가</h4>

> ex) application.yml

~~~yml
cloud:
  aws:
    stack:
      auto:
    region:
      static: ap-northeast-1 #(1)
    credentials:
      access-key: #(2)
      secret-key: #(3)
    sqs:
      queue:
        name: test-queue.fifo #(4)
        url: #(5)
~~~

(1): 리전 입력 <br>
(2): AmazonSQSFullAccess 권한의 access key <br>
(3): AmazonSQSFullAccess 권한의 secret key <br>
(4): queue 대기열 이름 <br>
(5): queue 대기열 세부정보의 url <br>

<br>

<h4>2.3. Configuration bean 등록 </h4>

> ex) AwsSqsConfig.java

~~~java
@Configuration
public class AwsSqsConfig {
    @Value("${cloud.aws.credentials.access-key}")
    private String accessKey;

    @Value("${cloud.aws.credentials.secret-key}")
    private String secretKey;

    @Value("${cloud.aws.region.static}")
    private String region;

    @Bean
    @Primary
    public AWSCredentialsProvider awsCredentialsProvider() {
        return new AWSStaticCredentialsProvider(new BasicAWSCredentials(accessKey, secretKey));
    }

    @Bean
    public AmazonSQS amazonSQSClient() {
        AmazonSQSClientBuilder builder = AmazonSQSClientBuilder.standard().withCredentials(awsCredentialsProvider());
        builder.withRegion(region);
        
        return builder.build();
    }

    @Bean
    public SimpleMessageListenerContainerFactory simpleMessageListenerContainerFactory(AmazonSQSAsync amazonSQSAsync) {
        SimpleMessageListenerContainerFactory factory = new SimpleMessageListenerContainerFactory();
        factory.setAmazonSqs(amazonSQSAsync);
        factory.setMaxNumberOfMessages(15);
        factory.setWaitTimeOut(30);
        factory.setTaskExecutor(messageThreadPoolTaskExecutor());
        
        return factory;
    }

    @Bean
    public ThreadPoolTaskExecutor messageThreadPoolTaskExecutor() {
        ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        taskExecutor.setThreadNamePrefix("aws-sqs-task-");
        taskExecutor.setCorePoolSize(20);
        taskExecutor.setMaxPoolSize(20);
        taskExecutor.afterPropertiesSet();
        
        return taskExecutor;
    }
}
~~~

<br><br>

<h3>3. Sender & Listener 구성</h3>

> ex) Message.java

~~~java
@Getter
@Builder
public class Message {
    private String message;
    private long createTimestamp;
}
~~~

<br>

> ex) SqsService.java

~~~java
@Service
@RequiredArgsConstructor
@Slf4j
public class SqsService {

    @Value("${cloud.aws.sqs.queue.url}")
    private String sqsUrl;

    private final ObjectMapper objectMapper;
    private final AmazonSQS amazonSQS;

    // (1)
    public void send() throws JsonProcessingException {
        String message = "success";
        long createTimestamp = Calendar.getInstance().getTimeInMillis();

        Message message = Message.builder()
                .message(message)
                .createTimestamp(createTimestamp)
                .build();

        SendMessageRequest sendMessageRequest = new SendMessageRequest(sqsUrl, objectMapper.writeValueAsString(messageDTO))
                .withMessageGroupId("test-queue.fifo")
                .withMessageDeduplicationId(UUID.randomUUID().toString()); //(2)

        SendMessageResult sendMessageResult = amazonSQS.sendMessage(sendMessageRequest);
    }

    //(3)
    @SqsListener(value = "${cloud.aws.sqs.queue.name}", deletionPolicy = SqsMessageDeletionPolicy.ON_SUCCESS)
    private void receiveMessage(@Payload String message) throws JsonProcessingException {
        log.info(message);
        Message readValue = objectMapper.readValue(message, Message.class);
    }
}
~~~

(1): Sender 샘플 메소드<.br>
(2): 메시지 중복 제거를 위한 설정으로 AWS 콘솔에서 메시지 큐 생성 시 콘텐츠 기반 중복 제거 설정으로 대체 가능 <br>
(3): Listener 샘플 메소드로 deletionPolicy (메시지 삭제 정책) 지정<br>

- ALWAYS: 항상 삭제
- NEVER: 삭제 안함
- NO_REDRIVE: Redrive policy가 정의되지 않은 경우 삭제
- ON_SUCCESS: 리스너 메소드가 성공하면 삭제