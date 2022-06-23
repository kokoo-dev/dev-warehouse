스프링 캐시인 ehcache예제입니다.

> ex) build.gradle
~~~gradle
dependencies {
  implementation 'org.springframework.boot:spring-boot-starter-cache'
}
~~~

spring-boot-starter-cache 의존성을 추가해줍니다.
<br/><br/>

> ex) ehcache.xml
~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd"
         updateCheck="false">
    <diskStore path="java.io.tmpdir" />
    <cache name="testCache"
           maxEntriesLocalHeap="1000"
           maxEntriesLocalDisk="1000"
           eternal="false"
           diskSpoolBufferSizeMB="20"
           timeToIdleSeconds="300" timeToLiveSeconds="600"
           memoryStoreEvictionPolicy="LRU"
           transactionalMode="off">
        <persistence strategy="localTempSwap" />
    </cache>
</ehcache>
~~~
- name: java 코드에서 사용할 이름
- maxEntriesLocalHeap: 메모리에 생성될 최대 캐시 개수
- maxEntriesLocalDisk: 디스크에 생성될 최대 캐시 개수
- eternal: 영속성 여부, true시 하위 두개 설정은 무시
- timeToIdleSeconds: 해당 초동안 캐시가 미호출시 삭제
- timeToLiveSeconds: 해당 초가 지나면 캐시 삭제
- memoryStoreEvictionPolicy: 캐시 객체수 가득찰 시 삭제 기법 (LRU, LFU, FIFO)
- transactionalMode: copyOnRead, copyOnWrite 시 트랜잭션 모드 설정
- diskSpoolBufferSizeMB: 스폴 버퍼 디스크 크기 설정 (OOM발생시 크기 낮추기)
<br/><br/>

> ex) TestController.java
~~~java
@RestController
@EnableCaching
public class TestController {
    @Autowired
    TestRepository testRepository;

    @GetMapping("/cache/{testData}")
    public Long getPerformTime(@PathVariable String testData){
        long start = System.currentTimeMillis(); // 수행시간 측정
        testRepository.getTestData(testData);
        long end = System.currentTimeMillis();

        return end - start;
    }

    @GetMapping("/refresh/{testData}")
    public void refresh(@PathVariable String testData){
        testRepository.clearCache(testData);
    }
}
~~~
- @EnableCaching 추가
- 캐시 데이터로 쓸 testData를 @PathVariable로 설정
<br/><br/>

> ex) TestRepository.java
~~~java
@Repository
public class TestRepository {

    @Cacheable(value="testCache", key = "#testData")
    public String getTestData(String testData){
        delayTime();
        return testData;
    }

    @CacheEvict(value="testCache", key = "#testData")
    public void clearCache(String testData){
        System.out.println("clear");
    }

    private void delayTime(){
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
~~~
- @Cacheable 어노테이션 추가
- value 값은 ehcache.xml에서 지정해준 name값
- key 값은 해당 캐시의 키값으로 사용
- @CacheEvict는 해당 value, key에 해당하는 캐시값 삭제
