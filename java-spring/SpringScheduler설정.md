특정 시간마다 작업을 하는 구성을 SpringScheduler를 통해서 간단하게 설정하는 방법입니다.

> ex) SchedulerApplication.java
~~~java
@EnableScheduling
@SpringBootApplication
public class SchedulerApplication {

  public static void main(String[] args) {
      SpringApplication.run(SchedulerApplication.class, args);
  }
}
~~~

스케쥴러를 이용할거라는 것을 @EnableScheduling 어노테이션을 이용해 설정해줍니다.
<br/><br/>

> ex) SchedulerConfig.java
~~~java
@Configuration
public class SchedulerConfig implements SchedulingConfigurer {

    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
        ThreadPoolTaskScheduler threadPoolTaskScheduler = new ThreadPoolTaskScheduler();
        threadPoolTaskScheduler.setPoolSize(8); // (1)
        threadPoolTaskScheduler.setThreadNamePrefix("scheduler-thread-"); // (2)
        threadPoolTaskScheduler.initialize();

        taskRegistrar.setTaskScheduler(threadPoolTaskScheduler);
    }
}
~~~
(1) 쓰레드 갯수, (2) 쓰레드 이름 앞에 사용할 접두사를 지정해줍니다.
<br/><br/>


> ex) TestScheduler.java
~~~java
@Component
public class TestScheduler {
    @Scheduled(cron = "0 * * * * *") // (1)
    public void executeTest(){
        System.out.println("*");
    }
}
~~~

(1) @Scheduled 어노테이션을 통해 해당 메소드가 스케줄에 맞게 실행됨을 설정하였고, <br/>
실행주기는 cron = "" 을 통해 크론 표현식으로 설정하였습니다.<br/>
크론 표현식은 초 | 분 | 시 | 일 | 월 | 요일 | 연도 순서이며 연도는 생략이 가능합니다. <br/>
예제의 경우 매분 0초마다 *을 출력하게됩니다.
