## 1. SpringDataJpa快速入门

### 1.1 数据库表

> 数据库表由JPA自动生成即可

### 1.2 创建Java工程，引入依赖

**Spring依赖必须和SpringDataJpa依赖适配，否则程序无法运行，可以查看SpringDataJpa依赖查看匹配的Spring版本号**

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
            <version>5.3.6</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>5.3.6</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>5.3.6</version>
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
       http://www.springframework.org/schema/data/jpa http://www.springframework.org/schema/data/jpa/spring-jpa.xsd
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
        <property name="persistenceProvider">
            <bean class="org.hibernate.jpa.HibernatePersistenceProvider"/>
        </property>
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

### 1.6 测试

使用了SpringDataJpa之后，Dao的代码量可以明显减少

在使用SpringDataJpa修改数据库中的记录之前，最好先做一次查询，然后再进行修改，防止误删原有数据

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext-jpa.xml")
public class SpringDataJpaTest {

    @Autowired
    private ArticleDao articleDao;

    // 增
    @Test
    public void testSave() {
        Article article = new Article();
        article.setAuthor("李四");
        article.setTitle("这是李四写出来的标题");
        article.setCreateTime(LocalDateTime.now());
        Article res = articleDao.save(article);
        System.out.println(res);
    }

    // 查
    @Test
    public void testFind() {
        Optional<Article> optional = articleDao.findById(1);
        System.out.println(optional.get());
    }

    // 改
    @Test
    public void testModify() {
        // 将要修改的记录查出来
        Optional<Article> optional = articleDao.findById(2);
        if(optional.isPresent()) {
            Article article = optional.get();
            article.setAuthor("李四的妈妈");
            article.setTitle("我是李四的妈妈，我修改了李四");
            // SpringDataJpa的保存和修改方法是相同的
            articleDao.save(article);
        }
    }

    // 删
    @Test
    public void testDel() {
        articleDao.deleteById(2);
    }
}
```

## 2. 查询扩展

### 2.1 使用父接口的方法进行查询

使用JpaRepository接口中已经定义好的接口进行查询

#### 2.1.1 根据主键进行查询

```java
@Test
public void testById() {
    // 1 直接传入主键
    Optional<Article> optional = articleDao.findById(7);
    System.out.println(optional.get());
    // 2 根据多个主键进行查询
    List<Integer> list = new ArrayList<>();
    list.add(8);
    list.add(9);
    list.add(10);
    List<Article> articles = articleDao.findAllById(list);
    for (Article article : articles) {
        System.out.println(article);
    }
}
```

#### 2.1.2 查询所有

```java
@Test
public void testAll() {
    List<Article> articles = articleDao.findAll();
    for (Article article : articles) {
        System.out.println(article);
    }
}
```

#### 2.1.3 查询所有并排序

```java
@Test
public void findAllAndSort() {
    Sort sort = Sort.by(Sort.Order.desc("aid"));
    List<Article> articles = articleDao.findAll(sort);
    System.out.println(articles);
}
```

#### 2.1.4 查询所有并分页

```java
@Test
public void findAllAndPage() {
    // 分页中第0页为首页
    Pageable pageable = PageRequest.of(1, 3);
    Page<Article> articlePage = articleDao.findAll(pageable);
    System.out.println("总记录数：" + articlePage.getTotalElements());
    System.out.println("总页数：" + articlePage.getTotalPages());
    System.out.println("当前页：" + articlePage.getNumber());
    System.out.println("记录数：" + articlePage.getNumberOfElements());
    System.out.println("查询出来的数据：" + articlePage.getContent());
}
```

#### 2.1.5 查询所有并分页排序

```java
@Test
public void findAllAndPageAndSort() {
    Sort sort = Sort.by(Sort.Order.desc("aid"));
    Pageable pageable = PageRequest
        .of(2, 2)
        .withSort(sort);
    Page<Article> articlePage = articleDao.findAll(pageable);
    System.out.println("总记录数：" + articlePage.getTotalElements());
    System.out.println("总页数：" + articlePage.getTotalPages());
    System.out.println("当前页：" + articlePage.getNumber());
    System.out.println("记录数：" + articlePage.getNumberOfElements());
    System.out.println("查询出来的数据：" + articlePage.getContent());
}
```

### 2.2 使用方法命名规则查询

按照规定好的命名规则在Dao接口中自定义方法即可实现条件查询

 ```java
 public interface ArticleDao 
 extends JpaRepository<Article, Integer>, JpaSpecificationExecutor<Article> {
 
     /**
      * 根据标题进行查询
      * @param title 标题
      * @return 文章集合
      */
     List<Article> findByTitle(String title);
 
     /**
      * 根据标题模糊查询
      * @param title 标题
      * @return 文章集合
      */
     List<Article> findByTitleLike(String title);
 
     /**
      * 根据标题和作者查询
      * @param title 标题
      * @param author 作者
      * @return 文章集合
      */
     List<Article> findByTitleAndAuthor(String title, String author);
 
     /**
      * 查询某个id值以下所有文章
      * @param aid 主键
      * @return 文章集合
      */
     List<Article> findByAidLessThan(Integer aid);
 
     /**
      * 查询某个时间之后创建的记录
      * @param time 时间
      * @return 文章集合
      */
     List<Article> findByCreateTimeAfter(LocalDateTime time);
 
     /**
      * 查询某个区间id的记录
      * @param collection id集合
      * @return 文章集合
      */
     List<Article> findByAidIn(Collection<Integer> collection);
 }
 ```

### 2.3 使用@Query注解 + JPQL进行查询

JPQL的语法类似于SQL，只不过是面向实体类的，程序员无需关注表结构

#### 2.3.1 基本参数绑定

使用`:变量名`进行绑定，需要使用注解@param指定对应关系，与参数名无关

```java
@Query("from Article a where a.title = :title and a.author = :author")
List<Article> findByCondition1(@Param("title") String title,@Param("author") String author);
```

#### 2.3.2 模糊查询

```java
@Query("from Article a where a.title like %:title%")
List<Article> findByCondition2(@Param("title") String title);
```

#### 2.3.3 排序查询

```java
@Query("from Article a where a.title like %:title% order by a.aid desc")
List<Article> findByCondition3(@Param("title") String title);
```

#### 2.3.4 分页查询

```java
@Query("from Article a where a.title like %:title%")
Page<Article> findByCondition4(Pageable pageable, @Param("title") String title);
```

#### 2.3.5 实体查询

使用`:#{#参数名.属性名}`表示实体中的某个属性值

```java
@Query("from Article a where a.title = :#{#article.title} and a.author = :#{#article.author}")
List<Article> findByCondition5(@Param("article") Article article);
```

