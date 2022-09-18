---
layout: post
title: "[soldout] DB Replication 적용"
author: kimcno3
categories: f-lab
tags: f-lab project1
---

## # 문제점
DB에 접근하는 로직은 크기 쓰기에 대한 요청과 읽기에 대한 요청이 많습니다.

그렇기 때문에 @Transactional 어노테이션을 선언하는 경우에도 읽기만을 위한 메소드엔 @Transactional(readOnly = true)로 선언해 의도치 않은 데이터 수정에 대해 방지하는 구조로 설계합니다.

또한 DB의 입장에선 쓰기에 대한 요청보단 읽기에 대한 요청이 더 자주 발생하는 경우가 많습니다. 이런 상황에서 읽기 처리에 대한 datasource와 쓰기 처리에 대한 datasource가 공유된다면 상대적으로 처리량은 적고 소요시간은 긴 쓰기 요청에 대한 처리가 비효율적으로 처리될 가능성이 높아집니다.

예를 들어 새로운 제품에 대한 등록을 하기 위해 날린 요청을 처리하기 위해 수많은 조회 요청에 대한 처리를 기다려야 할 수 있다는 뜻입니다.

이를 조금 더 효율적인 구조로 변경하기 위해 Master-Slave 구조를 적용해 보기로 했습니다.

## # 해결방안
### Master-Slave 구조

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Ftr78z%2Fbtq96EtX4AQ%2FZjOT5sSMZTI2dYSkJXb1U1%2Fimg.jpg)

`Master-Slave` 구조란 위 그림처럼 단순 읽기 처리에 대한 요청과 쓰기 처리에 대한 요청을 처리해주는 DB를 구분해 목적이 다른 두 요청의 처리 속도를 높혀줄 수 있는 구조를 의미합니다.

이를 구성하기 위해선 Master를 위한 datasource와 Slave를 위한 datasource를 구분해 빈 객체로 생성해줄 필요가 있습니다.

#### MasterDataSourceConfig
```java
@Configuration
@MapperScan(value = " api.soldout.io.soldout.mapper",
            sqlSessionFactoryRef = "masterSqlSessionFactory")
public class MasterDataSourceConfig {

  @Value("${mybatis.mapper-locations}")
  String mapperPath;

  @Primary
  @Bean(name = "masterDataSource")
  @ConfigurationProperties(prefix = "spring.datasource.master.hikari")
  public DataSource masterDataSource() {

    return DataSourceBuilder.create()
        .type(HikariDataSource.class)
        .build();
  }

  @Primary
  @Bean(name = "masterSqlSessionFactory")
  public SqlSessionFactory sqlSessionFactory(@Qualifier("masterDataSource") DataSource dataSource,
      ApplicationContext applicationContext) throws Exception {
    SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
    sqlSessionFactoryBean.setDataSource(dataSource);
    sqlSessionFactoryBean.setMapperLocations(applicationContext.getResources(mapperPath));
    return sqlSessionFactoryBean.getObject();
  }

  @Bean(name = "masterSessionTemplate")
  public SqlSessionTemplate sqlSessionTemplate(@Qualifier("masterSqlSessionFactory")
      SqlSessionFactory firstSqlSessionFactory) {
    return new SqlSessionTemplate(firstSqlSessionFactory);
  }

}
```

Master DB에 대한 DataSource 구성 파일입니다.

#### SlaveDataSourceConfig
```java
@Configuration
@MapperScan(value = " api.soldout.io.soldout.mapper",
            sqlSessionFactoryRef = "slaveSqlSessionFactory")
public class SlaveDataSourceConfig {

  @Value("${mybatis.mapper-locations}")
  String mapperPath;

  @Bean(name = "slaveDataSource")
  @ConfigurationProperties(prefix = "spring.datasource.slave.hikari")
  public DataSource slaveDataSource() {

    return DataSourceBuilder.create()
        .type(HikariDataSource.class)
        .build();
  }

  @Bean(name = "slaveSqlSessionFactory")
  public SqlSessionFactory sqlSessionFactory(@Qualifier("slaveDataSource") DataSource dataSource,
      ApplicationContext applicationContext)
      throws Exception {
    SqlSessionFactoryBean sqlSessionFactoryBean = new SqlSessionFactoryBean();
    sqlSessionFactoryBean.setDataSource(dataSource);
    sqlSessionFactoryBean.setMapperLocations(applicationContext.getResources(mapperPath));
    return sqlSessionFactoryBean.getObject();
  }

  @Bean(name = "slaveSessionTemplate")
  public SqlSessionTemplate sqlSessionTemplate(@Qualifier("slaveSqlSessionFactory")
      SqlSessionFactory firstSqlSessionFactory) {
    return new SqlSessionTemplate(firstSqlSessionFactory);
  }

}
```
Slave DB에 대한 DataSource 구성 파일입니다.

#### ReplicationRoutingDataSource
```java
public class ReplicationRoutingDataSource extends AbstractRoutingDataSource {

  @Override
  protected Object determineCurrentLookupKey() {

    return TransactionSynchronizationManager
        .isCurrentTransactionReadOnly() ? DataSourceType.SLAVE : DataSourceType.MASTER;

  }

}
```
`AbstractRoutingDataSource` 는 조회된 key를 기반으로 등록된 `Datasource` 중 하나를 호출하도록 조건문을 선언해줬습니다.

- `@Transactional(readOnly = false)` : Master DB 연결
- `@Transactional(readOnly = true)` : Slave DB 연결

#### RoutingDataSourceConfig
```java
public class RoutingDataSourceConfig {

  @Bean(name = "routingDataSource")
  public DataSource routingDataSource(
            @Qualifier("masterDataSource") final DataSource masterDataSource,
            @Qualifier("slaveDataSource") final DataSource slaveDataSource) {

    ReplicationRoutingDataSource routingDataSource = new ReplicationRoutingDataSource();

    Map<Object, Object> dataSourceMap = new HashMap<>();

    dataSourceMap.put(DataSourceType.MASTER, masterDataSource);
    dataSourceMap.put(DataSourceType.SLAVE, slaveDataSource);

    routingDataSource.setTargetDataSources(dataSourceMap);
    routingDataSource.setDefaultTargetDataSource(masterDataSource);

    return routingDataSource;

  }

  @Bean(name = "dataSource")
  public DataSource dataSource(@Qualifier("routingDataSource") DataSource routingDataSource) {

    return new LazyConnectionDataSourceProxy(routingDataSource);

  }

}
```
생성한 두 datasource 객체를 Map 자료구조에 저장하고 `ReplicationRoutingDataSource`에 저장했습니다.

키로 활용되는 값은 Enum 클래스로 추가 선언해 문자열에 의한 에러를 최대한 방지해줬습니다.

## # 마치며
Replication 구조를 DB에 적용함으로써 하나의 서버로만 구성하는 구조에서 서버 다운 시 발생할 수 있는 심각한 장애를 방지하는 구조로 리팩토링할 수 있었습니다.

## # 참고 자료
- https://k3068.tistory.com/102
- https://taes-k.github.io/2020/03/11/sprinig-master-slave-dynamic-routing-datasource/