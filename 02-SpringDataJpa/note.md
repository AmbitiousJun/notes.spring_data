## 1. SpringDataJpa快速入门

### 1.1 数据库表

> 数据库表由JPA自动生成即可

### 1.2 创建Java工程，引入依赖

- JPA相关依赖
- Spring依赖
- SpringDataJpa依赖

```xml
<dependencies>
        <!--Junit-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <!--lombok-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.22</version>
        </dependency>
        <!--hibernate-->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-entitymanager</artifactId>
            <version>5.6.5.Final</version>
        </dependency>
        <!--mysql-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.27</version>
        </dependency>

        <!--Spring-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.2.1.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>5.2.1.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.9.7</version>
        </dependency>

        <!--SpringDataJpa-->
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-jpa</artifactId>
            <version>2.5.0</version>
        </dependency>
    </dependencies>
```

### 1.3 创建实体类

```java
@Data
@Entity
@Table(name = "article")
public class Article implements Serializable {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer aid;
    @Column(name = "author")
    private String author;
    @Column(name = "createTime")
    private LocalDateTime createTime;
    @Column(name = "title")
    private String title;
}
```

### 1.4 定义一个符合SpringDataJpa规范的Dao接口

- 继承两个接口
  - JpaRepository<T, ID>
    - T： 实体类
    - ID：实体类的主键类型
  - JpaSpecificationExecutor<T>
    - T： 实体类

```java
public interface ArticleDao 
extends JpaRepository<Article, Integer>, JpaSpecificationExecutor<Article> {
}
```

### 1.5 编写Spring配置文件，添加Jpa相关配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:jpa="http://www.springframework.org/schema/data/jpa"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
       http://www.springframework.org/schema/jpa http://www.springframework.org/schema/data/jpa/spring-jpa.xsd
">
    <!-- 包扫描 -->
    <context:component-scan base-package="com.ambitious.jpa_qs"/>

    <!-- 数据源 -->
    <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
        <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql:///spring_data?serverTimeZone=Asia/Shanghai"/>
        <property name="username" value="root"/>
        <property name="password" value="cyj070723"/>
    </bean>

    <!-- 持久化管理器工厂 -->
    <bean id="entityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
        <!-- 数据源 -->
        <property name="dataSource" ref="dataSource"/>
        <!-- 实体类 -->
        <property name="packagesToScan" value="com.ambitious.jpa_qs.domain"/>
        <!-- 服务提供商 -->
        <property name="persistenceProvider" value="org.hibernate.jpa.HibernatePersistenceProvider"/>
        <!-- hibernate配置 -->
        <property name="jpaVendorAdapter">
            <bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
                <!-- 使用哪个数据库 -->
                <property name="database" value="MYSQL"/>
                <!-- 是否自动建表 -->
                <property name="generateDdl" value="true"/>
                <!-- 是否显示SQL语句 -->
                <property name="showSql" value="true"/>
            </bean>
        </property>
    </bean>

    <!-- 配置事务管理器 -->
    <bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!-- 配置jpa -->
    <jpa:repositories base-package="com.ambitious.jpa_qs"
                      entity-manager-factory-ref="entityManagerFactory"
                      transaction-manager-ref="transactionManager"/>
</beans>
```



