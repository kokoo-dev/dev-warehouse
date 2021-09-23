웹과 같은 실시간 처리가 아닌 일괄적으로 처리하는 방식입니다. <br/>
데이터를 읽고 처리하고 저장하는 흐름을 간단하게 구현할 수 있게 스프링에서 제공해줍니다. <br/>
데이터는 api가 될수도, db가 될수도 있습니다. api조회 -> db저장, db조회 -> api저장, api조회 -> api저장 등 <br/>
Chunk 지향 예제입니다.<br/><br/>

> ex) build.gradle
~~~gradle
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter'
	testImplementation 'org.springframework.boot:spring-boot-starter-test'

	implementation 'org.springframework.boot:spring-boot-starter-batch' //(1)
	testImplementation 'org.springframework.batch:spring-batch-test'
}
~~~
(1) 스프링 부트 배치 스타터 의존성 추가 <br/><br/>

> ex) BatchApplication.java
~~~java
@EnableBatchProcessing //(1)
@SpringBootApplication
public class BatchApplication {

  public static void main(String[] args) {
    SpringApplication.run(BatchApplication.class, args);
  }
}
~~~
(1) @EnableBatchProcessing 어노테이션 추가 <br/><br/>

> ex) JobConfig.java
~~~java
@Configuration
public class JobConfig {

  @Autowired
  JobBuilderFactory jobBuilderFactory;

  @Autowired
  StepBuilderFactory stepBuilderFactory;
    
  @Bean
  public Job job() {
      return jobBuilderFactory.get("job_name")
              .start(step())
              .build();
  }

  @Bean
  public Step step() {
      return stepBuilderFactory.get("step_name")
              .<InputData, OutputData>chunk(1000) //(1)
              .reader(read())
              .processor(process()) //(2)
              .writer(write())
              .build();
  }
    
  @Bean
  public QueueItemReader<InputData> read(){
      // 데이터 조회
      List<InputData> list = new ArrayList<>();
      
      return new QueueItemReader<>(list);
  }
  
  @Bean
  public ItemProcessor<InputData, OutputData> process(){
      return new CustomItemProcessor();
  }
  
  @Bean
  public ItemWriter<OutputData> write(){
      return new CustomItemWriter();
  }
}
~~~
(1) 데이터를 저장할 때 분할크기 <br/>
(2) 데이터를 처리하는 구간으로 reader, writer와 달리 생략이 가능합니다. <br/><br/>

QueueItemReader, CustomItemProcessor, CustomItemWriter 는 각각 ItemReader, ItemProcessor, ItemWriter를 상속받아 구현한 클래스입니다. <br/><br/>
> ex) QueueItemReader.java
~~~java
public class QueueItemReader<T> implements ItemReader<T> {
    private Queue<T> queue;

    public QueueItemReader(List<T> data){
        this.queue = new LinkedList<>(data);
    }

    @Override
    public T read() throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException {
        return queue.poll();
    }
}
~~~
<br/><br/>

> ex) CustomItemProcessor.java
~~~java
public class CustomItemProcessor implements ItemProcessor<InputData, OutputData> {

    @Override
    public OutputData process(InputData inputData) {
	// InputData -> OutputData 로 데이터 처리

	return new OutputData();
    }
    
}
~~~
<br/><br/>
> ex) CustomItemWriter.java
~~~java
public class CustomItemWriter implements ItemWriter<OutputData> {
    @Override
    public void write(List<? extends OutputData> outputDataList) throws Exception {
    	//데이터 저장 처리
    }
}
~~~
