Spring Batch에서 Spring Batch Component (ItemReader, ItemProcessor, ItemWriter) 등에 사용합니다. <br/>
Spring에서 Bean은 싱글톤 패턴으로 생성됩니다. <br/>
그러나 해당 어노테이션을 각 컴포넌트 위에 설정해주면 지연 생성(late binding) 즉 Step 실행 시점마다 빈을 생성해주게 됩니다.


> ex)
~~~java
 public class CustomJob{
  @Bean
  @StepScope
  public ItemProcessor<Integer, Integer> process(){
      //TODO
      return new CustomItemProcessor();
  }
}
~~~

참조 주소 : https://docs.spring.io/spring-batch/docs/current/api/org/springframework/batch/core/scope/StepScope.html
