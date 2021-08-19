## SpringBoot + MongoDB 연동 예제
MongoDB가 설치 과정은 생략하고 SpringBoot에서 설정 후 <br/>
MongoTemplate, MongoRepository 두 가지 방법을 통해 사용하는 간략한 방법을 설명하겠습니다.
<br/><br/>

#### 버전 정보
 - SpringBoot: 2.5.0
 - MongoDB: 4.4.6
<br/><br/>

### 예제 DB 초기 상태
<img src="./img/20210522_1.png"/> <br/>
persons 이름의 Documents에 2개의 값이 들어있는 상태
<br/><br/>

### Dependency 설정
> ex) build.gradle
~~~gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-mongodb'
}
~~~
spring-boot-starter-data-mongodb 추가
<br/><br/>

### yml 설정
> ex) application.yml
~~~yml
spring:
  data:
    mongodb:
      host: localhost
      port: 27017
      database: testdb
~~~
host, port, database 이름만으로 설정은 끝입니다.
<br/><br/>

### Document, Repository, Service 클래스 생성
> ex) PersonDoc.java
~~~java
@Document("persons")
@Data
public class PersonDoc {
    @Id
    private String _id;
    private String name;
    private int age;
}
~~~
@Document 어노테이션에 Documents 이름을 지정해 줍니다.
<br/><br/>

> ex) PersonRepository.java
~~~java
public interface PersonRepository extends MongoRepository<PersonDoc, String> {
    List<PersonDoc> findByAge(int age);
}
~~~
<br/><br/>

> ex) PersonService.java
~~~java
@Service
public class PersonService {

    @Autowired
    private MongoTemplate mongoTemplate;

    @Autowired
    private PersonRepository personRepository;

    public PersonDoc getPerson(String _id){
        PersonDoc personDoc = mongoTemplate.findById(_id,PersonDoc.class);
        return Optional.ofNullable(personDoc).orElseThrow(() -> new RestClientException(HttpStatus.NOT_FOUND.toString()));
    }

    public List<PersonDoc> getPersonList(int age){
        Query query = new Query().addCriteria(Criteria.where("age").is(age));
        return mongoTemplate.find(query, PersonDoc.class);
    }

    public PersonDoc insertPerson(PersonDoc personDoc){
        return mongoTemplate.insert(personDoc);
    }

    public UpdateResult updatePerson(PersonDoc personDoc){
        Query query = new Query();
        Update update = new Update();

        query.addCriteria(Criteria.where("_id").is(personDoc.get_id()));
        update.set("age", personDoc.getAge());

        return mongoTemplate.updateMulti(query, update, "persons");
    }

    public DeleteResult deletePerson(PersonDoc personDoc){
        Query query = new Query();
        query.addCriteria(Criteria.where("name").is(personDoc.getName()));

        return mongoTemplate.remove(query, "persons");
    }

    public PersonDoc getPersonByRepo(String _id){
        return personRepository.findById(_id).orElseThrow(() -> new RestClientException(HttpStatus.NOT_FOUND.toString()));
    }

    public List<PersonDoc> getPersonListByRepo(int age){
        return personRepository.findByAge(age);
    }

    public PersonDoc insertPersonByRepo(PersonDoc personDoc){
        return personRepository.insert(personDoc);
    }

    public PersonDoc updatePersonByRepo(PersonDoc personDoc){
        return personRepository.save(personDoc);
    }

    public void deletePersonByRepo(PersonDoc personDoc){
        personRepository.deleteByName(personDoc.getName());
    }
}
~~~
getPerson(), getPersonList(), insertPerson()는 MongoTemplate를 이용한 방법, <br/>
xxxByRepo()는 MongoRepository를 이용한 방법입니다. <br/>
<br/><br/>

### 테스트 코드
~~~java
@SpringBootTest
public class PersonTests {
    Logger logger = LoggerFactory.getLogger(PersonTests.class);

    @Autowired
    PersonService personService;

    @Test
    public void testSelectPerson(){
        String _id = "60a74978f114367ba856b176";
        PersonDoc personDoc = personService.getPerson(_id);
        assertEquals("HongGilDong", personDoc.getName());
    }

    @Test
    public void testSelectPersonList(){
        int age = 25;
        int expected = 2;

        List<PersonDoc> personDocList = personService.getPersonList(age);

        for(PersonDoc personDoc : personDocList){
            logger.info(personDoc.get_id() + " " + personDoc.getName() + " " + personDoc.getAge());
        }

        assertEquals(expected, personDocList.size());
    }

    @Test
    public void testInsertPerson(){
        PersonDoc personDoc = new PersonDoc();
        personDoc.setAge(21);
        personDoc.setName("KangGilDong");

        PersonDoc result = personService.insertPerson(personDoc);
        logger.info("result :: " + result.get_id() + " " + result.getName() + " " + result.getAge());
    }

    @Test
    public void testUpdatePerson(){
        String _id = "60a74978f114367ba856b176";
        PersonDoc personDoc = personService.getPerson(_id);
        personDoc.setAge(personDoc.getAge() + 1);

        UpdateResult updateResult = personService.updatePerson(personDoc);

        logger.info("result:: " + updateResult.getModifiedCount());
    }

    @Test
    public void testDeletePerson(){
        String name = "KangGilDong";
        PersonDoc personDoc = new PersonDoc();
        personDoc.setName(name);

        DeleteResult deleteResult = personService.deletePerson(personDoc);

        logger.info("result:: " + deleteResult.getDeletedCount());
    }

    @Test
    public void testSelectPersonByRepo(){
        String _id = "60a74978f114367ba856b176";
        PersonDoc personDoc = personService.getPersonByRepo(_id);
        assertEquals("HongGilDong", personDoc.getName());
    }

    @Test
    public void testSelectPersonListByRepo(){
        int age = 25;
        int expected = 2;

        List<PersonDoc> personDocList = personService.getPersonListByRepo(age);

        for(PersonDoc personDoc : personDocList){
            logger.info(personDoc.get_id() + " " + personDoc.getName() + " " + personDoc.getAge());
        }

        assertEquals(expected, personDocList.size());
    }

    @Test
    public void testInsertPersonByRepo(){
        PersonDoc personDoc = new PersonDoc();
        personDoc.setAge(22);
        personDoc.setName("LimGilDong");

        PersonDoc result = personService.insertPersonByRepo(personDoc);
        logger.info("result :: " + result.get_id() + " " + result.getName() + " " + result.getAge());
    }

    @Test
    public void testUpdatePersonByRepo(){
        String _id = "60a74978f114367ba856b176";
        PersonDoc personDoc = personService.getPerson(_id);
        personDoc.setAge(personDoc.getAge() + 1);

        PersonDoc result = personService.updatePersonByRepo(personDoc);

        logger.info("result :: " + result.getAge());
    }

    @Test
    public void testDeletePersonByRepo(){
        String name = "KangGilDong";
        PersonDoc personDoc = new PersonDoc();
        personDoc.setName(name);

        personService.deletePersonByRepo(personDoc);
    }
}
~~~
위 테스트 코드를 통해 CRUD를 확인해 볼 수 있습니다.
<br/>

Repository와 Service를 보다시피 JPA와 굉장히 유사하기에 <br/>
MongoDB의 CRUD 쿼리를 자세히 모름에도 예제를 간단히 만들 수 있었습니다.

<br/><br/>

예제 소스: https://bit.ly/3hMIa34
