다른 db두개를 연결하는 방법입니다.<br/>
yml, java를 이용한 설정방법을 설명합니다.


<h3>테스트 버전 </h3>
- spring boot: 2.5.2 <br/>
- mybatis: 2.1.4
<br/><br/>

> ex) appication.yml
~~~yml
mybatis:
  mysql:
    jdbc-url: {JdbcUrl}
    driver-class-name: {DriverClassName}
    username: {DB ID}
    password: {DB PW}
  mssql:
    jdbc-url: {JdbcUrl}
    driver-class-name: {DriverClassName}
    username: {DB ID}
    password: {DB PW}
~~~

url 설정 부분 이름을 url 이 아닌 jdbc-url로 설정해줍니다.
<br/><br/>

> ex) MybatisConfig.java
~~~java
public abstract class MybatisConfig {

    public abstract DataSource dataSource();

    public abstract SqlSessionFactory sqlSessionFactory(DataSource dataSource,
        ApplicationContext applicationContext) throws Exception;

    public SqlSessionFactory createFactoryBean(DataSource dataSource,
        ApplicationContext applicationContext, String locationPattern)
        throws Exception {
        Configuration configuration = new Configuration();
        //mybatis config 추가

        SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSource);
        sqlSessionFactoryBean.setConfiguration(configuration);
        sqlSessionFactoryBean.setTypeAliasesPackage({base패키지}); //ex) com.example.test
        sqlSessionFactoryBean
            .setMapperLocations(
                applicationContext.getResources(locationPattern));
        sqlSessionFactoryBean.setVfs(SpringBootVFS.class);

        return sqlSessionFactoryBean.getObject();
    }
}
~~~


> ex) MysqlConfig.java
~~~java
@Configuration
@MapperScan(value = {
        {mapper패키지}}, sqlSessionFactoryRef = "sqlSessionFactory")
public class MysqlConfig extends MybatisConfig {

    @Override
    @Bean(name = "dataSource")
    @ConfigurationProperties(prefix = "mybatis.mysql")
    public DataSource dataSource() {
        return DataSourceBuilder.create().build();
    }

    @Override
    @Bean(name = "sqlSessionFactory")
    public SqlSessionFactory sqlSessionFactory(@Qualifier("dataSource") DataSource dataSource,
                                               ApplicationContext applicationContext) throws Exception {
        return createFactoryBean(dataSource, applicationContext, {mapper xml 경로}); //ex) classpath:mapper/**.xml
    }

    @Bean
    public PlatformTransactionManager transactionManager() {
        return new DataSourceTransactionManager(dataSource());
    }
}
~~~

> ex) MssqlConfig.java
~~~java
@Configuration
@MapperScan(value = {
        {mapper패키지}}, sqlSessionFactoryRef = "mssqlSqlSessionFactory")
public class MssqlConfig extends MybatisConfig {

    @Override
    @Bean(name = "mssqlDataSource")
    @ConfigurationProperties(prefix = "mybatis.mssql")
    public DataSource dataSource() {
        return DataSourceBuilder.create().build();
    }

    @Override
    @Bean(name = "mssqlSqlSessionFactory")
    public SqlSessionFactory sqlSessionFactory(@Qualifier("mssqlDataSource") DataSource dataSource,
                                               ApplicationContext applicationContext) throws Exception {
        return createFactoryBean(dataSource, applicationContext, {mapper xml 경로}); //ex) classpath:mapper/**.xml
    }

    @Bean
    public PlatformTransactionManager transactionManager() {
        return new DataSourceTransactionManager(dataSource());
    }
}
~~~
<br/>

두 DB설정파일의 mapperScan 패키지, sqlSessionFactory이름, dataSource이름을 다르게 지정해줍니다.
