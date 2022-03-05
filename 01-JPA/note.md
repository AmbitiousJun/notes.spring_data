## 1. JPA基础

JPA的全称是Java Persistence API， 即Java持久化API， 是sun公司推出的一套基于ORM的规范， 注意不是ORM框架——因为JPA并未提供ORM实现，它只是提供了一些编程的API接口。

## 2. JPA实战

搭建jpa环境，实现一条数据的增删改查

### 2.1 创建数据库环境

#### 2.1.1 创建MySQL数据库，起名为spring-data

![](./assets/实战-创建数据库.png)



#### 2.1.2 新建表，起名为article

```sql
create table `article` (
	`aid` int auto_increment primary key comment '主键',
	`author` varchar(255) default null comment '作者',
	`createTime` datetime default null comment '创建时间',
	`title` varchar(255) default null comment '标题'
)
```

### 2.2 创建Java工程，引入依赖

- mysql驱动
- hibernate（JPA只是提供了ORM接口，所以需要引入一个具体的ORM框架）
- Junit
- lombok

```xml
<dependencies>
        <!--MySQL-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.27</version>
        </dependency>
        <!--hibernate-->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-entitymanager</artifactId>
            <version>5.6.5.Final</version>
        </dependency>
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
    </dependencies>
```

### 2.3 创建article实体类，并完成与数据表之间的映射

#### 2.3.1 创建实体类

```java
@Data
public class Article {

    private Integer aid;
    private String author;
    private Date createTime;
    private String title;
}
```

#### 2.3.2 添加JPA注解，完成映射

- @Entity：标注该类是一个实体类，<font style="color:red">必须添加，否则下面的注解无法生效</font>
- @Table：指定该实体类在数据库中的表名
- @Id： 指定主键子段
  - @GenerateValue：指定主键生成策略
- @Column： 指定字段名称，当实体类属性名和表字段名一致时，可不用该注解

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
    private Date createTime;
    @Column(name = "title")
    private String title;
}
```



