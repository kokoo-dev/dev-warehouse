## JPA, MongoDB Transaction 동시 설정 사용 방법

### version
- spring boot: 3.2.3
- java: 17
- kotlin: 1.9.22
- mysql: 8.0.x
- mongodb: 4.x

<br><br>

> build.gradle

~~~gradle
dependencies {
    // ..생략..
    
    // jpa
    implementation("org.springframework.boot:spring-boot-starter-data-jpa")
    
    // mongo
    implementation("org.springframework.boot:spring-boot-starter-data-mongodb")
}
~~~

<br><br>

> TransactionConfig.kt

~~~kt
@Configuration
@EnableMongoAuditing
class TransactionConfig {

    companion object {
        const val MONGODB_TRANSACTION_NAME: String = "mongodbTransactionManager"
    }
    
    @Bean
    @Primary
    fun transactionManager(): PlatformTransactionManager {
        return JpaTransactionManager()
    }

    @Bean(name = [MONGODB_TRANSACTION_NAME])
    fun transactionManager(dbFactory: MongoDatabaseFactory): MongoTransactionManager {
        return MongoTransactionManager(dbFactory)
    }
}
~~~

<br><br>

> Executor.kt

~~~kt
@Component
class Executor {

    @Transactional // @Primary 설정으로 별도 value 지정 필요 없음
    fun executeJpaTransaction() {
        // TODO jpa
    }

    @Transactional(value = MONGODB_TRANSACTION_NAME) // 빈으로 등록된 이름 설정
    fun executeMongodbTransaction() {
        // TODO mongodb
    }
}
~~~