# 스프링 배치
## @EnableBatchProcessing
최신 스프링 부트에서는 @EnableBatchProcessing 를 더 이상 사용하지 않는다.  
본래 이 어노테이션이 스프링 배치의 자동 구성을 활성화하는데, 이로인해 충돌로 배치 관련 테이블이 자동생성되지 않는 이슈가 있었다.

## No bean named 'transactionManager' available: No matching TransactionManager
해당 에러가 발생한 이유는 아래 @Bean 등록 name을 transactionManager로 바꾸면서 해결되었다.

```java
@Bean(name = "jpaTransactionManager")
public JpaTransactionManager jpaTransactionManager(EntityManagerFactory entityManagerFactory) {
    return new JpaTransactionManager(entityManagerFactory);
}
```

```java
@Bean(name = "transactionManager")
public JpaTransactionManager jpaTransactionManager(EntityManagerFactory entityManagerFactory) {
    return new JpaTransactionManager(entityManagerFactory);
}
```

스프링 배치, 스프링 JPA는 transactionManager라는 이름의 빈을 우선적으로 사용하려는 경향이 있다고한다.  
그래서 jpaTransactionManager라는 이름으로 등록 시, 빈을 못찾는다는 에러가 발생한 것이다.  

### JpaTransactionManager
JpaTransactionManager는 JPA기반의 데이터베이스 트랜잭션을 다루는 관리자이다. 트랜잭션이란 데이터베이스 작업을 "여러 개의 작업을 하나의 묶음"으로 묶어서 처리하는 개념이다.  

### EntityManager
EntityManager는 DB작업을 실제 수행하는 객체를 만들어내는 공장이다. JpaTransactionManager는 이 공장에서 만들어진 EntityManager를 이용해 트랜잭션을 commit, rollback 등을 하며 DB 작업을 처리한다.

### EntityManager와는 다르게 JpaTransactionManager는 왜 Bean 등록이 필요할까?
EntityManagerFactory는 JPA구현체가 제공하는 핵심 객체라서 스프링 부트에서 JPA관련 설정을 하면 자동으로 Bean 등록이 됨.  
JpaTransactionManager는 반면 자동 Bean 등록이 안된 이유는 상황에 따라서 어떤 트랜잭션 관리자를 사용할지 명확하지 않기 때문에 명시적으로 Bean등록을 해야함.  
예: JPA는 JpaTransactionManager, JDBC는 DataSourceTransactionManager, 복합 트랜잭션 상황에서는 JtaTransactionManager를 사용함.  

## Step 과 Job
JPA프로젝트의 경우 TransactionManager를 JpaTransactionManager로 설정해야한다.  

```java
@Bean
public Step step1() {
    return new StepBuilder("step1", jobRepository)
            .<CrawlMiceEntity, CrawlMiceEntity>chunk(10, jpaTransactionManager)
            .reader(new CrawlMiceReader(BASE_URL + "/kor/customer/index.php?code=notice", chromeDriverConfig, redisService, serviceName))
            .processor(urlFilterProcessor)
            .writer(jpaItemWriter(entityManagerFactory))
            .allowStartIfComplete(true)
            .build();
}

@Bean
public Job crawlMiceJob(Step step1) {
    return new JobBuilder("crawlMiceJob", jobRepository)
            .start(step1)
            .build();
}

@Bean
public JpaItemWriter<CrawlMiceEntity> jpaItemWriter(EntityManagerFactory entityManagerFactory) {
    JpaItemWriter<CrawlMiceEntity> writer = new JpaItemWriter<>();
    writer.setEntityManagerFactory(entityManagerFactory);
    return writer;
}
```

### JpaItemWriter
JpaItemWriter는 쓰기 작업만 수행하는 스프링 배치의 구성 요소이다.  

- JpaItemWriter는 DB 쓰기 작업만 하기 때문에 EntityManager만 사용한다.
- EntityManager는 EntityManagerFactory를 통해 생성된다.
- 트랜잭션은 Step에서 설정된 관리자(JpaTransactionManager)가 관리하므로 JpaItemWriter에서는 불필요하다.