SpringBatch + SpringScheduler에서 JobParameter를 사용하는 방법으로,<br/>
SpringScheduler가 아니더라도 다른 설정으로 JobParameter를 이용할 수 있습니다. <br/><br/>

> ex) TestJobConfig.java
~~~java
@Configuration
public class TestJobConfig {
    @Autowired
    private JobBuilderFactory jobBuilderFactory;

    @Autowired
    private StepBuilderFactory stepBuilderFactory;
    
    @Bean
    public Job testJob() {
        return jobBuilderFactory.get("test-job")
                .start(testStep())
                .build();
    }
    
    @Bean
    public Step testStep() {
        return stepBuilderFactory.get("test-step")
                .<String, String>chunk(100)
                .reader(readTest(null, null, null)) //(1)
                .writer(writeTest())
                .build();
    }
    
    @Bean
    @StepScope
    public ItemReader<String> readTest( @Value("#{jobParameters[param1]}") final Integer param1,
                                        @Value("#{jobParameters[param2]}") final String param2,
                                        @Value("#{jobParameters[param3]}") final Date param3) { //(2)
                                        
        //TODO parameter 활용 및 return
    }
    
    @Bean
    public ItemWriter<String> writeTest(){
        //TODO write
    }
}
~~~



(2)에서는 @Value 어노테이션을 통해 jobParameter로 지정된 이름으로 가져오며, <br/>
(1)에는 parameter 숫자만큼 null로 표시해줍니다. <br/><br/>

> ex) TestScheduler.java
~~~java
public class TestScheduler {
    @Autowired
    private JobLauncher jobLauncher;
    
    @Autowired
    private TestJobConfig testJobConfig;
    
    @Scheduled(cron = "0 * * * * *")
    public void executeTestJob(){
        Map<String, JobParameter> jobParamMap = new HashMap<>(); //(3)
        jobParamMap.put("param1", new JobParameter("11111"));
        jobParamMap.put("param2", new JobParameter("param2"));
        jobParamMap.put("param3", new JobParameter(new Date()));
        
        JobParameters jobParameters = new JobParameters(jobParamMap); //(4)
        
        try {
            jobLauncher.run(testJobConfig.testJob(), jobParameters); //(5)
        } catch (Exception e){
            e.printStackTrace();
        }
    }
}
~~~

(3) Map에다가 사용할 파라미터들을 생성 후 (4)JobParameters를 인스턴스화 해줍니다. <br/>
이후 (5) JobLauncher.run() 메소드에 실행할 Job과 jobParameters를 넘겨주면 <br/>
위 TestJobConfig.java에서 처럼 사용할 수 있습니다.
